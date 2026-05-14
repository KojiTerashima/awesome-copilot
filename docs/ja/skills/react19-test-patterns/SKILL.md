---
name: react19-test-patterns
description: 'Provides before/after patterns for migrating test files to React 19 compatibility, including act() imports, Simulate removal, and StrictMode call count changes.'
---
# React 19 テスト移行パターン

React 19 で必要なすべてのテスト ファイルの移行に関するリファレンス。

## 優先順位

この順序でテスト ファイルを修正します。各レイヤーは前のレイヤーに依存します。

1. **`act` import** を最初に修正し、他のすべてのブロックを解除します
2. **`Simulate` → `fireEvent`** 動作直後に修正
3. **react-dom/test-utils の完全なクリーンアップ** 残りのインポートを削除します
4. **StrictMode の呼び出し数** は実際の値です。推測ではありません
5. 残りの「行為にラップされていない」警告に対する **非同期行為のラッピング**
6. **カスタム レンダー ヘルパー** テストごとではなく、コードベースごとに 1 回検証します

---

## 1. act() インポートの修正```jsx
// Before  REMOVED in React 19:
import { act } from 'react-dom/test-utils';

// After:
import { act } from 'react';
```他の test-utils インポートと混合した場合:```jsx
// Before:
import { act, Simulate, renderIntoDocument } from 'react-dom/test-utils';

// After  split the imports:
import { act } from 'react';
import { fireEvent, render } from '@testing-library/react'; // replaces Simulate + renderIntoDocument
```---

## 2. シミュレート → fireEvent```jsx
// Before  Simulate REMOVED in React 19:
import { Simulate } from 'react-dom/test-utils';
Simulate.click(element);
Simulate.change(input, { target: { value: 'hello' } });
Simulate.submit(form);
Simulate.keyDown(element, { key: 'Enter', keyCode: 13 });

// After:
import { fireEvent } from '@testing-library/react';
fireEvent.click(element);
fireEvent.change(input, { target: { value: 'hello' } });
fireEvent.submit(form);
fireEvent.keyDown(element, { key: 'Enter', keyCode: 13 });
```---

## 3.react-dom/test-utils の完全な API マップ

|古い (react-dom/test-utils) |新しい場所 |
|---|---|
| `act` | `import { act } from 'react'` |
| `Simulate` | `fireEvent` `@testing-library/react` から |
| `renderIntoDocument` | `render` `@testing-library/react` から |
| `findRenderedDOMComponentWithTag` | RTL からの `getByRole`、`getByTestId` |
| `findRenderedDOMComponentWithClass` | `getByRole` または `container.querySelector` |
| `scryRenderedDOMComponentsWithTag` | `getAllByRole` RTL から |
| `isElement`、`isCompositeComponent` | RTL では不要な削除 |
| `isDOMComponent` |削除 |

---

## 4. StrictMode の呼び出し数の修正

React 19 StrictMode は、開発中に `useEffect` を二重呼び出ししなくなりました。エフェクト呼び出しをカウントするスパイ アサーションを更新する必要があります。

**戦略は常に測定し、決して推測しないでください:**```bash
# Run the failing test, read the actual count from the error:
npm test -- --watchAll=false --testPathPattern="[filename]" --forceExit 2>&1 | grep -E "Expected|Received"
```

```jsx
// Before (React 18 StrictMode  effects ran twice):
expect(mockFn).toHaveBeenCalledTimes(2);  // 1 call × 2 (strict double-invoke)

// After (React 19 StrictMode  effects run once):
expect(mockFn).toHaveBeenCalledTimes(1);
``````jsx
// レンダリング フェーズの呼び出し (コンポーネント本体) は React 19 StrictMode で引き続き二重呼び出しされます。
Expect(renderSpy).toHaveBeenCalledTimes(2);  // レンダリング本体呼び出しでは 2 のまま