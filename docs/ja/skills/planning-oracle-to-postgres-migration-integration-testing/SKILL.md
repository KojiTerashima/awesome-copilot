---
name: planning-oracle-to-postgres-migration-integration-testing
description: 'Creates an integration testing plan for .NET data access artifacts during Oracle-to-PostgreSQL database migrations. Analyzes a single project to identify repositories, DAOs, and service layers that interact with the database, then produces a structured testing plan. Use when planning integration test coverage for a migrated project, identifying which data access methods need tests, or preparing for Oracle-to-PostgreSQL migration validation.'
---
# Oracle から PostgreSQL への移行のための統合テストの計画

単一のターゲット プロジェクトを分析して、統合テストが必要なデータ アクセス アーティファクトを特定し、構造化された実用的なテスト計画を作成します。

## ワークフロー「」
進捗状況:
- [ ] ステップ 1: データ アクセス アーティファクトを特定する
- [ ] ステップ 2: テストの優先順位を分類する
- [ ] ステップ 3: テスト計画を作成する
「」**ステップ 1: データ アクセス アーティファクトを特定する**

スコープはターゲット プロジェクトのみです。データベースと直接対話するクラスとメソッド、つまりリポジトリ、DAO、ストアド プロシージャ呼び出し元、CRUD 操作を実行するサービス レイヤーを見つけます。

**ステップ 2: テストの優先順位を分類する**

移行リスク別にアーティファクトをランク付けします。単純な CRUD よりも、Oracle 固有の機能 (refcursors、`TO_CHAR`、暗黙的な型強制、`NO_DATA_FOUND`) を使用するメソッドを優先します。

**ステップ 3: テスト計画を作成する**

以下を対象とした値下げ計画を作成します。
- メソッド署名を含むテスト可能なアーティファクトのリスト
- アーティファクトごとに推奨されるテスト ケース
- シードデータの要件
- 検証する必要がある既知のOracle→PostgreSQLの動作の違い

## 出力

計画を次の宛先に書いてください: `.github/oracle-to-postgres-migration/Reports/{TARGET_PROJECT} Integration Testing Plan.md`

## 主要な制約

- **単一プロジェクト スコープ** - ターゲット プロジェクト内のアーティファクトのテストのみを計画します。
- **データベース インタラクションのみ** - データベースに触れないビジネス ロジックをスキップします。
- **Oracle は黄金のソース** — テストでは、PostgreSQL と比較するために Oracle の予想される動作をキャプチャする必要があります。
- **複数接続の利用はありません** - 移行されたアプリケーションはコピーされ、名前が変更されるため (例: `MyApp.Postgres`)、各インスタンスは 1 つのデータベースを対象とします。