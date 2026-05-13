# バグレポート template

Oracle から PostgreSQL への移行不具合のバグレポートを作成する際は、この template を使います。

## ファイル名形式

```
BUG_REPORT_<DescriptiveSlug>.md
```

## Template Structure

```markdown
# バグレポート: <Title>

**Status:** ✅ RESOLVED | ⛔ UNRESOLVED | ⏳ IN PROGRESS
**Component:** <高レベルの component/endpoint と主要な method>
**Test:** <関連する automated test 名>
**Severity:** Low | Medium | High | Critical

---

## Problem

<観測された誤った挙動。期待される挙動（Oracle baseline）
と、実際の挙動（PostgreSQL）を対比して記載する。
具体的かつ事実ベースで書く。>

## Scenario

<不具合を再現する順序付き手順。以下を含める:
1. 前提条件と seed data
2. 正確な操作または API call
3. 期待結果（Oracle）
4. 実際の結果（PostgreSQL）>

## Root Cause

<最小限で具体的な技術的原因。該当する Oracle / PostgreSQL
の挙動差（例: empty string と NULL、type coercion の厳格さ）を参照する。>

## Solution

<変更済みまたは必要な変更内容。data access layer の変更、
tracking flag、client code 修正があれば明示する。変更が
すでに適用済みか、まだ必要かも記載する。>

## Validation

<修正を確認する passing test または manual check の bullet list:
- Oracle と PostgreSQL の両方で再現手順を再実行する
- row/column の出力を比較する
- error handling の整合性を確認する>

## Files Modified

<relative file path と変更目的の短い説明を bullet list で記載する:
- `src/DataAccess/FooRepository.cs` — empty string parameter に対する明示的な NULL check を追加>

## Notes / Next Steps

<follow-up、environment 上の注意点、risk、他の修正への依存関係。>
```

## Status Values

| Status | Meaning |
|--------|---------|
| ✅ RESOLVED | 不具合が修正され、検証済みである |
| ⛔ UNRESOLVED | 不具合がまだ対処されていない |
| ⏳ IN PROGRESS | 不具合を調査中、または修正作業中である |

## 文体ルール

- 表現は簡潔かつ事実ベースに保つ
- 現在形または過去形を一貫して使う
- 手順と validation には bullet と numbered list を優先する
- data layer のニュアンス（tracking、padding、constraint）を明示する
- 既存の runtime / language version に従い、推測ベースの修正は避ける
- 証拠として最小限の SQL 抜粋と log を含め、機密データは省く
