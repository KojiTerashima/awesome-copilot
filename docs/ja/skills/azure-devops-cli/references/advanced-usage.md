# 高度な使い方: 出力、クエリ、パラメーター

## 目次
- [出力形式](#output-formats)
- [JMESPath クエリ](#jmespath-queries)
- [高度な JMESPath クエリ](#advanced-jmespath-queries)
- [グローバル引数](#global-arguments)
- [共通パラメーター](#common-parameters)
- [Git エイリアス](#git-aliases)
- [ヘルプの取得](#getting-help)

---

## 出力形式

すべてのコマンドは複数の出力形式をサポートしています。

```bash
# テーブル形式（人が読みやすい）
az pipelines list --output table

# JSON 形式（デフォルト、機械可読）
az pipelines list --output json

# JSONC（色付き JSON）
az pipelines list --output jsonc

# YAML 形式
az pipelines list --output yaml

# YAMLC（色付き YAML）
az pipelines list --output yamlc

# TSV 形式（タブ区切り値）
az pipelines list --output tsv

# None（出力なし）
az pipelines list --output none
```

## JMESPath クエリ

出力をフィルターして変換します。

```bash
# 名前でフィルター
az pipelines list --query "[?name=='myPipeline']"

# 特定のフィールドを取得
az pipelines list --query "[].{Name:name, ID:id}"

# クエリを連結
az pipelines list --query "[?name.contains('CI')].{Name:name, ID:id}" --output table

# 最初の結果を取得
az pipelines list --query "[0]"

# 上位 N 件を取得
az pipelines list --query "[0:5]"
```

## 高度な JMESPath クエリ

### フィルタリングと並べ替え

```bash
# 複数条件でフィルター
az pipelines list --query "[?name.contains('CI') && enabled==true]"

# ステータスと結果でフィルター
az pipelines runs list --query "[?status=='completed' && result=='succeeded']"

# 日付で並べ替え（降順）
az pipelines runs list --query "sort_by([?status=='completed'], &finishTime | reverse(@))"

# フィルター後に上位 N 件を取得
az pipelines runs list --query "[?result=='succeeded'] | [0:5]"
```

### ネストされたクエリ

```bash
# ネストされたプロパティを抽出
az pipelines show --id $PIPELINE_ID --query "{Name:name, Repo:repository.{Name:name, Type:type}, Folder:folder}"

# ビルド詳細をクエリ
az pipelines build show --id $BUILD_ID --query "{ID:id, Number:buildNumber, Status:status, Result:result, Requested:requestedFor.displayName}"
```

### 複雑なフィルタリング

```bash
# 特定の YAML パスを持つパイプラインを検索
az pipelines list --query "[?process.type.name=='yaml' && process.yamlFilename=='azure-pipelines.yml']"

# 特定のレビュアーによる PR を検索
az repos pr list --query "[?contains(reviewers[?displayName=='John Doe'].displayName, 'John Doe')]"

# 特定のイテレーションと状態を持つワークアイテムを検索
az boards work-item show --id $WI_ID --query "{Title:fields['System.Title'], State:fields['System.State'], Iteration:fields['System.IterationPath']}"
```

### 集計

```bash
# ステータスごとの件数
az pipelines runs list --query "groupBy([?status=='completed'], &[result]) | {Succeeded: [?key=='succeeded'][0].count, Failed: [?key=='failed'][0].count}"

# 一意なレビュアーを取得
az repos pr list --query "unique_by(reviewers[], &displayName)"

# 値を合計
az pipelines runs list --query "[?result=='succeeded'] | [].{Duration:duration} | [0].Duration"
```

### 条件付き変換

```bash
# 日付を整形
az pipelines runs list --query "[].{ID:id, Date:createdDate, Formatted:createdDate | format_datetime(@, 'yyyy-MM-dd HH:mm')}"

# 条件付き出力
az pipelines list --query "[].{Name:name, Status:(enabled ? 'Enabled' : 'Disabled')}"

# デフォルト値付きで抽出
az pipelines show --id $PIPELINE_ID --query "{Name:name, Folder:folder || 'Root', Description:description || 'No description'}"
```

### 複雑なワークフロー

```bash
# 実行時間が最長のビルドを検索
az pipelines build list --query "sort_by([?result=='succeeded'], &queueTime) | reverse(@) | [0:3].{ID:id, Number:buildNumber, Duration:duration}"

# レビュアーごとの PR 統計を取得
az repos pr list --query "groupBy([], &reviewers[].displayName) | [].{Reviewer:@.key, Count:length(@)}"

# 子アイテムが複数あるワークアイテムを検索
az boards work-item relation list --id $PARENT_ID --query "[?rel=='System.LinkTypes.Hierarchy-Forward'] | [].{ChildID:url | split('/', @) | [-1]}"
```

## グローバル引数

すべてのコマンドで使用できます。

| Parameter | Description |
|---|---|
| `--help` / `-h` | コマンドのヘルプを表示 |
| `--output` / `-o` | 出力形式（json, jsonc, none, table, tsv, yaml, yamlc） |
| `--query` | 出力をフィルターするための JMESPath クエリ文字列 |
| `--verbose` | ログの詳細度を上げる |
| `--debug` | すべてのデバッグログを表示 |
| `--only-show-errors` | エラーのみ表示し、警告を抑制 |
| `--subscription` | サブスクリプション名または ID |
| `--yes` / `-y` | 確認プロンプトをスキップ |

## 共通パラメーター

| Parameter | Description |
|---|---|
| `--org` / `--organization` | Azure DevOps 組織の URL（例: `https://dev.azure.com/{org}`） |
| `--project` / `-p` | プロジェクト名または ID |
| `--detect` | git config から組織を自動検出 |
| `--yes` / `-y` | 確認プロンプトをスキップ |
| `--open` | Web ブラウザーでリソースを開く |
| `--subscription` | Azure サブスクリプション（Azure リソース用） |

## Git エイリアス

git エイリアスを有効にした後:

```bash
# Git エイリアスを有効化
az devops configure --use-git-aliases true

# DevOps 操作に Git コマンドを使用
git pr create --target-branch main
git pr list
git pr checkout 123
```

## ヘルプの取得

```bash
# 全般ヘルプ
az devops --help

# 特定のコマンドグループのヘルプ
az pipelines --help
az repos pr --help

# 特定のコマンドのヘルプ
az repos pr create --help

# 例を検索
az find "az repos pr create"
```

