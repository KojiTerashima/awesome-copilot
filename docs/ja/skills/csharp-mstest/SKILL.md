---
name: csharp-mstest
description: 'Get best practices for MSTest 3.x/4.x unit testing, including modern assertion APIs and data-driven tests'
---
# MSTest のベスト プラクティス (MSTest 3.x/4.x)

あなたの目標は、最新の API とベスト プラクティスを使用して、最新の MSTest で効果的な単体テストを作成できるように支援することです。

## プロジェクトのセットアップ

- 命名規則 `[ProjectName].Tests` を使用して別のテスト プロジェクトを使用します。
- リファレンス MSTest 3.x+ NuGet パッケージ (アナライザーを含む)
- プロジェクトのセットアップを簡略化するために MSTest.Sdk の使用を検討してください。
- `dotnet test` でテストを実行する

## テストクラスの構造

- テストクラスには `[TestClass]` 属性を使用します
- **パフォーマンスと設計を明確にするために、デフォルトでテスト クラスをシールします**
- テスト メソッドには `[TestMethod]` を使用します (`[DataTestMethod]` よりも優先)
- Arrange-Act-Assert (AAA) パターンに従います
- パターン `MethodName_Scenario_ExpectedBehavior` を使用してテストに名前を付けます```csharp
[TestClass]
public sealed class CalculatorTests
{
    [TestMethod]
    public void Add_TwoPositiveNumbers_ReturnsSum()
    {
        // Arrange
        var calculator = new Calculator();

        // Act
        var result = calculator.Add(2, 3);

        // Assert
        Assert.AreEqual(5, result);
    }
}
```## テストのライフサイクル

- **`[TestInitialize]`** よりもコンストラクターを優先します - `readonly` フィールドを有効にし、標準の C# パターンに従います
- テストが失敗した場合でも実行する必要があるクリーンアップには `[TestCleanup]` を使用します
- 非同期セットアップが必要な場合は、コンストラクターと非同期 `[TestInitialize]` を組み合わせます```csharp
[TestClass]
public sealed class ServiceTests
{
    private readonly MyService _service;  // readonly enabled by constructor

    public ServiceTests()
    {
        _service = new MyService();
    }

    [TestInitialize]
    public async Task InitAsync()
    {
        // Use for async initialization only
        await _service.WarmupAsync();
    }

    [TestCleanup]
    public void Cleanup() => _service.Reset();
}
```### 実行順序

1. **アセンブリの初期化** - `[AssemblyInitialize]` (テスト アセンブリごとに 1 回)
2. **クラスの初期化** - `[ClassInitialize]` (テスト クラスごとに 1 回)
3. **テストの初期化** (すべてのテストメソッド):
   1. コンストラクター
   2. `TestContext` プロパティを設定します
   3.@@コード3@@
4. **テストの実行** - テストメソッドの実行
5. **テストのクリーンアップ** (すべてのテストメソッド):
   1.@@コード4@@
   2. `DisposeAsync` (実装されている場合)
   3. `Dispose` (実装されている場合)
6. **クラスのクリーンアップ** - `[ClassCleanup]` (テスト クラスごとに 1 回)
7. **アセンブリのクリーンアップ** - `[AssemblyCleanup]` (テスト アセンブリごとに 1 回)

## 最新のアサーション API

MSTest は、`Assert`、`StringAssert`、および `CollectionAssert` の 3 つのアサーション クラスを提供します。

### Assert クラス - コア アサーション```csharp
// Equality
Assert.AreEqual(expected, actual);
Assert.AreNotEqual(notExpected, actual);
Assert.AreSame(expectedObject, actualObject);      // Reference equality
Assert.AreNotSame(notExpectedObject, actualObject);

// Null checks
Assert.IsNull(value);
Assert.IsNotNull(value);

// Boolean
Assert.IsTrue(condition);
Assert.IsFalse(condition);

// Fail/Inconclusive
Assert.Fail("Test failed due to...");
Assert.Inconclusive("Test cannot be completed because...");
```### 例外テスト (`[ExpectedException]` よりも優先)```csharp
// Assert.Throws - matches TException or derived types
var ex = Assert.Throws<ArgumentException>(() => Method(null));
Assert.AreEqual("Value cannot be null.", ex.Message);

// Assert.ThrowsExactly - matches exact type only
var ex = Assert.ThrowsExactly<InvalidOperationException>(() => Method());

// Async versions
var ex = await Assert.ThrowsAsync<HttpRequestException>(async () => await client.GetAsync(url));
var ex = await Assert.ThrowsExactlyAsync<InvalidOperationException>(async () => await Method());
```### コレクション アサーション (Assert クラス)```csharp
Assert.Contains(expectedItem, collection);
Assert.DoesNotContain(unexpectedItem, collection);
Assert.ContainsSingle(collection);  // exactly one element
Assert.HasCount(5, collection);
Assert.IsEmpty(collection);
Assert.IsNotEmpty(collection);
```### 文字列アサーション (Assert クラス)```csharp
Assert.Contains("expected", actualString);
Assert.StartsWith("prefix", actualString);
Assert.EndsWith("suffix", actualString);
Assert.DoesNotStartWith("prefix", actualString);
Assert.DoesNotEndWith("suffix", actualString);
Assert.MatchesRegex(@"\d{3}-\d{4}", phoneNumber);
Assert.DoesNotMatchRegex(@"\d+", textOnly);
```### 比較アサーション```csharp
Assert.IsGreaterThan(lowerBound, actual);
Assert.IsGreaterThanOrEqualTo(lowerBound, actual);
Assert.IsLessThan(upperBound, actual);
Assert.IsLessThanOrEqualTo(upperBound, actual);
Assert.IsInRange(actual, low, high);
Assert.IsPositive(number);
Assert.IsNegative(number);
```### 型アサーション```csharp
// MSTest 3.x - uses out parameter
Assert.IsInstanceOfType<MyClass>(obj, out var typed);
typed.DoSomething();

// MSTest 4.x - returns typed result directly
var typed = Assert.IsInstanceOfType<MyClass>(obj);
typed.DoSomething();

Assert.IsNotInstanceOfType<WrongType>(obj);
```### Assert.That (MSTest 4.0+)```csharp
Assert.That(result.Count > 0);  // Auto-captures expression in failure message
```### StringAssert クラス

> **注意:** 可能な場合は、`Assert` クラスと同等のものを優先してください (例: `StringAssert.Contains(actual, "expected")` よりも `Assert.Contains("expected", actual)`)。```csharp
StringAssert.Contains(actualString, "expected");
StringAssert.StartsWith(actualString, "prefix");
StringAssert.EndsWith(actualString, "suffix");
StringAssert.Matches(actualString, new Regex(@"\d{3}-\d{4}"));
StringAssert.DoesNotMatch(actualString, new Regex(@"\d+"));
```### CollectionAssert クラス

> **注意:** 可能な場合は、`Assert` クラスと同等のものを優先します (例: `Assert.Contains`)。```csharp
// Containment
CollectionAssert.Contains(collection, expectedItem);
CollectionAssert.DoesNotContain(collection, unexpectedItem);

// Equality (same elements, same order)
CollectionAssert.AreEqual(expectedCollection, actualCollection);
CollectionAssert.AreNotEqual(unexpectedCollection, actualCollection);

// Equivalence (same elements, any order)
CollectionAssert.AreEquivalent(expectedCollection, actualCollection);
CollectionAssert.AreNotEquivalent(unexpectedCollection, actualCollection);

// Subset checks
CollectionAssert.IsSubsetOf(subset, superset);
CollectionAssert.IsNotSubsetOf(notSubset, collection);

// Element validation
CollectionAssert.AllItemsAreInstancesOfType(collection, typeof(MyClass));
CollectionAssert.AllItemsAreNotNull(collection);
CollectionAssert.AllItemsAreUnique(collection);
```## データ駆動型テスト

### データ行```csharp
[TestMethod]
[DataRow(1, 2, 3)]
[DataRow(0, 0, 0, DisplayName = "Zeros")]
[DataRow(-1, 1, 0, IgnoreMessage = "Known issue #123")]  // MSTest 3.8+
public void Add_ReturnsSum(int a, int b, int expected)
{
    Assert.AreEqual(expected, Calculator.Add(a, b));
}
```### 動的データ

データ ソースは次のいずれかのタイプを返すことができます。

- `IEnumerable<(T1, T2, ...)>` (ValueTuple) - **推奨**、タイプ セーフティを提供します (MSTest 3.7+)
- `IEnumerable<Tuple<T1, T2, ...>>` - タイプ セーフティを提供します
- `IEnumerable<TestDataRow>` - タイプ セーフティとテスト メタデータ (表示名、カテゴリ) の制御を提供します。
- `IEnumerable<object[]>` - **最も好ましくない**、タイプ セーフティなし

> **注:** 新しいテスト データ メソッドを作成する場合は、`IEnumerable<object[]>` よりも `ValueTuple` または `TestDataRow` を優先してください。 `object[]` アプローチではコンパイル時の型チェックが行われないため、型の不一致による実行時エラーが発生する可能性があります。```csharp
[TestMethod]
[DynamicData(nameof(TestData))]
public void DynamicTest(int a, int b, int expected)
{
    Assert.AreEqual(expected, Calculator.Add(a, b));
}

// ValueTuple - preferred (MSTest 3.7+)
public static IEnumerable<(int a, int b, int expected)> TestData =>
[
    (1, 2, 3),
    (0, 0, 0),
];

// TestDataRow - when you need custom display names or metadata
public static IEnumerable<TestDataRow<(int a, int b, int expected)>> TestDataWithMetadata =>
[
    new((1, 2, 3)) { DisplayName = "Positive numbers" },
    new((0, 0, 0)) { DisplayName = "Zeros" },
    new((-1, 1, 0)) { DisplayName = "Mixed signs", IgnoreMessage = "Known issue #123" },
];

// IEnumerable<object[]> - avoid for new code (no type safety)
public static IEnumerable<object[]> LegacyTestData =>
[
    [1, 2, 3],
    [0, 0, 0],
];
```## テストコンテキスト

`TestContext` クラスは、テスト実行情報、キャンセル サポート、および出力メソッドを提供します。
完全なリファレンスについては、[TestContext ドキュメント](https://learn.microsoft.com/dotnet/core/testing/unit-testing-mstest-writing-tests-testcontext) を参照してください。

### TestContext へのアクセス```csharp
// Property (MSTest suppresses CS8618 - don't use nullable or = null!)
public TestContext TestContext { get; set; }

// Constructor injection (MSTest 3.6+) - preferred for immutability
[TestClass]
public sealed class MyTests
{
    private readonly TestContext _testContext;

    public MyTests(TestContext testContext)
    {
        _testContext = testContext;
    }
}

// Static methods receive it as parameter
[ClassInitialize]
public static void ClassInit(TestContext context) { }

// Optional for cleanup methods (MSTest 3.6+)
[ClassCleanup]
public static void ClassCleanup(TestContext context) { }

[AssemblyCleanup]
public static void AssemblyCleanup(TestContext context) { }
```### キャンセルトークン

`[Timeout]` との連携キャンセルには常に `TestContext.CancellationToken` を使用します。```csharp
[TestMethod]
[Timeout(5000)]
public async Task LongRunningTest()
{
    await _httpClient.GetAsync(url, TestContext.CancellationToken);
}
```### テスト実行のプロパティ```csharp
TestContext.TestName              // Current test method name
TestContext.TestDisplayName       // Display name (3.7+)
TestContext.CurrentTestOutcome    // Pass/Fail/InProgress
TestContext.TestData              // Parameterized test data (3.7+, in TestInitialize/Cleanup)
TestContext.TestException         // Exception if test failed (3.7+, in TestCleanup)
TestContext.DeploymentDirectory   // Directory with deployment items
```### 出力ファイルと結果ファイル```csharp
// Write to test output (useful for debugging)
TestContext.WriteLine("Processing item {0}", itemId);

// Attach files to test results (logs, screenshots)
TestContext.AddResultFile(screenshotPath);

// Store/retrieve data across test methods
TestContext.Properties["SharedKey"] = computedValue;
```## 高度な機能

### 不安定なテストの再試行 (MSTest 3.9+)```csharp
[TestMethod]
[Retry(3)]
public void FlakyTest() { }
```### 条件付き実行 (MSTest 3.10+)

OS または CI 環境に基づいてテストをスキップまたは実行します。```csharp
// OS-specific tests
[TestMethod]
[OSCondition(OperatingSystems.Windows)]
public void WindowsOnlyTest() { }

[TestMethod]
[OSCondition(OperatingSystems.Linux | OperatingSystems.MacOS)]
public void UnixOnlyTest() { }

[TestMethod]
[OSCondition(ConditionMode.Exclude, OperatingSystems.Windows)]
public void SkipOnWindowsTest() { }

// CI environment tests
[TestMethod]
[CICondition]  // Runs only in CI (default: ConditionMode.Include)
public void CIOnlyTest() { }

[TestMethod]
[CICondition(ConditionMode.Exclude)]  // Skips in CI, runs locally
public void LocalOnlyTest() { }
```### 並列化```csharp
// Assembly level
[assembly: Parallelize(Workers = 4, Scope = ExecutionScope.MethodLevel)]

// Disable for specific class
[TestClass]
[DoNotParallelize]
public sealed class SequentialTests { }
```### 作業項目のトレーサビリティ (MSTest 3.8+)

テストレポートでトレーサビリティを確保するために、テストを作業項目にリンクします。```csharp
// Azure DevOps work items
[TestMethod]
[WorkItem(12345)]  // Links to work item #12345
public void Feature_Scenario_ExpectedBehavior() { }

// Multiple work items
[TestMethod]
[WorkItem(12345)]
[WorkItem(67890)]
public void Feature_CoversMultipleRequirements() { }

// GitHub issues (MSTest 3.8+)
[TestMethod]
[GitHubWorkItem("https://github.com/owner/repo/issues/42")]
public void BugFix_Issue42_IsResolved() { }
```作業項目の関連付けはテスト結果に表示され、次の用途に使用できます。
- テストカバレッジを要件まで追跡する
- バグ修正を回帰テストにリンクする
- CI/CD パイプラインでのトレーサビリティ レポートの生成

## 避けるべきよくある間違い```csharp
// ❌ Wrong argument order
Assert.AreEqual(actual, expected);
// ✅ Correct
Assert.AreEqual(expected, actual);

// ❌ Using ExpectedException (obsolete)
[ExpectedException(typeof(ArgumentException))]
// ✅ Use Assert.Throws
Assert.Throws<ArgumentException>(() => Method());

// ❌ Using LINQ Single() - unclear exception
var item = items.Single();
// ✅ Use ContainsSingle - better failure message
var item = Assert.ContainsSingle(items);

// ❌ Hard cast - unclear exception
var handler = (MyHandler)result;
// ✅ Type assertion - shows actual type on failure
var handler = Assert.IsInstanceOfType<MyHandler>(result);

// ❌ Ignoring cancellation token
await client.GetAsync(url, CancellationToken.None);
// ✅ Flow test cancellation
await client.GetAsync(url, TestContext.CancellationToken);

// ❌ Making TestContext nullable - leads to unnecessary null checks
public TestContext? TestContext { get; set; }
// ❌ Using null! - MSTest already suppresses CS8618 for this property
public TestContext TestContext { get; set; } = null!;
// ✅ Declare without nullable or initializer - MSTest handles the warning
public TestContext TestContext { get; set; }
```## テスト組織

- 機能またはコンポーネントごとにテストをグループ化します
- フィルタリングには `[TestCategory("Category")]` を使用します
- カスタム メタデータには `[TestProperty("Name", "Value")]` を使用します (例: `[TestProperty("Bug", "12345")]`)
- 重要なテストには `[Priority(1)]` を使用します
- 関連する MSTest アナライザーを有効にする (コンストラクター設定の MSTEST0020)

## 嘲笑と孤立

- 依存関係をモックするには Moq または NSubstitute を使用します
- インターフェイスを使用してモックを容易にする
- 依存関係をモックしてテスト対象のユニットを分離する