---
name: java-junit
description: 'JUnit 5のユニットテストにおけるベストプラクティスを取得、データ駆動テストも含む'
---

# JUnit 5+ ベストプラクティス

JUnit 5を使った効果的なユニットテストの作成を支援します。標準的なテストとデータ駆動テストの両方のアプローチをカバーします。

## プロジェクトセットアップ

- 標準的なMavenまたはGradleのプロジェクト構成を使用してください。
- テストソースコードは `src/test/java` に配置します。
- パラメータ化テスト用に `junit-jupiter-api`、`junit-jupiter-engine`、`junit-jupiter-params` の依存関係を含めます。
- ビルドツールのコマンドでテストを実行します：`mvn test` または `gradle test`。

## テスト構造

- テストクラス名は `Test` サフィックスを付けます。例：`Calculator` クラスのテストは `CalculatorTest`。
- テストメソッドには `@Test` を使用します。
- Arrange-Act-Assert (AAA) パターンに従います。
- テスト名は `methodName_should_expectedBehavior_when_scenario` のような説明的な命名規則を使います。
- 各テストのセットアップと後片付けには `@BeforeEach` と `@AfterEach` を使います。
- クラス単位のセットアップと後片付けには `@BeforeAll` と `@AfterAll` を使います（staticメソッドである必要があります）。
- テストクラスやメソッドには `@DisplayName` を使って人間に読みやすい名前を付けます。

## 標準テスト

- テストは単一の振る舞いに集中させます。
- 1つのテストメソッドで複数の条件をテストするのは避けます。
- テストは独立していて冪等（どの順番でも実行可能）であるべきです。
- テスト間の依存関係は避けてください。

## データ駆動（パラメータ化）テスト

- メソッドをパラメータ化テストとしてマークするには `@ParameterizedTest` を使います。
- 単純なリテラル値（文字列、整数など）には `@ValueSource` を使います。
- テスト引数を `Stream` や `Collection` などで提供するファクトリメソッドには `@MethodSource` を使います。
- インラインのカンマ区切り値には `@CsvSource` を使います。
- クラスパス上のCSVファイルを使うには `@CsvFileSource` を使います。
- 列挙型定数を使うには `@EnumSource` を使います。

## アサーション

- `org.junit.jupiter.api.Assertions` の静的メソッド（例：`assertEquals`、`assertTrue`、`assertNotNull`）を使います。
- より流暢で読みやすいアサーションにはAssertJのようなライブラリ（`assertThat(...).is...`）の使用を検討してください。
- 例外のテストには `assertThrows` または `assertDoesNotThrow` を使います。
- 関連するアサーションは `assertAll` でまとめて、すべてのアサーションがチェックされるようにします。
- 失敗時の明確さのためにアサーションには説明的なメッセージを付けます。

## モックと分離

- 依存関係のモックオブジェクト作成にはMockitoのようなモッキングフレームワークを使います。
- Mockitoの `@Mock` と `@InjectMocks` アノテーションを使ってモック作成と注入を簡素化します。
- モック化を容易にするためにインターフェースを利用します。

## テストの整理

- 機能やコンポーネントごとにパッケージでテストをグループ化します。
- テストのカテゴリ分けには `@Tag` を使います（例：`@Tag("fast")`、`@Tag("integration")`）。
- 必要に応じてテスト実行順序を制御するには `@TestMethodOrder(MethodOrderer.OrderAnnotation.class)` と `@Order` を使います。
- 一時的にテストメソッドやクラスをスキップするには `@Disabled` を使い、理由を記述します。
- より良い整理と構造化のためにネストされた内部クラスでテストをグループ化するには `@Nested` を使います。
