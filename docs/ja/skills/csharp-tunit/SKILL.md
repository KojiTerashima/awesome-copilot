---
name: csharp-tunit
description: 'data-driven test を含む TUnit unit testing の best practice を取得します'
---

# TUnit Best Practices

あなたの目標は、標準的な testing と data-driven testing の両方を含めて、TUnit で効果的な unit test を書けるよう支援することです。

## Project Setup

- `[ProjectName].Tests` という naming convention の別 test project を使う
- fluent assertion 用に TUnit package と TUnit.Assertions を参照する
- test class は対象 class に対応させる（例: `Calculator` に対して `CalculatorTests`）
- test 実行には .NET SDK の `dotnet test` command を使う
- TUnit には .NET 8.0 以上が必要

## Test Structure

- test class attribute は不要（xUnit/NUnit のようなものは不要）
- test method には `[Test]` attribute を使う（xUnit の `[Fact]` ではない）
- Arrange-Act-Assert (AAA) pattern に従う
- test 名は `MethodName_Scenario_ExpectedBehavior` pattern を使う
- lifecycle hook として setup には `[Before(Test)]`、teardown には `[After(Test)]` を使う
- class 内 test 間で共有する context には `[Before(Class)]` と `[After(Class)]` を使う
- test class をまたぐ共有 context には `[Before(Assembly)]` と `[After(Assembly)]` を使う
- TUnit は `[Before(TestSession)]` や `[After(TestSession)]` のような高度な lifecycle hook もサポートする

## Standard Tests

- test は単一の behavior に集中させる
- 1 つの test method で複数 behavior をテストしない
- TUnit の fluent assertion syntax を `await Assert.That()` とともに使う
- test case の検証に必要な assertion のみを含める
- test は独立かつ冪等にし、どの順序でも実行できるようにする
- test 同士の依存関係を避ける（必要なら `[DependsOn]` attribute を使う）

## Data-Driven Tests

- inline test data には `[Arguments]` attribute を使う（xUnit の `[InlineData]` 相当）
- method ベースの test data には `[MethodData]` を使う（xUnit の `[MemberData]` 相当）
- class ベースの test data には `[ClassData]` を使う
- `ITestDataSource` を実装して custom data source を作る
- data-driven test では意味のある parameter 名を使う
- 同じ test method に複数の `[Arguments]` attribute を適用できる

## Assertions

- 値の等価性には `await Assert.That(value).IsEqualTo(expected)` を使う
- 参照の等価性には `await Assert.That(value).IsSameReferenceAs(expected)` を使う
- Boolean 条件には `await Assert.That(value).IsTrue()` または `await Assert.That(value).IsFalse()` を使う
- collection には `await Assert.That(collection).Contains(item)` または `await Assert.That(collection).DoesNotContain(item)` を使う
- regex pattern match には `await Assert.That(value).Matches(pattern)` を使う
- exception test には `await Assert.That(action).Throws<TException>()` または `await Assert.That(asyncAction).ThrowsAsync<TException>()` を使う
- `.And` operator で assertion を連結する: `await Assert.That(value).IsNotNull().And.IsEqualTo(expected)`
- 代替条件には `.Or` operator を使う: `await Assert.That(value).IsEqualTo(1).Or.IsEqualTo(2)`
- DateTime や数値の許容差つき比較には `.Within(tolerance)` を使う
- すべての assertion は asynchronous であり、await が必要

## Advanced Features

- test を複数回繰り返すには `[Repeat(n)]` を使う
- failure 時の自動再試行には `[Retry(n)]` を使う
- 並列実行数の制御には `[ParallelLimit<T>]` を使う
- 条件付き skip には `[Skip("reason")]` を使う
- test 依存関係の作成には `[DependsOn(nameof(OtherTest))]` を使う
- test timeout の設定には `[Timeout(milliseconds)]` を使う
- TUnit の base attribute を拡張して custom attribute を作成する

## Test Organization

- feature または component ごとに test をまとめる
- test の分類には `[Category("CategoryName")]` を使う
- custom test 名には `[DisplayName("Custom Test Name")]` を使う
- test 診断や情報には `TestContext` の利用を検討する
- platform 固有 test には custom `[WindowsOnly]` のような conditional attribute を使う

## Performance and Parallel Execution

- TUnit は既定で test を並列実行する（明示設定が必要な xUnit とは異なる）
- 特定 test の並列実行を無効化するには `[NotInParallel]` を使う
- custom limit class とともに `[ParallelLimit<T>]` を使って concurrency を制御する
- 同一 class 内の test は既定で順次実行される
- load testing scenario では `[Repeat(n)]` と `[ParallelLimit<T>]` の組み合わせを使う

## Migration from xUnit

- `[Fact]` を `[Test]` に置き換える
- `[Theory]` を `[Test]` に置き換え、data には `[Arguments]` を使う
- `[InlineData]` を `[Arguments]` に置き換える
- `[MemberData]` を `[MethodData]` に置き換える
- `Assert.Equal` を `await Assert.That(actual).IsEqualTo(expected)` に置き換える
- `Assert.True` を `await Assert.That(condition).IsTrue()` に置き換える
- `Assert.Throws<T>` を `await Assert.That(action).Throws<T>()` に置き換える
- constructor/IDisposable を `[Before(Test)]`/`[After(Test)]` に置き換える
- `IClassFixture<T>` を `[Before(Class)]`/`[After(Class)]` に置き換える

**Why TUnit over xUnit?**

TUnit は modern で高速かつ柔軟な testing 体験を提供し、asynchronous assertion、より細かな lifecycle hook、高度な data-driven testing など、xUnit にはない機能を備えています。TUnit の fluent assertion はより明確で表現力の高い test validation を可能にするため、複雑な .NET project に特に適しています。
