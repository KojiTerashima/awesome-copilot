---
description: 'Playwright テスト生成 instruction'
applyTo: '**'
---

## テスト作成ガイドライン

### コード品質基準
- **Locators**: 耐久性とアクセシビリティのため、user-facing で role-based な locator (`getByRole`, `getByLabel`, `getByText` など) を優先します。`test.step()` を使って操作をグループ化し、テストの可読性とレポート品質を高めます。
- **Assertions**: 自動リトライ付きの web-first assertion を使います。これらの assertion は `await` キーワードで始まります (例: `await expect(locator).toHaveText()`)。可視性の変化を特にテストする場合を除き、`expect(locator).toBeVisible()` は避けてください。
- **Timeouts**: Playwright に組み込まれている auto-waiting 機構に依存します。ハードコードした待機や、既定 timeout の増加は避けてください。
- **Clarity**: 意図が明確に伝わる説明的なテスト名と step 名を使います。コメントは複雑なロジックや自明でない操作を説明する場合にのみ追加します。


### テスト構造
- **Imports**: `import { test, expect } from '@playwright/test';` で始めます。
- **Organization**: ある機能に関連するテストは `test.describe()` ブロックの下にまとめます。
- **Hooks**: `describe` ブロック内の全テストに共通する setup 操作 (例: ページ遷移) には `beforeEach` を使います。
- **Titles**: `Feature - Specific action or scenario` のような明確な命名規則に従います。


### ファイル構成
- **Location**: すべてのテスト ファイルは `tests/` ディレクトリに保存します。
- **Naming**: `<feature-or-page>.spec.ts` 形式の命名規則を使います (例: `login.spec.ts`、`search.spec.ts`)。
- **Scope**: 主要なアプリケーション機能またはページごとに、テスト ファイルを 1 つにすることを目指します。

### Assertion のベスト プラクティス
- **UI Structure**: component のアクセシビリティ ツリー構造の検証には `toMatchAriaSnapshot` を使います。これにより、包括的でアクセシブルな snapshot を得られます。
- **Element Counts**: locator で見つかった要素数の検証には `toHaveCount` を使います。
- **Text Content**: 完全一致のテキストには `toHaveText`、部分一致には `toContainText` を使います。
- **Navigation**: 操作後のページ URL の検証には `toHaveURL` を使います。


## テスト構造の例

```typescript
import { test, expect } from '@playwright/test';

test.describe('Movie Search Feature', () => {
  test.beforeEach(async ({ page }) => {
    // Navigate to the application before each test
    await page.goto('https://debs-obrien.github.io/playwright-movies-app');
  });

  test('Search for a movie by title', async ({ page }) => {
    await test.step('Activate and perform search', async () => {
      await page.getByRole('search').click();
      const searchInput = page.getByRole('textbox', { name: 'Search Input' });
      await searchInput.fill('Garfield');
      await searchInput.press('Enter');
    });

    await test.step('Verify search results', async () => {
      // Verify the accessibility tree of the search results
      await expect(page.getByRole('main')).toMatchAriaSnapshot(`
        - main:
          - heading "Garfield" [level=1]
          - heading "search results" [level=2]
          - list "movies":
            - listitem "movie":
              - link "poster of The Garfield Movie The Garfield Movie rating":
                - /url: /playwright-movies-app/movie?id=tt5779228&page=1
                - img "poster of The Garfield Movie"
                - heading "The Garfield Movie" [level=2]
      `);
    });
  });
});
```

## テスト実行戦略

1. **Initial Run**: `npx playwright test --project=chromium` でテストを実行します。
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
