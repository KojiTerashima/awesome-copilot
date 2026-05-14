# ツールスパン

## 目的

TOOL スパンは、外部ツールまたは関数の呼び出し (API 呼び出し、データベース クエリ、計算機、カスタム関数) を表します。

## 必須の属性

|属性 |タイプ |説明 |必須 |
| ------------------------- | ------ | ------------------ | ----------- |
| `openinference.span.kind` |文字列 | 「ツール」である必要があります |はい |
| `tool.name` |文字列 |ツール/機能名 |おすすめ |

## 属性参照

### ツール実行属性

|属性 |タイプ |説明 |
| ------------------ | ------------- | ------------------------------------------ |
| `tool.name` |文字列 |ツール/機能名 |
| `tool.description` |文字列 |ツールの目的/説明 |
| `tool.parameters` |文字列 (JSON) |ツールのパラメーターを定義する JSON スキーマ |
| `input.value` |文字列 (JSON) |ツールに渡される実際の入力値 |
| `output.value` |文字列 |ツールの出力/結果 |
| `output.mime_type` |文字列 |結果のコンテンツ タイプ (例: "application/json") |

## 例

### API呼び出しツール```json
{
  "openinference.span.kind": "ツール",
  "tool.name": "get_weather",
  "tool.description": "場所の現在の天気を取得します",
  "tool.parameters": "{\"type\": \"object\", \"properties\": {\"location\": {\"type\": \"string\"}, \"units\": {\"type\": \"string\", \"enum\": [\"celsius\", \"fahrenheit\"]}}, \"required\": [\"場所\"]}",
  "input.value": "{\"location\": \"San Francisco\", \"units\": \"celsius\"}",
  "output.value": "{\"気温\": 18、\"条件\": \"曇り\"}"
}
「」### 計算ツール```json
{
  "openinference.span.kind": "ツール",
  "tool.name": "電卓",
  "tool.description": "数学的計算を実行します",
  "tool.parameters": "{\"type\": \"object\", \"properties\": {\"expression\": {\"type\": \"string\", \"description\": \"評価する数学式\"}}, \"required\": [\"expression\"]}",
  "input.value": "{\"expression\": \"2 + 2\"}",
  "出力.値": "4"
}
「」### データベースクエリツール```json
{
  "openinference.span.kind": "ツール",
  "ツール名": "sql_query",
  "tool.description": "ユーザー データベースに対して SQL クエリを実行します",
  "tool.parameters": "{\"type\": \"object\", \"properties\": {\"query\": {\"type\": \"string\", \"description\": \"実行する SQL クエリ\"}}, \"required\": [\"query\"]}",
  "input.value": "{\"query\": \"SELECT * FROM users WHERE id = 123\"}",
  "output.value": "[{\"id\": 123, \"name\": \"Alice\", \"email\": \"alice@example.com\"}]",
  "output.mime_type": "アプリケーション/json"
}
「」
