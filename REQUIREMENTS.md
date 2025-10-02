# LINE名言チャットボット 要件定義書

**バージョン**: 1.0  
**作成日**: 2025年10月2日  
**プロジェクト**: line-quote-bot  

---

## 1. プロジェクト概要

### 1.1 システム名称
**LINE名言ボット（Python版）**

### 1.2 目的・用途
- **主目的**: 自分専用、どんな会話にも適切な名言を返すLINEチャットボット
- **対象ユーザー**: 開発者本人（1名）
- **利用シーン**: 日常会話、励まし、インスピレーション取得

### 1.3 主要機能
1. **テーマ抽出＋名言返信**: ユーザー入力に応じた適切な名言の自動選択・返信
2. **コンテキスト判断**: 日付・曜日・過去会話履歴に基づくテーマ判断
3. **定期通知**: 毎日指定時刻の自動名言配信
4. **学習機能**: 利用状況・フィードバックに応じた推薦スコア調整
5. **品質管理**: 不適切な名言のフィードバック・除外機能

---

## 2. 技術アーキテクチャ

### 2.1 技術スタック
| 項目 | 技術 | 理由 |
|------|------|------|
| **API処理** | Python 3.11 | 豊富なAIライブラリ、可読性 |
| **ホスティング** | Vercel Serverless Functions | 無料枠、サーバーレス |
| **データベース** | PostgreSQL (Supabase) | 無料枠500MB、高機能 |
| **AI処理** | sentence-transformers/distilbert | 軽量、高精度 |
| **定期実行** | GitHub Actions | 無料枠、簡単設定 |
| **キャッシュ** | Vercel KV / Redis | 高速応答 |

### 2.2 システム構成図
```
ユーザー(LINE) 
    ↓
LINE Messaging API 
    ↓
Vercel Serverless Function (Python)
    ↓
├─ Supabase (PostgreSQL) ← 名言DB、会話履歴
├─ AI モデル (sentence-transformers) ← テーマ抽出
└─ Vercel KV ← キャッシュ
    ↓
LINE返信
```

---

## 3. 機能要件

### 3.1 メイン機能：名言返信

#### 3.1.1 入力処理
- **対象**: LINE テキストメッセージ（最大1,000文字）
- **前処理**: 特殊文字エスケープ、空文字・スパム検知
- **除外**: 画像・スタンプ・位置情報等は「テキストでお話しください」返信

#### 3.1.2 テーマ抽出
- **AIモデル**: sentence-transformers (distilbert-base)
- **処理時間**: 1-3秒以内
- **精度目標**: F1スコア 0.8以上
- **テーマ例**: 励まし、恋愛、仕事、友情、家族、成長、挑戦、感謝

#### 3.1.3 名言選択ロジック
```python
def select_quote(theme, user_context):
    # 1. テーママッチング（完全一致→部分一致）
    # 2. コンテキスト考慮（曜日、時刻、季節）
    # 3. 利用履歴考慮（7日以内の重複回避）
    # 4. 重要度スコア順でランキング
    # 5. 上位3件からランダム選択
    return selected_quote
```

### 3.2 定期通知機能

#### 3.2.1 実行タイミング
- **頻度**: 毎日1回（朝7:00 JST）
- **実行方法**: GitHub Actions → Vercel API呼び出し
- **祝日対応**: 日本の祝日カレンダー考慮

#### 3.2.2 通知内容
- 日付に応じた特別名言（誕生日、記念日、季節イベント）
- 曜日ベース名言（月曜日→やる気、金曜日→達成感）
- デフォルト：汎用的な励まし・インスピレーション名言

### 3.3 フィードバック機能

#### 3.3.1 評価収集
- **方法**: LINE Quick Reply（👍/👎ボタン）
- **詳細評価**: 1-5段階評価（任意）
- **コメント**: 自由記述（任意、最大200文字）

#### 3.3.2 学習反映
- 高評価名言の重要度スコア向上（+0.1）
- 低評価名言の重要度スコア低下（-0.2）
- 連続低評価（3回以上）で自動非表示

---

## 4. 非機能要件

### 4.1 パフォーマンス要件

| 項目 | 目標値 | 測定方法 |
|------|--------|----------|
| **応答時間（Cold Start）** | 5秒以内 | Vercel Analytics |
| **応答時間（Warm）** | 1秒以内 | Vercel Analytics |
| **99%ile応答時間** | 3秒以内 | 週次レポート |
| **同時接続数** | 10ユーザー | 自分専用のため |
| **1日リクエスト数** | 200件以内 | 使用量監視 |

### 4.2 可用性要件

| 項目 | 目標値 | 対策 |
|------|--------|------|
| **稼働率** | 99%以上 | Vercel SLA + 監視 |
| **RTO（復旧時間）** | 15分以内 | 手動復旧手順 |
| **RPO（データ損失）** | 24時間以内 | 日次バックアップ |

### 4.3 拡張性要件
- **名言DB容量**: 最大10,000件（約100MB）
- **会話履歴**: 14日間保持（自動削除）
- **ユーザー数**: 1名（将来的に5名まで拡張可能）

---

## 5. データ設計

### 5.1 名言マスタ（quotes）

```sql
CREATE TABLE quotes (
    id SERIAL PRIMARY KEY,
    text TEXT NOT NULL CHECK (length(text) BETWEEN 10 AND 500),
    author VARCHAR(100),
    theme VARCHAR(50) NOT NULL,
    weekday INTEGER CHECK (weekday >= 0 AND weekday <= 6),
    date_type VARCHAR(20), -- 'birthday', 'newyear', 'christmas', etc.
    importance_score FLOAT DEFAULT 1.0 CHECK (importance_score >= 0),
    usage_count INTEGER DEFAULT 0,
    positive_feedback INTEGER DEFAULT 0,
    negative_feedback INTEGER DEFAULT 0,
    source_url TEXT,
    language VARCHAR(10) DEFAULT 'ja',
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    last_used TIMESTAMP,
    is_active BOOLEAN DEFAULT true
);

-- インデックス
CREATE INDEX idx_quotes_theme_active ON quotes(theme, is_active);
CREATE INDEX idx_quotes_weekday_active ON quotes(weekday, is_active);
CREATE INDEX idx_quotes_importance ON quotes(importance_score DESC);
```

### 5.2 会話履歴（conversation_history）

```sql
CREATE TABLE conversation_history (
    id SERIAL PRIMARY KEY,
    user_id VARCHAR(100) NOT NULL,
    timestamp TIMESTAMP DEFAULT NOW(),
    input_text TEXT NOT NULL,
    extracted_theme VARCHAR(50),
    selected_quote_id INTEGER REFERENCES quotes(id),
    response_time_ms INTEGER,
    ai_confidence FLOAT CHECK (ai_confidence >= 0 AND ai_confidence <= 1),
    session_id VARCHAR(100),
    user_feedback INTEGER CHECK (user_feedback >= 1 AND user_feedback <= 5),
    feedback_comment TEXT
);

-- インデックス
CREATE INDEX idx_conv_user_timestamp ON conversation_history(user_id, timestamp DESC);
CREATE INDEX idx_conv_theme ON conversation_history(extracted_theme);
```

### 5.3 ユーザー設定（user_settings）

```sql
CREATE TABLE user_settings (
    user_id VARCHAR(100) PRIMARY KEY,
    notification_time TIME DEFAULT '07:00:00',
    timezone VARCHAR(50) DEFAULT 'Asia/Tokyo',
    language VARCHAR(10) DEFAULT 'ja',
    is_notification_enabled BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

---

## 6. 外部システム連携

### 6.1 LINE Messaging API

#### 6.1.1 制約事項
- **無料枠**: 1,000通/月
- **Webhook タイムアウト**: 30秒
- **メッセージサイズ**: 最大5,000文字

#### 6.1.2 必要な設定
- Channel Access Token
- Channel Secret
- Webhook URL: `https://your-app.vercel.app/api/webhook`

### 6.2 Supabase

#### 6.2.1 制約事項
- **無料枠**: 500MB ストレージ、60同時接続
- **バックアップ**: 日次自動バックアップ（7日間保持）

#### 6.2.2 接続設定
- Connection Pool: 最大50接続
- Query Timeout: 30秒
- SSL: 必須

---

## 7. セキュリティ要件

### 7.1 認証・認可
- **LINE署名検証**: 必須（Channel Secretによる）
- **ユーザー制限**: 許可されたLINE User IDのみ応答
- **管理画面**: Basic認証（ユーザー名/パスワード）

### 7.2 データ保護
- **通信暗号化**: HTTPS必須
- **データベース**: SSL/TLS接続
- **ログマスキング**: ユーザーIDの一部のみ記録
- **個人情報**: 会話内容は14日で自動削除

### 7.3 入力値検証
```python
def validate_input(text):
    # 文字数制限（1-1000文字）
    # 特殊文字エスケープ
    # SQLインジェクション対策（ORMで対応）
    # XSS対策（出力時エスケープ）
    return sanitized_text
```

### 7.4 Rate Limiting
- **同一ユーザー**: 1分間10リクエスト
- **IP別**: 1時間100リクエスト
- **グローバル**: 1日1,000リクエスト

---

## 8. 運用要件

### 8.1 監視項目

#### 8.1.1 アプリケーション監視
- **エラー率**: 5%以下維持
- **応答時間**: 平均1秒以下
- **Webhook成功率**: 95%以上
- **AI処理精度**: F1スコア 0.8以上

#### 8.1.2 インフラ監視
- **Vercel Function実行回数**: 月125,000回以内
- **Supabaseストレージ使用量**: 500MB以内
- **GitHub Actions実行時間**: 月2,000分以内

### 8.2 アラート設定
- **Critical**: エラー率10%超過 → 即座通知
- **High**: 応答時間3秒超過 → 5分以内通知
- **Medium**: 無料枠90%到達 → 日次通知

### 8.3 バックアップ・復旧
- **自動バックアップ**: Supabase日次バックアップ
- **手動バックアップ**: 週次フルエクスポート（JSON/CSV）
- **復旧テスト**: 月次リストアテスト実施

---

## 9. 開発・テスト要件

### 9.1 テスト戦略

#### 9.1.1 テストレベル
- **単体テスト**: カバレッジ80%以上
- **統合テスト**: API・DB連携テスト
- **E2Eテスト**: LINE Webhook → 名言返信の全体フロー

#### 9.1.2 テストケース例
```python
# 正常系
def test_normal_message():
    assert extract_theme("頑張ります") == "励まし"
    
# 異常系  
def test_empty_message():
    assert handle_message("") == "テキストでお話しください"
    
# 境界値
def test_long_message():
    long_text = "a" * 1001
    assert len(handle_message(long_text)) > 0
```

### 9.2 環境分離
- **開発環境**: ローカル + Docker Compose
- **テスト環境**: Vercel Preview + Test DB
- **本番環境**: Vercel Production + Supabase Production

---

## 10. 制約事項・前提条件

### 10.1 無料枠制約
| サービス | 制限 | 対策 |
|----------|------|------|
| Vercel | 125,000回/月 | 使用量監視・制限 |
| Supabase | 500MB | データ圧縮・古いデータ削除 |
| LINE API | 1,000通/月 | 送信数カウント |
| GitHub Actions | 2,000分/月 | 効率的なワークフロー |

### 10.2 技術制約
- **Cold Start**: 初回アクセス時2-5秒の遅延
- **ファイル永続化不可**: Vercel Serverlessの制約
- **同時接続数**: Supabase 60接続まで

### 10.3 運用制約
- **サポート対象**: 開発者本人のみ
- **運用時間**: 24時間365日（ベストエフォート）
- **障害対応**: 手動復旧（15分以内目標）

---

## 11. 今後の拡張計画

### 11.1 Phase 2（3ヶ月後）
- 多言語対応（英語）
- 画像付き名言
- カスタムテーマ作成

### 11.2 Phase 3（6ヶ月後）
- 音声メッセージ対応
- グループチャット対応
- 名言投稿機能

### 11.3 Phase 4（1年後）
- 有料プラン検討
- マルチユーザー対応
- 高度なAI分析

---

## 12. 関連ドキュメント

- [開発フロー](./DEVELOPMENT.md)
- [API仕様書](./api/README.md) ※今後作成
- [デプロイ手順](./deploy/README.md) ※今後作成
- [運用マニュアル](./operations/README.md) ※今後作成

---

**承認者**: 開発者本人  
**承認日**: 2025年10月2日