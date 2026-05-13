---
name: csharp-xunit
description: 'data-driven test を含む XUnit unit testing の best practice を取得します'
---

# XUnit Best Practices

あなたの目標は、標準的な testing と data-driven testing の両方を含めて、XUnit で効果的な unit test を書けるよう支援することです。

## Project Setup

- `[ProjectName].Tests` という naming convention の別 test project を使う
- Microsoft.NET.Test.Sdk、xunit、xunit.runner.visualstudio package を参照する
- test class は対象 class に対応させる（例: `Calculator` に対して `CalculatorTests`）
- test 実行には .NET SDK の `dotnet test` command を使う

## Test Structure

- test class attribute は不要（MSTest/NUnit とは異なる）
- 単純な test には `[Fact]` attribute による fact-based test を使う
- Arrange-Act-Assert (AAA) pattern に従う
- test 名は `MethodName_Scenario_ExpectedBehavior` pattern を使う
- setup には constructor、teardown には `IDisposable.Dispose()` を使う
- class 内 test 間の共有 context には `IClassFixture<T>` を使う
- 複数 test class 間の共有 context には `ICollectionFixture<T>` を使う

## Standard Tests

- test は単一の behavior に集中させる
- 1 つの test method で複数 behavior をテストしない
- 意図が伝わる明確な assertion を使う
- test case の検証に必要な assertion のみを含める
- test は独立かつ冪等にし、どの順序でも実行できるようにする
- test 同士の依存関係を避ける

## Data-Driven Tests

- `[Theory]` と data source attribute を組み合わせて使う
- inline test data には `[InlineData]` を使う
- method ベースの test data には `[MemberData]` を使う
- class ベースの test data には `[ClassData]` を使う
- `DataAttribute` を実装して custom data attribute を作る
- data-driven test では意味のある parameter 名を使う

## Assertions

- 値の等価比較には `Assert.Equal` を使う
- 参照の等価比較には `Assert.Same` を使う
- Boolean 条件には `Assert.True`/`Assert.False` を使う
- collection には `Assert.Contains`/`Assert.DoesNotContain` を使う
- regex pattern match には `Assert.Matches`/`Assert.DoesNotMatch` を使う
- exception test には `Assert.Throws<T>` または `await Assert.ThrowsAsync<T>` を使う
- より読みやすい assertion のために fluent assertions library の利用を検討する

## Mocking and Isolation

- XUnit と併用して Moq または NSubstitute の利用を検討する
- dependency を mock して test 対象 unit を isolate する
- mocking を容易にするため interface を使う
- 複雑な test setup では DI container の利用を検討する

## Test Organization

- feature または component ごとに test をまとめる
- 分類には `[Trait("Category", "CategoryName")]` を使う
- 共有 dependency を持つ test をまとめるには collection fixture を使う
- test 診断には output helper (`ITestOutputHelper`) の利用を検討する
- fact/theory attribute の `Skip = "reason"` で条件付き skip を行う
