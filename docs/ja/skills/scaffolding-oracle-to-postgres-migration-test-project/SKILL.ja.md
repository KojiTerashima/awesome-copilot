---
name: scaffolding-oracle-to-postgres-migration-test-project
description: 'Scaffolds an xUnit integration test project for validating Oracle-to-PostgreSQL database migration behavior in .NET solutions. Creates the test project, transaction-rollback base class, and seed data manager. Use when setting up test infrastructure before writing migration integration tests, or when a test project is needed for Oracle-to-PostgreSQL validation.'
---
# Oracle から PostgreSQL への移行のための統合テスト プロジェクトのスキャフォールディング

単一のターゲット プロジェクトのトランザクション管理とシード データ インフラストラクチャを備えた、コンパイル可能な空の xUnit テスト プロジェクトを作成します。テストを作成する前に、プロジェクトごとに 1 回実行します。

## ワークフロー```
Progress:
- [ ] Step 1: Inspect the target project
- [ ] Step 2: Create the xUnit test project
- [ ] Step 3: Implement transaction-rollback base class
- [ ] Step 4: Implement seed data manager
- [ ] Step 5: Verify the project compiles
```**ステップ 1: 対象プロジェクトを検査する**

ターゲット プロジェクトの `.csproj` を読み取り、.NET バージョンと既存のパッケージ参照を確認します。これらのバージョンは正確に一致します。アップグレードしないでください。

**ステップ 2: xUnit テスト プロジェクトを作成する**

- テスト対象のアプリケーションと同じ .NET バージョンをターゲットにします。
- Oracle データベース接続と xUnit 用の NuGet パッケージを追加します。
- プロジェクト参照はターゲット プロジェクトにのみ追加します。他のアプリケーション プロジェクトには追加しません。
- Oracle データベース接続用に構成された `appsettings.json` を追加します。

**ステップ 3: トランザクション ロールバック基本クラスを実装する**

- 各テストの前にトランザクションを開き、テスト後にロールバックする基本テスト クラスを作成します。
- すべての例外をキャッチして処理し、ロールバックを保証します。
- すべての下流のテスト クラスでパターンを継承できるようにします。

**ステップ 4: シード データ マネージャーを実装する**

- トランザクション スコープ内でテスト データをロードするためのグローバル シード マネージャーを作成します。
- シード データをコミットしないでください。トランザクションは各テスト後にロールバックされます。
- `TRUNCATE TABLE` は使用しないでください。既存のデータベース データを保持します。
- 利用可能な場合は、既存のシード ファイルを再利用します。
- ダウンストリームのテスト作成時に従うシード ファイルの場所の命名規則を確立します。

**ステップ 5: プロジェクトがコンパイルされていることを確認します**

テスト プロジェクトをビルドし、完了する前にエラーなしでコンパイルされることを確認します。

## 主要な制約

- Oracle は黄金の動作ソースです。最初に Oracle の足場を構築します。
- 既存の .NET および C# バージョンを維持します。新しい言語やランタイム機能を導入しないでください。
- 出力はインフラストラクチャのみを備えた空のテスト プロジェクトであり、テスト ケースはありません。