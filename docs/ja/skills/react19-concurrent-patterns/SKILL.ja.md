---
name: react19-concurrent-patterns
description: 'Preserve React 18 concurrent patterns and adopt React 19 APIs (useTransition, useDeferredValue, Suspense, use(), useOptimistic, Actions) during migration.'
---
# React 19 の同時パターン

React 19 では、移行作業を補完する新しい API が導入されました。このスキルは次の 2 つの懸念事項に対応します。

1. 移行中に壊れてはいけない既存の React 18 同時パターンを **保持**
2. 移行が安定した後に導入する価値のある新しい React 19 API を **採用**

## パート 1 保持: 移行後も存続する必要がある React 18 の同時パターン

これらのパターンは React 18 コードベースに存在するため、誤って削除したり破損したりしないでください。

### createRoot は R18 Orchestra によってすでに移行されています

R18 オケがすでに実行されている場合は、`ReactDOM.render` → `createRoot` が完了します。正しいことを確認してください:```jsx
// CORRECT React 19 root (same as React 18):
import { createRoot } from 'react-dom/client';
const root = createRoot(document.getElementById('root'));
root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```### useTransition 移行は必要ありません

React 18 の `useTransition` は React 19 でも同様に機能します。移行中は次のパターンに触れないでください。```jsx
// React 18 useTransition  unchanged in React 19:
const [isPending, startTransition] = useTransition();

function handleClick() {
  startTransition(() => {
    setFilteredResults(computeExpensiveFilter(input));
  });
}
```### useDeferredValue 移行は必要ありません```jsx
// React 18 useDeferredValue  unchanged in React 19:
const deferredQuery = useDeferredValue(query);
```### コード分割のため一時停止、移行は不要```jsx
// React 18 Suspense with lazy  unchanged in React 19:
const LazyComponent = React.lazy(() => import('./LazyComponent'));

function App() {
  return (
    <Suspense fallback={<Spinner />}>
      <LazyComponent />
    </Suspense>
  );
}
```---

## パート 2 React 19 の新しい API

これらは、移行後のクリーンアップ スプリントで採用する価値があります。最初に移行が安定するまでは、これらを導入しないでください。

それぞれの新しい API の完全なパターンについては、以下をお読みください。
- **`references/react19-use.md`** プロミスとコンテキストの `use()` フック
- **`references/react19-actions.md`** アクション、useActionState、useFormStatus、useOptimistic
- **`references/react19-suspense.md`** データ取得のサスペンド（新パターン）

## 移行の安全ルール

React 19 への移行中は、これらの同時モード パターンは **完全にそのままにしておく必要があります**。```bash
# Verify nothing touched these during migration:
grep -rn "useTransition\|useDeferredValue\|Suspense\|startTransition" \
  src/ --include="*.js" --include="*.jsx" | grep -v "\.test\."
```移行者がこれらのファイルのいずれかを操作した場合は、移行によって変更された変更は React API サーフェス (forwardRef、defaultProps など) のみであり、同時モード ロジックは変更されていないことを確認してください。