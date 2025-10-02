# 開発フロー（LINE名言ボット）

## 概要
LINE名言ボットの開発における1サイクルの標準的なワークフローを定義します。

## 1. イシュー作成

### GitHubのIssueでタスク定義
- **機能追加**：テーマ抽出ロジック改善、新機能追加
- **バグ修正**：Webhookタイムアウト対策、エラー修正
- **テスト追加**：特殊文字入力ケース、エッジケーステスト

### Issue内容テンプレート
```markdown
## 概要
[何をするのか簡潔に説明]

## 目的
[なぜこの作業が必要なのか]

## 期待結果
[完了時の状態・成果物]

## 依存関係
[他のIssueや作業との関連]

## 優先度・期限
- 優先度: High/Medium/Low
- 期限: YYYY-MM-DD
```

---

## 2. ブランチ作成

### 命名規則
```bash
# 機能追加
git checkout -b feature/issue-123-theme-detection-improvement

# バグ修正
git checkout -b bugfix/issue-456-webhook-timeout

# 緊急修正
git checkout -b hotfix/critical-db-connection-fix
```

---

## 3. ローカル開発

### 開発内容
- **LINE Webhook処理**の実装・修正
- **名言DBアクセス**（Supabaseクエリ）の改善
- **AIテーマ抽出**（sentence-transformers）の精度向上

### 動作確認
```bash
# ユニットテスト実行
pytest tests/ --cov=src/ --cov-report=html

# コードフォーマット・リント
black .
flake8 src/ tests/

# ローカル環境でのAPI動作確認
python -m src.main  # ローカルサーバー起動
```

### 環境設定
- 開発環境: `.env.dev`
- テスト環境: `.env.test`
- 本番環境: Vercel Environment Variables

---

## 4. コミット・プッシュ

### コミットメッセージ規則
```bash
# 新機能
git commit -m "feat: add weekday-based quote filtering"

# バグ修正
git commit -m "fix: handle empty LINE messages properly"

# ドキュメント更新
git commit -m "docs: update API documentation"

# テスト追加
git commit -m "test: add edge cases for theme detection"

# リファクタリング
git commit -m "refactor: optimize database query performance"
```

### コミット頻度
- 小さな単位で頻繁にコミット
- 1つの機能・修正につき複数コミットOK
- PR時にSquash mergeで整理

---

## 5. プルリクエスト（PR）作成

### PR内容テンプレート
```markdown
## 変更内容
[何を変更したか]

## 関連Issue
Closes #123

## テスト結果
- [ ] ユニットテスト: 全て通過
- [ ] 統合テスト: 全て通過
- [ ] 手動テスト: 動作確認済み

## レビューポイント
[特に注意して見てほしい箇所]

## デプロイ前の確認事項
- [ ] 環境変数の設定確認
- [ ] DBマイグレーション不要
- [ ] ブレイキングチェンジなし
```

---

## 6. コードレビュー

### 自動チェック（GitHub Actions）
- lint/format チェック（black, flake8）
- ユニットテスト実行（pytest）
- セキュリティチェック
- コードカバレッジ確認（80%以上）

### AIレビュー活用
1. **GitHub Copilot**: リアルタイム提案
2. **ChatGPT/Claude**: PRのDiffレビュー
3. **自動セキュリティスキャン**: GitHub Advanced Security

### レビュー観点
- **ロジック**: アルゴリズムの妥当性
- **エラーハンドリング**: 例外処理の適切性
- **パフォーマンス**: クエリ・API呼び出しの最適化
- **セキュリティ**: 入力値検証・SQLインジェクション対策
- **可読性**: 命名・コメント・構造

---

## 7. マージ

### マージ戦略
- **Squash merge**: コミット履歴を整理
- **main**ブランチへの直接プッシュ禁止
- PR承認後のマージ

### マージ後の作業
```bash
# ローカルブランチ削除
git checkout main
git pull origin main
git branch -d feature/issue-123-theme-detection-improvement

# リモートブランチ削除（自動削除設定推奨）
git push origin --delete feature/issue-123-theme-detection-improvement
```

---

## 8. CI/CDデプロイ

### 自動デプロイフロー
1. **mainブランチマージ** → GitHub Actions トリガー
2. **ビルド・テスト** → pytest, lint 実行
3. **Vercelデプロイ** → Serverless Function更新
4. **本番確認** → Webhook疎通確認

### デプロイ確認項目
- Vercel Function正常起動
- LINE Webhook応答確認
- Supabase接続確認
- エラーログ確認

---

## 9. 監視・検証

### 監視項目
- **Vercel Analytics**: 関数実行回数・エラー率・応答時間
- **Supabaseダッシュボード**: DB接続数・クエリパフォーマンス
- **LINEログ**: Webhook成功率・メッセージ送信数

### アラート設定
- エラー率5%超過 → Slack通知
- 応答時間3秒超過 → メール通知
- DB接続失敗 → 即座通知

### 定期確認（毎日）
- [ ] エラーログ確認
- [ ] パフォーマンス指標確認
- [ ] 無料枠使用量確認

---

## 10. サイクル完了・次イシューへ

### 完了後の作業
1. **Issue Close**: Issueを完了としてクローズ
2. **ドキュメント更新**: 必要に応じてREADME更新
3. **次タスク選択**: Issue Boardから次の優先度確認

### 継続改善
- 開発効率の振り返り
- テストカバレッジの確認
- パフォーマンス改善の検討

---

## トラブルシューティング

### よくある問題と対処法

#### デプロイ失敗
```bash
# Vercelログ確認
vercel logs

# ローカルで再現テスト
vercel dev
```

#### テスト失敗
```bash
# 詳細なテスト結果確認
pytest -v --tb=long

# 特定のテストのみ実行
pytest tests/test_webhook.py::test_message_handling -v
```

#### DB接続エラー
- Supabaseコネクション制限確認
- 環境変数設定確認
- ネットワーク接続確認

---

## 開発ツール

### 推奨ツール
- **GitHub CLI**: `gh pr create`, `gh issue list`
- **Vercel CLI**: `vercel dev`, `vercel logs`
- **Supabase CLI**: `supabase start`, `supabase db reset`
- **pre-commit**: コミット前自動チェック

### VSCode拡張機能
- Python
- GitLens
- GitHub Pull Requests
- Pylance
- Black Formatter

---

## 参考リンク
- [GitHub Issues](https://github.com/takahiro64/line-quote-bot/issues)
- [Vercel Dashboard](https://vercel.com/dashboard)
- [Supabase Dashboard](https://app.supabase.com/)
- [LINE Developers Console](https://developers.line.biz/)