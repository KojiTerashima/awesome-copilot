---
name: reviewing-oracle-to-postgres-migration
description: 'Identifies Oracle-to-PostgreSQL migration risks by cross-referencing code against known behavioral differences (empty strings, refcursors, type coercion, sorting, timestamps, concurrent transactions, etc.). Use when planning a database migration, reviewing migration artifacts, or validating that integration tests cover Oracle/PostgreSQL differences.'
---
# Oracle から PostgreSQL データベースへの移行

移行のリスクを明らかにし、`references/` フォルダーに文書化されている既知の Oracle/PostgreSQL の動作の違いに対して移行作業を検証します。

## いつ使用するか

1. **計画** — プロシージャ、トリガー、クエリ、または Refcursor クライアントでの移行作業を開始する前に。どの参照情報が適用されるかを特定して、リスクに事前に対処します。
2. **検証中** — 移行作業が完了したら、該当するすべての洞察が対処され、統合テストで新しい PostgreSQL セマンティクスがカバーされていることを確認します。

## ワークフロー

タスクのタイプを決定します。

**移行を計画していますか?** リスク評価ワークフローに従います。
**完了した作業を検証していますか?** 検証ワークフローに従います。

### リスク評価ワークフロー (計画)```
Risk Assessment:
- [ ] Step 1: Identify the migration scope
- [ ] Step 2: Screen each insight for applicability
- [ ] Step 3: Document risks and recommended actions
```**ステップ 1: 移行範囲を特定する**

影響を受けるデータベース オブジェクト (プロシージャ、トリガー、クエリ、ビュー) とそれらを呼び出すアプリケーション コードをリストします。

**ステップ 2: それぞれの洞察が適用可能かどうかを選別する**

[references/REFERENCE.md](references/REFERENCE.md) の参照インデックスを確認してください。エントリごとに、その洞察によって影響を受けるパターンが移行スコープに含まれているかどうかを判断します。洞察が関連する可能性がある場合にのみ、参照ファイル全体を読んでください。

**ステップ 3: リスクと推奨されるアクションを文書化する**

該当する洞察ごとに、参照ファイルから特定のリスクと推奨される修正パターンをメモします。設計上の決定が必要な洞察にフラグを立てます (Oracle の空の文字列を NULL として保持するセマンティクスを維持するか、PostgreSQL の動作を採用するかなど)。

### 検証ワークフロー (移行後)```
Validation:
- [ ] Step 1: Map the migration artifact
- [ ] Step 2: Cross-check applicable insights
- [ ] Step 3: Verify integration test coverage
- [ ] Step 4: Gate the result
```**ステップ 1: 移行アーティファクトをマッピングする**

Identify the migrated object and summarize the change set.

**ステップ 2: 該当する洞察をクロスチェックします**

[references/REFERENCE.md](references/REFERENCE.md) 内の各参照について、動作またはテスト要件が認識され、移行作業で対処されていることを確認します。

**ステップ 3: 統合テストのカバレッジを確認する**

テストが、適切な洞察 (例外、並べ替え、リカーサーの消費、同時トランザクション、タイムスタンプなど) で強調表示されている正常なパスと失敗シナリオの両方を実行していることを確認します。

**ステップ 4: 結果をゲートする**

Return a checklist asserting each applicable insight was addressed, migration scripts run, and integration tests pass.