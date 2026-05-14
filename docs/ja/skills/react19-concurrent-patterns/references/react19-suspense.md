---
title: React 19 Suspense for Data Fetching Pattern Reference
---
# React 19 Suspense のデータ取得パターンのリファレンス

**データ取得**のための React 19 の新しい Suspense 統合は、`useEffect + state` を使用せずに、データが利用可能になるまでコンポーネントを一時停止 (レンダリングの一時停止) できるようにするプレビュー機能です。

**重要:** これは**プレビュー**であり、特定の設定が必要であり、本番環境ではまだ安定していませんが、React 19 の移行計画のパターンを知っておく必要があります。

---

## React 19 で何が変わったのでしょうか?

React 18 Suspense は **コード分割** (遅延コンポーネント) のみをサポートしていました。 React 19 では、特定の条件が満たされた場合にこれを **データフェッチ** に拡張します。

- **ライブラリの使用法** データ取得ライブラリは Suspense を実装する必要があります (React Query 5+、SWR、Remix ローダーなど)
- **または独自のプロミス追跡** React が一時停止を追跡できる方法で Promise をラップします
- **「サスペンスの後にフックがない」** ことがなくなり、`use()` を使用してコンポーネント内で直接サスペンスを使用できるようになりました。

---

## React 18 サスペンス (コード分割のみ)```jsx
// React 18  Suspense for lazy imports only:
const LazyComponent = React.lazy(() => import('./Component'));

function App() {
  return (
    <Suspense fallback={<Spinner />}>
      <LazyComponent />
    </Suspense>
  );
}
```React 18 でデータを一時停止しようとすると、ハックまたはライブラリが必要になりました。```jsx
// React 18 hack  not recommended:
const dataPromise = fetchData();
const resource = {
  read: () => {
    throw dataPromise; // Throw to suspend
  }
};

function Component() {
  const data = resource.read(); // Throws promise → Suspense catches it
  return <div>{data}</div>;
}
```---

## データ取得のための React 19 サスペンド (プレビュー)

React 19 は、`use()` フックを介した Promise によるサスペンスの **ファーストクラス サポート** を提供します。```jsx
// React 19  Suspense for data fetching:
function UserProfile({ userId }) {
  const user = use(fetchUser(userId)); // Suspends if promise pending
  return <div>{user.name}</div>;
}

function App() {
  return (
    <Suspense fallback={<Spinner />}>
      <UserProfile userId={123} />
    </Suspense>
  );
}
```**React 18 との主な違い:**

- `use()` は Promise をアンラップし、コンポーネントは自動的に一時停止します
- `useEffect + state` トリックは必要ありません
- よりクリーンなコード、定型句の削減

---

## パターン 1: シンプルな約束のサスペンス```jsx
// Raw promise (not recommended in production):
function DataComponent() {
  const data = use(fetch('/api/data').then(r => r.json()));
  return <pre>{JSON.stringify(data, null, 2)}</pre>;
}

function App() {
  return (
    <Suspense fallback={<Spinner />}>
      <DataComponent />
    </Suspense>
  );
}
```**問題:** Promise はレンダリングごとに再作成されます。解決策: `useMemo` で囲みます。

---

## パターン 2: メモ化された約束 (より良い)```jsx
function DataComponent({ id }) {
  // Only create promise once per id:
  const dataPromise = useMemo(() => 
    fetch(`/api/data/${id}`).then(r => r.json()),
    [id]
  );
  
  const data = use(dataPromise);
  return <pre>{JSON.stringify(data, null, 2)}</pre>;
}

function App() {
  const [id, setId] = useState(1);
  
  return (
    <Suspense fallback={<Spinner />}>
      <DataComponent id={id} />
      <button onClick={() => setId(id + 1)}>Next</button>
    </Suspense>
  );
}
```---

## パターン 3: ライブラリの統合 (React Query)

最新のデータ ライブラリはサスペンスを直接サポートしています。 React Query 5+ の例:```jsx
// React Query 5+ with Suspense:
import { useSuspenseQuery } from '@tanstack/react-query';

function UserProfile({ userId }) {
  // useSuspenseQuery throws promise if suspended
  const { data: user } = useSuspenseQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
  });
  
  return <div>{user.name}</div>;
}

function App() {
  return (
    <Suspense fallback={<Spinner />}>
      <UserProfile userId={123} />
    </Suspense>
  );
}
```**利点:** ライブラリはキャッシュ、再試行、キャッシュの無効化を処理します。

---

## パターン 4: エラー境界の統合

Suspense と Error Boundary を組み合わせて、読み込みとエラーの両方を処理します。```jsx
function UserProfile({ userId }) {
  const user = use(fetchUser(userId)); // Suspends while loading
  return <div>{user.name}</div>;
}

function App() {
  return (
    <ErrorBoundary fallback={<ErrorScreen />}>
      <Suspense fallback={<Spinner />}>
        <UserProfile userId={123} />
      </Suspense>
    </ErrorBoundary>
  );
}

class ErrorBoundary extends React.Component {
  state = { error: null };
  
  static getDerivedStateFromError(error) {
    return { error };
  }
  
  render() {
    if (this.state.error) return this.props.fallback;
    return this.props.children;
  }
}
```---

## 入れ子になったサスペンス境界

複数のサスペンス境界を使用して、別のデータを待機している間に部分的な UI を表示します。```jsx
function App({ userId }) {
  return (
    <div>
      <Suspense fallback={<UserSpinner />}>
        <UserProfile userId={userId} />
      </Suspense>
      
      <Suspense fallback={<PostsSpinner />}>
        <UserPosts userId={userId} />
      </Suspense>
    </div>
  );
}

function UserProfile({ userId }) {
  const user = use(fetchUser(userId));
  return <h1>{user.name}</h1>;
}

function UserPosts({ userId }) {
  const posts = use(fetchUserPosts(userId));
  return <ul>{posts.map(p => <li key={p.id}>{p.title}</li>)}</ul>;
}
```今:

- ユーザープロフィールには読み込み中にスピナーが表示されます
- 投稿ではスピナーが独立して表示されます
- どちらも完了時にレンダリングできます

---

## シーケンシャル サスペンスとパラレル サスペンス

### シーケンシャル (最初を待ってから 2 番目をフェッチする)```jsx
function App({ userId }) {
  const user = use(fetchUser(userId)); // Must complete first
  
  return (
    <Suspense fallback={<PostsSpinner />}>
      <UserPosts userId={user.id} /> {/* Depends on user */}
    </Suspense>
  );
}

function UserPosts({ userId }) {
  const posts = use(fetchUserPosts(userId));
  return <ul>{posts.map(p => <li>{p.title}</li>)}</ul>;
}
```### 並列 (両方を一度にフェッチ)```jsx
function App({ userId }) {
  return (
    <div>
      <Suspense fallback={<UserSpinner />}>
        <UserProfile userId={userId} />
      </Suspense>
      
      <Suspense fallback={<PostsSpinner />}>
        <UserPosts userId={userId} /> {/* Fetches in parallel */}
      </Suspense>
    </div>
  );
}
```---

## React 18 → React 19 への移行戦略

### フェーズ 1 変更は必要ありません

サスペンスはまだオプションであり、データ取得に関しては実験的です。既存の `useEffect + state` パターンはすべて引き続き機能します。

### フェーズ 2 安定するまで待ちます

実稼働環境でサスペンス データの取得を採用する前に、次のことを行ってください。

- React 19 が出荷されるまで待ちます (プレビューではありません)
- データ ライブラリがサスペンスをサポートしていることを確認します
- アプリが React 19 コアで安定した後に移行を計画する

### フェーズ 3 サスペンスへのリファクタリング (オプション、プレビュー後)

安定したら、候補者のプロフィールを作成します。```bash
grep -rn "useEffect.*fetch\|useEffect.*axios\|useEffect.*graphql" src/ --include="*.js" --include="*.jsx"
```

```jsx
// Before (React 18):
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, [userId]);
  
  if (!user) return <Spinner />;
  return <div>{user.name}</div>;
}

// After (React 19 with Suspense):
function UserProfile({ userId }) {
  const user = use(fetchUser(userId));
  return <div>{user.name}</div>;
}

// Must be wrapped in Suspense:
<Suspense fallback={<Spinner />}>
  <UserProfile userId={123} />
</Suspense>
```---

## 重要な警告

1. **まだプレビュー** データのサスペンスは実験的なものであるため、動作が変更される可能性があります
2. **パフォーマンス** の約束は、メモ化せずにレンダリングごとに再作成されます。 `useMemo`を使用してください
3. **キャッシュ** `use()` はキャッシュしません。本番アプリには React Query などを使用してください
4. **SSR** サスペンス SSR のサポートは制限されています。 Next.js のバージョン要件を確認する