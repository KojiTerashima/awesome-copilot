---
name: csharp-nunit
description: 'data-driven test を含む NUnit unit testing の best practice を取得します'
---

# NUnit Best Practices

あなたの目標は、標準的な testing と data-driven testing の両方を含めて、NUnit で効果的な unit test を書けるよう支援することです。

## Project Setup

- `[ProjectName].Tests` という naming convention の別 test project を使う
- Microsoft.NET.Test.Sdk、NUnit、NUnit3TestAdapter package を参照する
- test class は対象 class に対応させる（例: `Calculator` に対して `CalculatorTests`）
- test 実行には .NET SDK の `dotnet test` command を使う

## Test Structure

- test class には `[TestFixture]` attribute を付ける
- test method には `[Test]` attribute を使う
- Arrange-Act-Assert (AAA) pattern に従う
- test 名は `MethodName_Scenario_ExpectedBehavior` pattern を使う
- test ごとの setup/teardown には `[SetUp]` と `[TearDown]` を使う
- class ごとの setup/teardown には `[OneTimeSetUp]` と `[OneTimeTearDown]` を使う
- assembly レベルの setup/teardown には `[SetUpFixture]` を使う

## Standard Tests

- test は単一の behavior に集中させる
- 1 つの test method で複数 behavior をテストしない
- 意図が伝わる明確な assertion を使う
- test case の検証に必要な assertion のみを含める
- test は独立かつ冪等にし、どの順序でも実行できるようにする
- test 同士の依存関係を避ける

## Data-Driven Tests

- inline test data には `[TestCase]` を使う
- programmatically generated な test data には `[TestCaseSource]` を使う
- 単純な parameter の組み合わせには `[Values]` を使う
- property または method ベースの data source には `[ValueSource]` を使う
- ランダムな数値 test 値には `[Random]` を使う
- 連番の数値 test 値には `[Range]` を使う
- 複数 parameter の組み合わせには `[Combinatorial]` または `[Pairwise]` を使う

## Assertions

- `Assert.That` と constraint model を使う（推奨される NUnit style）
- `Is.EqualTo`、`Is.SameAs`、`Contains.Item` などの constraint を使う
- 単純な値の等価比較には `Assert.AreEqual` を使う（classic style）
- collection 比較には `CollectionAssert` を使う
- string 固有の assertion には `StringAssert` を使う
- exception test には `Assert.Throws<T>` または `Assert.ThrowsAsync<T>` を使う
- failure 時に意図がわかるよう、説明的な message を assertion に付ける

## Mocking and Isolation

- NUnit と併用して Moq または NSubstitute の利用を検討する
- dependency を mock して test 対象 unit を isolate する
- mocking を容易にするため interface を使う
- 複雑な test setup では DI container の利用を検討する

## Test Organization

- feature または component ごとに test をまとめる
- `[Category("CategoryName")]` で category を使う
- 必要な場合のみ `[Order]` で test 実行順を制御する
- `[Author("DeveloperName")]` で ownership を示す
- `[Description]` で追加の test 情報を記述する
- 自動実行すべきでない test には `[Explicit]` を検討する
- 一時的に skip する test には `[Ignore("Reason")]` を使う
