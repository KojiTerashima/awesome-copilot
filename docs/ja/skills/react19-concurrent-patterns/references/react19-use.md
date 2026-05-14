---
title: React 19 use() Hook Pattern Reference
---
# React 19 use() フックパターンリファレンス

`use()` フックは、React コンポーネント内の Promise とコンテキストをアンラップするための React 19 の答えです。これにより、コンポーネント本体で直接、よりクリーンな非同期パターンが可能になり、以前は個別の遅延コンポーネントや複雑な状態管理を必要としたアーキテクチャの複雑さが回避されます。

## use() とは何ですか?

`use()` は次のようなフックです。

- **Promise または context オブジェクトを受け入れます**
- **解決された値またはコンテキスト値を返します**
- **処理** プロミスに対する自動的な一時停止
- **条件付きで呼び出すことができます** コンポーネント内で (Promise のトップレベルではありません)
- **エラーをスローします**。サスペンス + エラー境界でキャッチできる

## Promise で use()

### React 18 パターン```jsx
// React 18 approach 1  lazy load a component module:
const UserComponent = React.lazy(() => import('./User'));

function App() {
  return (
    <Suspense fallback={<Spinner />}>
      <UserComponent />
    </Suspense>
  );
}

// React 18 approach 2  fetch data with state + useEffect:
function App({ userId }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, [userId]);
  
  if (!user) return <Spinner />;
  return <User user={user} />;
}
```### React 19 use() パターン```jsx
// React 19  use() directly in component:
function App({ userId }) {
  const user = use(fetchUser(userId)); // Suspends automatically
  return <User user={user} />;
}

// Usage:
function Root() {
  return (
    <Suspense fallback={<Spinner />}>
      <App userId={123} />
    </Suspense>
  );
}
```**主な違い:**

- `use()` は、Promise をコンポーネント本体で直接アンラップします
- サスペンス境界は引き続き必要ですが、アプリのルートに配置できます (コンポーネントごとではありません)。
- 単純な非同期データには状態や useEffect は必要ありません
- コンポーネント内での条件付きラッピングが可能

## Promise を使用した use() -- 条件付きフェッチ```jsx
// React 18  conditional with state
function SearchResults() {
  const [results, setResults] = useState(null);
  const [query, setQuery] = useState('');
  
  useEffect(() => {
    if (query) {
      search(query).then(setResults);
    } else {
      setResults(null);
    }
  }, [query]);
  
  if (!results) return null;
  return <Results items={results} />;
}

// React 19  use() with conditional
function SearchResults() {
  const [query, setQuery] = useState('');
  
  if (!query) return null;
  
  const results = use(search(query)); // Only fetches if query is truthy
  return <Results items={results} />;
}
```## コンテキストで use()

`use()` は、コンポーネントのルートになくてもコンテキストをアンラップできます。 Promise の使用ほど一般的ではありませんが、条件付きコンテキストの読み取りに役立ちます。```jsx
// React 18  always in component body, only works at top level
const theme = useContext(ThemeContext);

// React 19  can be conditional
function Button({ useSystemTheme }) {
  const theme = useSystemTheme ? use(ThemeContext) : defaultTheme;
  return <button style={theme}>Click</button>;
}
```---

## 移行戦略

### フェーズ 1 変更は必要ありません

React 19 `use()` はオプトインです。既存のサスペンス + コンポーネントの分割パターンはすべて引き続き機能します。```jsx
// Keep this as-is if it's working:
const Lazy = React.lazy(() => import('./Component'));
<Suspense fallback={<Spinner />}><Lazy /></Suspense>
```### フェーズ 2 移行後のクリーンアップ (オプション)

React 19 の移行が安定したら、`useEffect + state` 非同期パターンのコードベースをプロファイリングします。以下は `use()` リファクタリングの適切な候補です。

パターンを特定する:```bash
grep -rn "useEffect.*\(.*fetch\|async\|promise" src/ --include="*.js" --include="*.jsx"
```対象:

- シンプルなフェッチオンマウントパターン
- 複雑な依存関係配列は不要
- コンポーネントごとに単一のプロミス
- サスペンスはアプリ内の他の場所ですでに使用されています

リファクタリングの例:```jsx
// Before:
function Post({ postId }) {
  const [post, setPost] = useState(null);
  
  useEffect(() => {
    fetchPost(postId).then(setPost);
  }, [postId]);
  
  if (!post) return <Spinner />;
  return <PostContent post={post} />;
}

​// After:
function Post({ postId }) {
  const post = use(fetchPost(postId));
  return <PostContent post={post} />;
}

// And ensure Suspense at app level:
<Suspense fallback={<AppSpinner />}>
  <Post postId={123} />
</Suspense>
```---

## エラー処理

`use()` はエラーをスローし、サスペンス エラー境界がキャッチします。```jsx
function Root() {
  return (
    <ErrorBoundary fallback={<ErrorScreen />}>
      <Suspense fallback={<Spinner />}>
        <DataComponent />
      </Suspense>
    </ErrorBoundary>
  );
}

function DataComponent() {
  const data = use(fetchData()); // If fetch rejects, error boundary catches it
  return <Data data={data} />;
}
```---

## use() を使用しない場合

- **移行中は避けてください** まず React 19 を安定化してください
- **複雑な依存関係** 複数の Promise または複雑な順序付けロジックがある場合は、`useEffect` を使用してください。
- **再試行ロジック** `use()` は再試行を処理しません。 `useEffect` の状態がより明確になります
- **デバウンスされた更新** `use()` はプロパティが変更されるたびに再フェッチします。 `useEffect` をクリーンアップした方が良い