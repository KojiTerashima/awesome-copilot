# チェーンスパン

## 目的

CHAIN スパンは、アプリケーション内のオーケストレーション レイヤー (LangChain チェーン、カスタム ワークフロー、アプリケーション エントリ ポイント) を表します。ルート スパンとしてよく使用されます。

## 必須の属性

|属性 |タイプ |説明 |必須 |
| ------------------------- | ------ | --------------- | -------- |
| `openinference.span.kind` |文字列 | 「チェーン」である必要があります |はい |

## 共通の属性

CHAIN スパンは通常、[ユニバーサル属性](fundamentals-universal-attributes.md) を使用します。

- `input.value` - チェーンへの入力 (ユーザークエリ、リクエストペイロード)
- `output.value` - チェーンからの出力（最終応答）
- `input.mime_type` / `output.mime_type` - フォーマットインジケーター

## 例: ルートチェーン```json
{
  "openinference.span.kind": "チェーン",
  "input.value": "{\"question\": \"フランスの首都はどこですか?\"}",
  "input.mime_type": "アプリケーション/json",
  "output.value": "{\"answer\": \"フランスの首都はパリです。\", \"sources\": [\"doc_123\"]}",
  "output.mime_type": "アプリケーション/json",
  "セッションID": "セッション_abc123",
  "user.id": "user_xyz789"
}
「」## 例: ネストされたサブチェーン```json
{
  "openinference.span.kind": "チェーン",
  "input.value": "この文書の要約: ...",
  "output.value": "このドキュメントでは...について説明します。"
}
「」
