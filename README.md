# LINE名言チャットボット

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/takahiro64s-projects/line-quote-bot-)
[![Python](https://img.shields.io/badge/Python-3.11-blue.svg)](https://www.python.org/)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

🎯 **自分専用、どんな会話にも適切な名言を返すLINEチャットボット**

AIが会話の文脈を理解し、その時の気持ちや状況に最適な名言を選んで返信します。毎日の励ましや、ふとした瞬間のインスピレーションを提供します。

## ✨ 主な機能

### 🤖 インテリジェント名言返信
- **コンテキスト理解**: AI（sentence-transformers）が会話内容からテーマを抽出
- **最適な選択**: 10,000以上の名言から状況に応じて最適なものを選択
- **重複回避**: 過去7日間の履歴を考慮して新鮮な名言を提供

### 📅 スマート定期通知
- **毎日の励まし**: 朝7時に自動で今日にぴったりな名言を配信
- **カレンダー連携**: 曜日・祝日・季節イベントに応じた特別な名言
- **パーソナライズ**: 過去の反応を学習して好みを反映

### 🎯 学習機能
- **フィードバック対応**: 👍/👎で名言の評価を収集
- **自動改善**: 評価に基づいて推薦アルゴリズムを継続改善
- **品質管理**: 低評価が続く名言は自動で非表示

## 🛠️ 技術スタック

| 項目 | 技術 | 用途 |
|------|------|------|
| **Backend** | Python 3.11 | メインアプリケーション |
| **Hosting** | Vercel Serverless | APIホスティング |
| **Database** | PostgreSQL (Supabase) | 名言・履歴データ |
| **AI/ML** | sentence-transformers | テーマ抽出・自然言語処理 |
| **Cache** | Vercel KV | 高速レスポンス |
| **CI/CD** | GitHub Actions | 自動テスト・デプロイ |
| **Monitoring** | Vercel Analytics | パフォーマンス監視 |

## 🚀 セットアップ

### 前提条件
- Python 3.11+
- Node.js 18+ (Vercel CLI用)
- Git

### 1. リポジトリのクローン
```bash
git clone https://github.com/takahiro64/line-quote-bot.git
cd line-quote-bot
```

### 2. 依存関係のインストール
```bash
# Python依存関係
pip install -r requirements.txt

# Vercel CLI（グローバル）
npm install -g vercel
```

### 3. 環境変数の設定
```bash
# .env.local ファイルを作成
cp .env.example .env.local
```

`.env.local` に以下を設定：
```env
# LINE Bot設定
LINE_CHANNEL_ACCESS_TOKEN=your_line_channel_access_token
LINE_CHANNEL_SECRET=your_line_channel_secret

# Supabase設定
SUPABASE_URL=your_supabase_url
SUPABASE_KEY=your_supabase_anon_key

# 環境設定
ENVIRONMENT=development
LOG_LEVEL=INFO
CACHE_TTL=3600

# ユーザー制限（あなたのLINE User ID）
ALLOWED_USER_IDS=your_line_user_id
```

### 4. データベースの初期化
```bash
# Supabaseマイグレーション実行
python scripts/init_database.py

# サンプル名言データの投入
python scripts/load_sample_quotes.py
```

### 5. ローカル開発サーバーの起動
```bash
# Vercel開発サーバー
vercel dev

# または直接Python実行
python -m src.main
```

🎉 `http://localhost:3000` でローカル環境が起動します！

## 📱 LINE Bot設定

### 1. LINE Developers Console設定
1. [LINE Developers Console](https://developers.line.biz/) にアクセス
2. 新しいProvider・Channelを作成
3. Messaging API設定:
   - Webhook URL: `https://your-app.vercel.app/api/webhook`
   - Use webhook: ON
   - Auto-reply messages: OFF
   - Greeting messages: OFF

### 2. Webhook URLの設定
```bash
# Vercelデプロイ後
vercel --prod

# WebhookテストEndpoint
curl -X POST https://your-app.vercel.app/api/webhook \
  -H "Content-Type: application/json" \
  -d '{"events":[]}'
```

## 🏗️ プロジェクト構造

```
line-quote-bot/
├── api/                    # Vercel Serverless Functions
│   └── webhook.py         # LINE Webhook処理
├── src/                   # メインアプリケーション
│   ├── __init__.py
│   ├── main.py           # エントリーポイント
│   ├── handlers/         # メッセージハンドラー
│   ├── services/         # ビジネスロジック
│   ├── models/           # データモデル
│   ├── ai/               # AI・ML処理
│   └── utils/            # ユーティリティ
├── tests/                 # テストファイル
├── scripts/              # セットアップ・管理スクリプト
├── migrations/           # データベースマイグレーション
├── docs/                 # ドキュメント
├── .github/              # GitHub Actions設定
├── requirements.txt      # Python依存関係
├── vercel.json          # Vercel設定
├── REQUIREMENTS.md      # 要件定義
├── DEVELOPMENT.md       # 開発フロー
└── README.md            # このファイル
```

## 🧪 テスト実行

```bash
# 全テスト実行
pytest

# カバレッジ付きテスト
pytest --cov=src --cov-report=html

# 特定のテストファイル
pytest tests/test_webhook.py -v

# リント・フォーマット
black . && flake8 src/ tests/
```

## 🚀 デプロイ

### Vercelデプロイ
```bash
# 初回デプロイ
vercel

# 本番デプロイ
vercel --prod

# 環境変数設定
vercel env add LINE_CHANNEL_ACCESS_TOKEN
vercel env add SUPABASE_URL
# ... その他の環境変数
```

### GitHub Actions（自動デプロイ）
1. GitHub Secretsに環境変数を設定
2. `main`ブランチにプッシュで自動デプロイ
3. PRマージ時に本番環境更新

## 📊 使用方法

### 基本的な会話
```
あなた: 最近仕事がうまくいかなくて...
ボット: 「失敗は成功の母である。」- トーマス・エジソン

あなた: 明日から新しいことを始めます
ボット: 「千里の道も一歩から。」- 老子
```

### フィードバック機能
名言に対して👍/👎ボタンで評価可能。評価は学習に反映されます。

### 定期通知
毎朝7時に、その日に適した名言が自動配信されます。

## 🔧 設定・カスタマイズ

### 通知時間の変更
```python
# src/config.py
NOTIFICATION_TIME = "07:00"  # JST
```

### 新しいテーマの追加
```python
# src/ai/theme_extractor.py
THEMES = {
    "励まし": ["頑張", "辛い", "疲れ"],
    "成功": ["成功", "達成", "目標"],
    "新しいテーマ": ["キーワード1", "キーワード2"]  # 追加
}
```

### 名言の追加
```sql
-- Supabaseで直接実行
INSERT INTO quotes (text, author, theme, importance_score) 
VALUES ('新しい名言', '著者名', 'テーマ', 1.0);
```

## 📈 監視・分析

### ダッシュボード
- **Vercel Analytics**: 関数実行回数・エラー率・応答時間
- **Supabase Dashboard**: DB使用量・クエリパフォーマンス
- **GitHub Actions**: ビルド・デプロイ状況

### ログ確認
```bash
# Vercelログ
vercel logs

# アプリケーションログ
tail -f logs/application.log
```

## 🤝 コントリビューション

### 開発フロー
1. Issueを作成してタスクを定義
2. `feature/issue-123-description` ブランチを作成
3. 実装・テスト・コミット
4. Pull Requestを作成
5. レビュー・マージ

詳細は [DEVELOPMENT.md](DEVELOPMENT.md) を参照してください。

### バグ報告・機能要望
[GitHub Issues](https://github.com/takahiro64/line-quote-bot/issues) でお気軽にご報告ください。

## 📄 ドキュメント

- [要件定義書](REQUIREMENTS.md) - 詳細な機能・技術仕様
- [開発フロー](DEVELOPMENT.md) - 開発サイクル・ツール使用法
- [API仕様書](docs/api.md) - REST API詳細 ※今後作成
- [運用マニュアル](docs/operations.md) - 障害対応・メンテナンス ※今後作成

## 🔒 セキュリティ

- **認証**: LINE署名検証・ユーザーID制限
- **暗号化**: HTTPS・SSL/TLS通信
- **データ保護**: 個人情報は14日で自動削除
- **入力検証**: SQLインジェクション・XSS対策

セキュリティの問題を発見した場合は、Issueではなく直接ご連絡ください。

## 📊 パフォーマンス指標

| 項目 | 目標値 | 現在値 |
|------|--------|--------|
| 応答時間（Warm） | 1秒以内 | - |
| 応答時間（Cold Start） | 5秒以内 | - |
| 稼働率 | 99%以上 | - |
| エラー率 | 5%以下 | - |

## 💸 コスト管理

### 無料枠での運用
| サービス | 制限 | 使用量目安 |
|----------|------|------------|
| Vercel | 125,000回/月 | ~50,000回/月 |
| Supabase | 500MB | ~100MB |
| LINE API | 1,000通/月 | ~200通/月 |
| GitHub Actions | 2,000分/月 | ~100分/月 |

**月額コスト**: $0（無料枠内で運用可能）

## 🗓️ ロードマップ

### v1.0（現在）
- [x] 基本的な名言返信機能
- [x] AIテーマ抽出
- [x] 定期通知
- [x] フィードバック機能

### v1.1（3ヶ月後）
- [ ] 多言語対応（英語）
- [ ] 画像付き名言
- [ ] 詳細な統計ダッシュボード
- [ ] カスタムテーマ作成

### v2.0（6ヶ月後）
- [ ] 音声メッセージ対応
- [ ] グループチャット対応
- [ ] 外部名言API連携
- [ ] モバイルアプリ版

## 🆘 トラブルシューティング

### よくある問題

#### 1. Webhookが応答しない
```bash
# LINE設定確認
curl -v https://your-app.vercel.app/api/webhook

# Vercelログ確認
vercel logs --follow
```

#### 2. データベース接続エラー
```python
# 接続テスト
python scripts/test_db_connection.py
```

#### 3. AI処理が遅い
```python
# モデルキャッシュ確認
python scripts/warm_up_model.py
```

詳細は [DEVELOPMENT.md#トラブルシューティング](DEVELOPMENT.md#トラブルシューティング) を参照。

## 📞 サポート・連絡先

- **GitHub Issues**: バグ報告・機能要望
- **Email**: [your-email@example.com](mailto:your-email@example.com)
- **Twitter**: [@your_twitter](https://twitter.com/your_twitter)

## 📜 ライセンス

このプロジェクトは [MIT License](LICENSE) の下で公開されています。

## 🙏 謝辞

- [LINE Messaging API](https://developers.line.biz/) - LINE Bot基盤
- [Supabase](https://supabase.com/) - データベース・認証
- [Vercel](https://vercel.com/) - ホスティング・デプロイ
- [Hugging Face](https://huggingface.co/) - AI・ML モデル
- すべての名言の原著者・引用元

---

**Made with ❤️ for daily inspiration**

> 「今日が人生最後の日だとしたら、今日やろうとしていることを本当にやりたいか？」  
> - スティーブ・ジョブズ
