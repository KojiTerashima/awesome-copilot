---
name: creating-oracle-to-postgres-migration-integration-tests
description: '.NET data access artifact 向けに、Oracle-to-PostgreSQL database migration 中の integration test case を作成します。deterministic な seed data を使い、両 database system での動作整合性を検証する DB-agnostic な xUnit test を生成します。migrated project の integration test を作成するとき、data access layer の test coverage を生成するとき、Oracle-to-PostgreSQL migration の validation test を書くときに使用します。'
---

# Creating Integration Tests for Oracle-to-PostgreSQL Migration

単一の target project にある data access artifact 向けに integration test case を生成します。test は Oracle または PostgreSQL に対して実行したときの動作整合性を検証します。

## Prerequisites

- test project は事前に存在し、compile できる状態である必要があります（別途 scaffold 済み）。
- test を書く前に、既存の base test class と seed manager の convention を読みます。

## Workflow

```
Test Creation:
- [ ] Step 1: Discover the test project conventions
- [ ] Step 2: Identify testable data access artifacts
- [ ] Step 3: Create seed data
- [ ] Step 4: Write test cases
- [ ] Step 5: Review determinism
```

**Step 1: Discover the test project conventions**

base test class、seed manager、project file を読んで、継承パターン、transaction management、seed file convention を理解します。

**Step 2: Identify testable data access artifacts**

対象は target project のみに限定します。database とやり取りする data access method を列挙します。repository、DAO、stored procedure caller、query builder などです。

**Step 3: Create seed data**

- 既存 project の seed file 配置と naming convention に従う
- 可能なら既存の seed file を再利用する
- `TRUNCATE TABLE` は避ける。既存 database data はそのまま保つ
- seed data は commit しない。test は rollback される transaction 内で実行される
- seed data が他の test と衝突しないことを確認する
- assertion が依存する前に seed data を load し、検証する

**Step 4: Write test cases**

- 自動 transaction create/rollback を得るため、base test class を継承する
- platform 固有の message ではなく、logical output（rows、columns、counts、error types）を assert する
- 具体的な期待値を assert する。seed data から具体値が得られるときに、単に non-null や non-empty だけを assert してはならない
- 存在しない code path をテストしたり、発生しえない behavior を assert したりしない
- 同じ method を対象とする test 間で冗長な assertion を避ける

**Step 5: Review determinism**

non-null 値に対するすべての assertion を再確認します。seed data に対して deterministic であることを確認し、test の制御外にある database state に依存する assertion は修正します。

## Key Constraints

- **Oracle is the golden source** — test は Oracle の期待される behavior を記録する
- **DB-agnostic assertions** — assertion に platform 固有の error message や syntax を含めない
- **Seed only against Oracle** — test project 自体は後で PostgreSQL に migration される
- **Scoped to one project** — target project 外の artifact に対する test は作成しない
