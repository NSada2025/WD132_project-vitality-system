# GitHub Projects統合ガイド

## GitHub Projectsとは
カンバンボード形式のプロジェクト管理ツール。Issue、PR、カスタムアイテムを視覚的に管理。

## WD132との統合メリット

### 1. 全プロジェクト横断管理
- WD126-132の全タスクを一元管理
- 健康状態の可視化
- 放置プロジェクトの自動検出

### 2. 自動化可能な要素
- Issueの自動作成（5分タスク）
- ステータスの自動更新
- 進捗レポートの生成

### 3. ビュー設定例

#### 健康状態ビュー
```
フィルター: 
- status:active → 緑
- status:sleeping → 黄
- status:abandoned → 赤
```

#### 優先度ビュー
```
グループ化: priority
ソート: last_updated
```

## 実装手順

### Step 1: Organization Projectの作成
1. https://github.com/orgs/NSada2025/projects
2. "New project" → "Board"
3. 名前: "WD Projects Health Dashboard"

### Step 2: カスタムフィールド追加
- Health Status (単一選択)
- Last Update (日付)
- 5-min Tasks (テキスト)
- Project Code (テキスト)

### Step 3: 自動化ワークフロー
```yaml
# .github/workflows/update-project-board.yml
name: Update Project Board
on:
  push:
    branches: [main, master]
    
jobs:
  update-board:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/github-script@v6
        with:
          script: |
            // プロジェクトボードの更新
            const projectId = 'PROJECT_ID'
            const itemId = context.payload.repository.name
            
            // 健康状態を更新
            await github.projects.updateProjectItem({
              project_id: projectId,
              item_id: itemId,
              field_id: 'HEALTH_STATUS_FIELD',
              value: 'active'
            })
```

### Step 4: Discordとの連携
```javascript
// Discord通知
const webhook = process.env.DISCORD_WEBHOOK
const message = {
  embeds: [{
    title: "プロジェクト健康レポート",
    fields: [
      {name: "Active", value: activeCount},
      {name: "Sleeping", value: sleepingCount},
      {name: "Abandoned", value: abandonedCount}
    ]
  }]
}
```

## ベストプラクティス

### 1. ラベルの活用
- `5min-task`: すぐできるタスク
- `help-wanted`: 協力者募集
- `good-first-issue`: 初心者向け

### 2. マイルストーンの設定
- 構想フェーズ
- 実装フェーズ
- リリース準備

### 3. 自動化ルール
- PR作成時 → "In Review"
- PR マージ時 → "Done"
- 30日更新なし → "Sleeping"

## 統合後の効果

1. **可視性向上**: 全プロジェクトの状態が一目瞭然
2. **モチベーション維持**: 進捗の可視化でやる気UP
3. **効率化**: 次に触るプロジェクトが明確に
4. **協働促進**: チームメンバーとのタスク共有

## 次のステップ

1. テストプロジェクトでの試験運用
2. 全WDプロジェクトへの展開
3. 週次レビューの自動化
4. 四半期レポートの生成