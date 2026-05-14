# エージェントのスパン

AGENT スパンは、自律的な推論ブロック (ReAct エージェント、計画ループ、複数ステップの意思決定) を表します。

**必須:** `openinference.span.kind` = "エージェント"

## 例```json
{
  "openinference.span.kind": "エージェント",
  "input.value": "来週月曜日のニューヨーク行きのフライトを予約します",
  "output.value": "月曜日の午前 9 時に出発する AA123 便を予約しました。"
}
「」
