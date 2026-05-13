---
description: 'Playwright .NET テスト生成 instruction'
applyTo: '**'
---

# Playwright .NET テスト生成 instruction

## テスト作成ガイドライン

### コード品質基準

- **Locators**: 耐久性とアクセシビリティのため、user-facing で role-based な locator (`GetByRole`、`GetByLabel`、`GetByText` など) を優先します。`await Test.StepAsync()` を使って操作をグループ化し、テストの可読性とレポート品質を高めます。
- **Assertions**: 自動リトライ付きの web-first assertion を使います。これらの assertion には Playwright assertion の `Expect()` を使います (例: `await Expect(locator).ToHaveTextAsync()`)。可視性の変化を特にテストする場合を除き、可視性チェックは避けてください。
- **Timeouts**: Playwright に組み込まれている auto-waiting 機構に依存します。ハードコードした待機や、既定 timeout の増加は避けてください。
- **Clarity**: 意図が明確に伝わる説明的なテスト名と step 名を使います。コメントは複雑なロジックや自明でない操作を説明する場合にのみ追加します。

### テスト構造

- **Usings**: `using Microsoft.Playwright;` と、MSTest であれば `using Microsoft.Playwright.Xunit;` または `using Microsoft.Playwright.NUnit;` または `using Microsoft.Playwright.MSTest;` で始めます。
- **Organization**: `PageTest` を継承するテスト class を作成し (NUnit、xUnit、MSTest package で利用可能)、または custom fixture を使う xUnit では `IClassFixture<PlaywrightFixture>` を使います。ある機能に関連するテストは同じテスト class にまとめます。
- **Setup**: すべてのテストに共通する setup 操作 (例: ページ遷移) には、`[SetUp]` (NUnit)、`[TestInitialize]` (MSTest)、または constructor 初期化 (xUnit) を使います。
- **Titles**: 適切なテスト attribute (`[Test]` は NUnit、`[Fact]` は xUnit、`[TestMethod]` は MSTest) を使い、C# の命名規則に従った説明的な method 名を付けます (例: `SearchForMovieByTitle`)。

### ファイル構成

- **Location**: すべてのテスト ファイルは `Tests/` ディレクトリに保存するか、機能ごとに整理します。
- **Naming**: `<FeatureOrPage>Tests.cs` 形式の命名規則を使います (例: `LoginTests.cs`、`SearchTests.cs`)。
- **Scope**: 主要なアプリケーション機能またはページごとに、テスト class を 1 つにすることを目指します。

### Assertion のベスト プラクティス

- **UI Structure**: component のアクセシビリティ ツリー構造の検証には `ToMatchAriaSnapshotAsync` を使います。これにより、包括的でアクセシブルな snapshot を得られます。
- **Element Counts**: locator で見つかった要素数の検証には `ToHaveCountAsync` を使います。
- **Text Content**: 完全一致のテキストには `ToHaveTextAsync`、部分一致には `ToContainTextAsync` を使います。
- **Navigation**: 操作後のページ URL の検証には `ToHaveURLAsync` を使います。

## テスト構造の例

```csharp
using Microsoft.Playwright;
using Microsoft.Playwright.Xunit;
using static Microsoft.Playwright.Assertions;

namespace PlaywrightTests;

public class MovieSearchTests : PageTest
{
    public override async Task InitializeAsync()
    {
        await base.InitializeAsync();
        // Navigate to the application before each test
        await Page.GotoAsync("https://debs-obrien.github.io/playwright-movies-app");
    }

    [Fact]
    public async Task SearchForMovieByTitle()
    {
        await Test.StepAsync("Activate and perform search", async () =>
        {
            await Page.GetByRole(AriaRole.Search).ClickAsync();
            var searchInput = Page.GetByRole(AriaRole.Textbox, new() { Name = "Search Input" });
            await searchInput.FillAsync("Garfield");
            await searchInput.PressAsync("Enter");
        });

        await Test.StepAsync("Verify search results", async () =>
        {
            // Verify the accessibility tree of the search results
            await Expect(Page.GetByRole(AriaRole.Main)).ToMatchAriaSnapshotAsync(@"
                - main:
                  - heading ""Garfield"" [level=1]
                  - heading ""search results"" [level=2]
                  - list ""movies"":
                    - listitem ""movie"":
                      - link ""poster of The Garfield Movie The Garfield Movie rating"":
                        - /url: /playwright-movies-app/movie?id=tt5779228&page=1
                        - img ""poster of The Garfield Movie""
                        - heading ""The Garfield Movie"" [level=2]
            ");
        });
    }
}
```

## テスト実行戦略

1. **Initial Run**: `dotnet test` または IDE の test runner を使ってテストを実行します。
2. **Debug Failures**: テスト失敗を分析し、根本原因を特定します。
3. **Iterate**: locator、assertion、テスト ロジックを必要に応じて調整します。
4. **Validate**: テストが安定して成功し、意図した機能をカバーしていることを確認します。
5. **Report**: テスト結果と見つかった問題についてフィードバックを提供します。

## 品質チェックリスト

テストを確定する前に、以下を確認してください。

- [ ] すべての locator がアクセシブルかつ十分に具体的で、strict mode violation を避けている
- [ ] テストが論理的にグループ化され、明確な構造に従っている
- [ ] assertion が意味のあるもので、ユーザーの期待を反映している
- [ ] テストが一貫した命名規則に従っている
- [ ] コードが適切に整形され、必要に応じてコメントされている
