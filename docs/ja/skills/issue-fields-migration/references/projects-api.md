# Projects V2 APIリファレンス（マイグレーション用）

このリファレンスは、フィールドマイグレーションに必要なProjects V2 APIのサブセット、すなわちプロジェクトフィールドの検出とアイテム値の読み取りについて説明します。

## プロジェクトフィールドの一覧取得

### MCPツール経由

```
mcp__github__projects_list(
  owner: "{org}",
  project_number: {n},
  method: "list_project_fields"
)
```

### GraphQL経由

```bash
gh api graphql -f query='
  query {
    organization(login: "ORG") {
      projectV2(number: N) {
        fields(first: 30) {
          pageInfo { hasNextPage endCursor }
          nodes {
            ... on ProjectV2Field {
              id
              name
              dataType
            }
            ... on ProjectV2SingleSelectField {
              id
              name
              dataType
              options { id name }
            }
            ... on ProjectV2IterationField {
              id
              name
              dataType
            }
          }
        }
      }
    }
  }'
```

### フィールドデータタイプ

| dataType | 説明 | マイグレーション先 |
|----------|-------------|-------------|
| TEXT | 自由形式テキスト | `text` イシューフィールド |
| SINGLE_SELECT | オプション付きドロップダウン | `single_select` イシューフィールド |
| NUMBER | 数値 | `number` イシューフィールド |
| DATE | 日付値 | `date` イシューフィールド |
| ITERATION | スプリント／イテレーションサイクル | 対応なし（スキップ） |

## プロジェクトアイテムの一覧取得（フィールド値付き）

### MCPツール経由

```
mcp__github__projects_list(
  owner: "{org}",
  project_number: {n},
  method: "list_project_items"
)
```

ページネーションされた結果を返します。各アイテムには以下が含まれます：
- アイテムタイプ（ISSUE、DRAFT_ISSUE、PULL_REQUEST）
- コンテンツ参照（リポジトリオーナー、リポジトリ名、イシュー番号）
- すべてのプロジェクトフィールドのフィールド値

### GraphQL経由

```bash
gh api graphql -f query='
  query($cursor: String) {
    organization(login: "ORG") {
      projectV2(number: N) {
        items(first: 100, after: $cursor) {
          pageInfo { hasNextPage endCursor }
          nodes {
            type
            content {
              ... on Issue {
                number
                repository { nameWithOwner }
              }
            }
            fieldValues(first: 20) {
              pageInfo { hasNextPage endCursor }
              nodes {
                ... on ProjectV2ItemFieldTextValue { text field { ... on ProjectV2Field { name } } }
                ... on ProjectV2ItemFieldSingleSelectValue { name field { ... on ProjectV2SingleSelectField { name } } }
                ... on ProjectV2ItemFieldNumberValue { number field { ... on ProjectV2Field { name } } }
                ... on ProjectV2ItemFieldDateValue { date field { ... on ProjectV2Field { name } } }
              }
            }
          }
        }
      }
    }
  }' -f cursor="$CURSOR"
```

### マイグレーションに関する重要な注意点

- **ページネーション**：プロジェクトには最大10,000件のアイテムが存在する可能性があります。常に`pageInfo.hasNextPage`と`pageInfo.endCursor`を使ってページネーションしてください。
- **ドラフトアイテム**：`type: DRAFT_ISSUE`のアイテムは実際のイシューが紐づいていません。マイグレーション時にはこれらをスキップしてください。
- **プルリクエスト**：`type: PULL_REQUEST`のアイテムはPRでありイシューではありません。イシューフィールドはイシューにのみ適用されるため、これらもスキップしてください。
- **クロスリポジトリ**：単一のプロジェクトに複数リポジトリのイシューが含まれることがあります。リポジトリごとにアイテムをグループ化してリポジトリIDの一括取得を行ってください。
- **フィールド値のアクセス**：各フィールド値ノードのタイプは異なります（`ProjectV2ItemFieldTextValue`、`ProjectV2ItemFieldSingleSelectValue`など）。各タイプごとに適切に処理してください。
