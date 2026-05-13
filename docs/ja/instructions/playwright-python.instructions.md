---
description: '公式ドキュメントに基づく Playwright Python の AI テスト生成 instruction'
applyTo: '**'
---

# Playwright Python テスト生成 instruction

## テスト作成ガイドライン

### コード品質基準
- **Locators**: 耐久性とアクセシビリティのため、user-facing で role-based な locator (`get_by_role`, `get_by_label`, `get_by_text`) を優先します。
- **Assertions**: `expect` API による自動リトライ付きの web-first assertion を使います (例: `expect(page).to_have_title(...)`)。要素の可視性変化を特にテストする場合を除き、`expect(locator).to_be_visible()` は避けてください。通常は、より具体的な assertion の方が信頼性に優れます。
- **Timeouts**: Playwright に組み込まれている auto-waiting 機構に依存します。ハードコードした待機や、既定 timeout の増加は避けてください。
- **Clarity**: 意図が明確に伝わる説明的なテスト名 (例: `def test_navigation_link_works():`) を使います。コメントは複雑なロジックを説明する場合にのみ追加し、"click a button" のような単純な操作の説明には使いません。

### テスト構造
- **Imports**: すべてのテスト ファイルは `from playwright.sync_api import Page, expect` で始めます。
- **Fixtures**: ブラウザー ページを操作するため、テスト関数の引数として `page: Page` fixture を使います。
- **Setup**: `page.goto()` のようなナビゲーション手順は各テスト関数の先頭に置きます。複数テストで共有する setup 操作には、標準の Pytest fixture を使います。

### ファイル構成
- **Location**: テスト ファイルは専用の `tests/` ディレクトリに置くか、既存のプロジェクト構成に従います。
- **Naming**: Pytest に検出されるよう、テスト ファイルは `test_<feature-or-page>.py` 命名規則に従う必要があります。
- **Scope**: 主要なアプリケーション機能またはページごとに、テスト ファイルを 1 つにすることを目指します。

## Assertion のベスト プラクティス
- **Element Counts**: locator で見つかった要素数の検証には `expect(locator).to_have_count()` を使います。
- **Text Content**: 完全一致のテキストには `expect(locator).to_have_text()`、部分一致には `expect(locator).to_contain_text()` を使います。
- **Navigation**: ページ URL の検証には `expect(page).to_have_url()` を使います。
- **Assertion Style**: UI テストの信頼性を高めるため、`assert` より `expect` を優先します。


## 例

```python
import re
import pytest
from playwright.sync_api import Page, expect

@pytest.fixture(scope="function", autouse=True)
def before_each_after_each(page: Page):
    # Go to the starting url before each test.
    page.goto("https://playwright.dev/")

def test_main_navigation(page: Page):
    expect(page).to_have_url("https://playwright.dev/")

def test_has_title(page: Page):
    # Expect a title "to contain" a substring.
    expect(page).to_have_title(re.compile("Playwright"))

def test_get_started_link(page: Page):
    page.get_by_role("link", name="Get started").click()

    # Expects page to have a heading with the name of Installation.
    expect(page.get_by_role("heading", name="Installation")).to_be_visible()
```

## テスト実行戦略

1. **Execution**: テストは terminal から `pytest` コマンドで実行します。
2. **Debug Failures**: テスト失敗を分析し、根本原因を特定します。
