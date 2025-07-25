# GitHub Actions スケジュール実行ガイド

## 概要
GitHub Actionsのcron式を使用した定期実行により、プロジェクトの健康管理を自動化します。

## 実装したワークフロー

### 1. 日次健康チェック（daily-health-check.yml）
**実行時刻**: 毎日朝9時（JST）
**機能**:
- 全WDプロジェクトの更新状況確認
- 健康状態バッジの自動更新
- Discord通知
- レポートの生成とアーカイブ

### 2. 週次リマインダー（weekly-project-reminder.yml）
**実行時刻**: 毎週月曜朝8時（JST）
**機能**:
- 放置プロジェクトの検出
- GitHubイシューの自動作成
- 具体的なアクション提案
- Discord通知

### 3. 日次5分タスク生成（auto-5min-tasks.yml）
**実行時刻**: 毎日朝7時（JST）
**機能**:
- 最も古いプロジェクトを選択
- ランダムな5分タスクを3つ提案
- 対象リポジトリにIssue作成

### 4. 月次サマリー（monthly-summary.yml）
**実行時刻**: 毎月1日朝10時（JST）
**機能**:
- 月間統計の収集
- グラフ生成
- 総合レポート作成
- アーカイブ保存

## cron式の基本

```
┌───────────── 分 (0 - 59)
│ ┌───────────── 時 (0 - 23)
│ │ ┌───────────── 日 (1 - 31)
│ │ │ ┌───────────── 月 (1 - 12)
│ │ │ │ ┌───────────── 曜日 (0 - 7) (0と7は日曜日)
│ │ │ │ │
│ │ │ │ │
* * * * *
```

### JST（日本標準時）への変換
GitHub ActionsはUTCで動作するため、JSTから9時間引く必要があります：
- JST 9:00 → UTC 0:00 → cron: `0 0 * * *`
- JST 18:00 → UTC 9:00 → cron: `0 9 * * *`

## セットアップ手順

### 1. リポジトリシークレットの設定
```
Settings → Secrets and variables → Actions
```
必要なシークレット：
- `DISCORD_WEBHOOK_URL`: Discord通知用
- `PERSONAL_ACCESS_TOKEN`: 他リポジトリへのIssue作成用

### 2. ワークフローの有効化
```
Actions → ワークフローを選択 → Enable workflow
```

### 3. 手動実行テスト
```
Actions → ワークフローを選択 → Run workflow
```

## カスタマイズ例

### 営業日のみ実行
```yaml
schedule:
  # 月-金の朝9時
  - cron: '0 0 * * 1-5'
```

### 複数回実行
```yaml
schedule:
  # 朝9時と夕方6時
  - cron: '0 0 * * *'
  - cron: '0 9 * * *'
```

### 特定日のみ実行
```yaml
schedule:
  # 毎月15日の正午
  - cron: '0 3 15 * *'
```

## トラブルシューティング

### ワークフローが実行されない
1. デフォルトブランチ（main/master）に.ymlファイルがあるか確認
2. 60日以上リポジトリが更新されていない場合は自動的に無効化される
3. cron式の構文エラーをチェック

### 実行時刻がずれる
- GitHub Actionsは最大1時間程度の遅延が発生する可能性
- 正確な時刻が必要な場合は外部サービスとの連携を検討

### API制限
- GitHub APIのレート制限に注意（認証済み: 5000/時）
- 大量のリポジトリを扱う場合はバッチ処理を実装

## ベストプラクティス

1. **エラーハンドリング**: 各ステップに適切なエラー処理を実装
2. **通知の適切化**: 過度な通知は避け、重要な情報のみ送信
3. **リトライ機能**: ネットワークエラーに備えてリトライを実装
4. **ログ出力**: デバッグのために詳細なログを出力
5. **テスト環境**: 本番環境とは別にテスト用ワークフローを作成

## 応用アイデア

### 1. プロジェクト間の依存関係チェック
```yaml
- name: Check dependencies
  run: |
    python scripts/check_cross_dependencies.py
```

### 2. 自動PRレビュー催促
```yaml
- name: Find stale PRs
  run: |
    python scripts/find_stale_prs.py --days 3
```

### 3. コード品質の定期チェック
```yaml
- name: Run quality checks
  run: |
    npm run lint
    npm run test
    python -m pytest
```

## 次のステップ

1. 実際のプロジェクトでワークフローをテスト
2. Discord Webhookの設定
3. カスタムスクリプトの実装
4. ダッシュボードの構築