# Issue Fields REST API リファレンス

Issue Fieldsは、組織レベルの課題カスタムメタデータです。すべてのエンドポイントでAPIバージョンヘッダーが必要です：

```
-H "X-GitHub-Api-Version: 2026-03-10"
```

## 組織のIssue Fields一覧取得

```bash
gh api /orgs/{org}/issue-fields \
  -H "X-GitHub-Api-Version: 2026-03-10"
```

フィールドオブジェクトの配列を返します：

```json
[
  {
    "id": "IF_abc123",
    "name": "Priority",
    "content_type": "single_select",
    "options": [
      { "id": "OPT_1", "name": "Critical" },
      { "id": "OPT_2", "name": "High" },
      { "id": "OPT_3", "name": "Medium" },
      { "id": "OPT_4", "name": "Low" }
    ]
  },
  {
    "id": "IF_def456",
    "name": "Due Date",
    "content_type": "date",
    "options": null
  }
]
```

**フィールドタイプ**: `text`, `single_select`, `number`, `date`

**便利なjqフィルター**:

```bash
gh api /orgs/{org}/issue-fields \
  -H "X-GitHub-Api-Version: 2026-03-10" \
  --jq '.[] | {id, name, content_type, options: [.options[]?.name]}'
```

## Issue Field値の取得

```bash
gh api /repos/{owner}/{repo}/issues/{number}/issue-field-values \
  -H "X-GitHub-Api-Version: 2026-03-10"
```

課題の現在のフィールド値を返します。書き込み前に値が既に存在するか確認するために使用してください。

## Issue Field値の書き込み（POST、追加）

既存の他のフィールド値を削除せずに、課題に値を追加します。

**重要**: `owner/repo`ではなく、`repository_id`（整数）を使用します。

```bash
# まずリポジトリIDを取得：
REPO_ID=$(gh api /repos/{owner}/{repo} --jq .id)

# 次に値を書き込み：
echo '[
  {
    "field_id": "IF_abc123",
    "value": "High"
  }
]' | gh api /repositories/$REPO_ID/issues/{number}/issue-field-values \
  -X POST \
  -H "X-GitHub-Api-Version: 2026-03-10" \
  --input -
```

### フィールドタイプ別の値フォーマット

| フィールドタイプ | 値のフォーマット | 例 |
|-----------|-------------|---------|
| text | 文字列 | `"value": "Some text"` |
| single_select | オプション名（文字列） | `"value": "High"` |
| number | 数値 | `"value": 42` |
| date | ISO 8601日付文字列 | `"value": "2025-03-15"` |

**ポイント**: `single_select`の場合、REST APIはオプションの**名前**を文字列として受け入れます。オプションIDを調べる必要はありません。

### 複数フィールドを一度に書き込む

配列内に複数のオブジェクトを渡すことで、一度の呼び出しで複数フィールドを設定できます：

```bash
echo '[
  {"field_id": "IF_abc123", "value": "High"},
  {"field_id": "IF_def456", "value": "2025-06-01"}
]' | gh api /repositories/$REPO_ID/issues/{number}/issue-field-values \
  -X POST \
  -H "X-GitHub-Api-Version: 2026-03-10" \
  --input -
```

## Issue Field値の書き込み（PUT、全置換）

課題のすべてのフィールド値を置き換えます。使用には注意してください。

```bash
echo '[{"field_id": "IF_abc123", "value": "Low"}]' | \
  gh api /repositories/$REPO_ID/issues/{number}/issue-field-values \
    -X PUT \
    -H "X-GitHub-Api-Version: 2026-03-10" \
    --input -
```

**警告**: PUTはリクエストボディに含まれていないフィールド値をすべて削除します。マイグレーションでは他のフィールド値を保持するために常にPOSTを使用してください。

## 権限

- **リポジトリ**: 「Issues」読み書き
- **組織**: 「Issue Fields」読み書き

## レート制限

- 標準レート制限が適用されます（認証ユーザーは5,000リクエスト/時間）
- 連続した高速書き込みでセカンダリレート制限が発動する場合があります
- 推奨: 呼び出し間に100msの遅延、429エラー時は指数的バックオフを行うこと
