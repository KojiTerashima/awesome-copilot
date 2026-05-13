---
name: creating-oracle-to-postgres-migration-bug-report
description: 'Oracle から PostgreSQL への移行中に見つかった不具合について、構造化されたバグレポートを作成します。Oracle と PostgreSQL の挙動差を、深刻度、根本原因、対処手順を備えた実行可能なバグレポートとして文書化する場合に使用します。'
---

# Oracle から PostgreSQL への移行向けバグレポート作成

## 使うタイミング

- Oracle と PostgreSQL の挙動差によって生じた不具合を文書化する場合
- Oracle から PostgreSQL への移行プロジェクト向けのバグレポートを作成またはレビューする場合

## バグレポート形式

[references/BUG-REPORT-TEMPLATE.md](references/BUG-REPORT-TEMPLATE.md) の template を使用します。各レポートには次を含めてください。

- **Status**: ✅ RESOLVED、⛔ UNRESOLVED、または ⏳ IN PROGRESS
- **Component**: 影響を受ける endpoint、repository、または stored procedure
- **Test**: 関連する automated test 名
- **Severity**: Low / Medium / High / Critical。影響範囲に基づいて判断する
- **Problem**: 期待される Oracle の挙動と、観測された PostgreSQL の挙動
- **Scenario**: seed data、操作、期待結果、実際の結果を含む順序付きの再現手順
- **Root Cause**: 不具合を引き起こしている、Oracle と PostgreSQL の具体的な挙動差
- **Solution**: 変更済みまたは必要な変更内容。明示的な file path を含める
- **Validation**: 両 database で修正を確認する手順

## Oracle から PostgreSQL への移行ガイダンス

- **Oracle is the source of truth**。期待される挙動は Oracle の baseline に基づいて記述する
- data layer のニュアンスを明示する: empty string と NULL、type coercion の厳格さ、collation、sequence 値、time zone、padding、constraint
- client code の変更は、正しい挙動のために必要な場合を除いて避ける。提案する場合は、明確に文書化して正当化する

## 文体

- 平易な言葉、短い文、明確な次の action
- 現在形または過去形を一貫して使う
- 手順と validation には bullet と numbered list を使う
- 証拠として最小限の SQL 抜粋と log を含める。機密データは除外し、再現可能な snippet に保つ
- 既存の runtime / language version に従い、推測ベースの修正は避ける

## ファイル名規則

バグレポートは `BUG_REPORT_<DescriptiveSlug>.md` として保存します。`<DescriptiveSlug>` には短い PascalCase の識別子を使います（例: `EmptyStringNullHandling`、`RefCursorUnwrapFailure`）。
