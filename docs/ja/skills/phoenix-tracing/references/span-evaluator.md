# 評価者のスパン

## 目的

EVALUATOR スパンは、品質評価操作 (回答の関連性、忠実性、幻覚の検出) を表します。

## 必須の属性

|属性 |タイプ |説明 |必須 |
|----------|------|---------------|----------|
| `openinference.span.kind` |文字列 | 「評価者」である必要があります |はい |

## 共通の属性

|属性 |タイプ |説明 |
|----------|------|---------------|
| `input.value` |文字列 |評価中のコンテンツ |
| `output.value` |文字列 |評価結果（スコア、ラベル、説明） |
| `metadata.evaluator_name` |文字列 |評価者識別子 |
| `metadata.score` |フロート |数値スコア (0-1) |
| `metadata.label` |文字列 |カテゴリラベル (関連/無関係) |

## 例: 回答の関連性```json
{
  "openinference.span.kind": "評価者",
  "input.value": "{\"question\": \"フランスの首都はどこですか?\", \"answer\": \"フランスの首都はパリです。\"}",
  "input.mime_type": "アプリケーション/json",
  "出力.値": "0.95",
  "metadata.evaluator_name": "answer_relevance",
  「メタデータ.スコア」: 0.95、
  "metadata.label": "関連",
  "metadata.explanation": "回答は正しい情報を含む質問に直接対処します"
}
「」## 例: 忠実性チェック```json
{
  "openinference.span.kind": "評価者",
  "input.value": "{\"context\": \"パリはフランスにあります。\", \"answer\": \"パリはフランスの首都です。\"}",
  "input.mime_type": "アプリケーション/json",
  "出力.値": "0.5",
  "metadata.evaluator_name": "誠実さ",
  「メタデータ.スコア」: 0.5、
  "metadata.label": "partially_faithful",
  "metadata.explanation": "回答はパリが首都であるという裏付けのない主張をしています。"
}
「」
