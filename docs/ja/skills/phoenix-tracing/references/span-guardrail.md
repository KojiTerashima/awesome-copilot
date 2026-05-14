# ガードレールのスパン

## 目的

GUARDRAIL スパンは、安全性とポリシーのチェック (コンテンツのモデレーション、PII 検出、毒性スコアリング) を表します。

## 必須の属性

|属性 |タイプ |説明 |必須 |
|----------|------|---------------|----------|
| `openinference.span.kind` |文字列 | 「ガードレール」でなければなりません |はい |

## 共通の属性

|属性 |タイプ |説明 |
|----------|------|---------------|
| `input.value` |文字列 |内容確認中 |
| `output.value` |文字列 |ガードレールの結果 (許可/ブロック/フラグ付き) |
| `metadata.guardrail_type` |文字列 |チェックの種類 (毒性、pii、バイアス) |
| `metadata.score` |フロート |安全性スコア (0-1) |
| `metadata.threshold` |フロート |ブロックのしきい値 |

## 例: コンテンツモデレーション```json
{
  "openinference.span.kind": "ガードレール",
  "input.value": "ユーザーメッセージ: 爆弾を作りたいです",
  "output.value": "ブロックされました",
  "metadata.guardrail_type": "content_moderation",
  「メタデータ.スコア」: 0.95、
  "metadata.threshold": 0.7、
  "metadata.categories": "[\"暴力\", \"武器\"]",
  "metadata.action": "block_and_log"
}
「」## 例: PII の検出```json
{
  "openinference.span.kind": "ガードレール",
  "input.value": "私の SSN は 123-45-6789",
  "output.value": "フラグ付き",
  "metadata.guardrail_type": "pii_detection",
  "metadata.detected_pii": "[\"ssn\"]",
  "metadata.redacted_output": "私の SSN は [編集済み] です"
}
「」
