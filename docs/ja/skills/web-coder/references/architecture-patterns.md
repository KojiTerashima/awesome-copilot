# アーキテクチャとパターンのリファレンス

Web アプリケーションのアーキテクチャ、デザイン パターン、アーキテクチャの概念。

## アプリケーション アーキテクチャ

### シングルページアプリケーション (SPA)

単一の HTML ページを読み込み、コンテンツを動的に更新する Web アプリ。

**特徴**:
- クライアント側ルーティング
- JavaScript の多用
- 初期ロード後の高速ナビゲーション
- 複雑な状態管理

**長所**:
- スムーズなユーザーエクスペリエンス
- サーバー負荷の軽減
- モバイルアプリのような操作感

**短所**:
- 初期ダウンロードが大きくなる
- SEO の課題 (SSR で軽減)
- 複雑な状態管理

**例**: React、Vue、Angular アプリ```javascript
// React Router example
import { BrowserRouter, Routes, Route } from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/products/:id" element={<Product />} />
      </Routes>
    </BrowserRouter>
  );
}
```### マルチページ アプリケーション (MPA)

複数の HTML ページを含む従来の Web アプリ。

**特徴**:
- サーバーは各ページをレンダリングします
- ナビゲーションでページ全体をリロードします
- よりシンプルなアーキテクチャ

**長所**:
- すぐに使える優れた SEO
- 構築がより簡単
- コンテンツの多いサイトに適しています

**短所**:
- ナビゲーションが遅くなる
- より多くのサーバーリクエスト

### プログレッシブ ウェブ アプリ (PWA)

ネイティブ アプリ機能を備えた Web アプリ。

**特徴**:
- インストール可能
- オフラインサポート (サービスワーカー)
- プッシュ通知
- アプリのような体験```javascript
// Service Worker registration
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/sw.js')
    .then(reg => console.log('SW registered', reg))
    .catch(err => console.error('SW error', err));
}
```**manifest.json**:```json
{
  "name": "My PWA",
  "short_name": "PWA",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#000000",
  "icons": [
    {
      "src": "/icon-192.png",
      "sizes": "192x192",
      "type": "image/png"
    }
  ]
}
```### サーバーサイド レンダリング (SSR)

サーバー上でページをレンダリングし、HTML をクライアントに送信します。

**長所**:
- SEOの向上
- 最初のコンテンツフルペイントの高速化
- JavaScriptなしで動作します

**短所**:
- サーバー負荷が高くなる
- より複雑なセットアップ

**フレームワーク**: Next.js、Nuxt.js、SvelteKit```javascript
// Next.js SSR
export async function getServerSideProps() {
  const data = await fetchData();
  return { props: { data } };
}

function Page({ data }) {
  return <div>{data.title}</div>;
}
```### 静的サイト生成 (SSG)

ビルド時にページを事前レンダリングします。

**長所**:
- 非常に速い
- サーバーコストが低い
- 優れたSEO

**最適な用途**: ブログ、ドキュメント、マーケティング サイト

**ツール**: Next.js、Gatsby、Hugo、Jekyll、イレブンティ```javascript
// Next.js SSG
export async function getStaticProps() {
  const data = await fetchData();
  return { props: { data } };
}

export async function getStaticPaths() {
  const paths = await fetchPaths();
  return { paths, fallback: false };
}
```### インクリメンタル静的再生 (ISR)

ビルド後に静的コンテンツを更新します。```javascript
// Next.js ISR
export async function getStaticProps() {
  const data = await fetchData();
  return {
    props: { data },
    revalidate: 60 // Revalidate every 60 seconds
  };
}
```### ジャムスタック

JavaScript、API、マークアップ アーキテクチャ。

**原則**:
- 事前にレンダリングされた静的ファイル
- 動的機能用の API
- Gitベースのワークフロー
- CDN展開

**利点**:
- 高速なパフォーマンス
- 高いセキュリティ
- スケーラビリティ
- 開発者の経験

## レンダリングパターン

### クライアントサイド レンダリング (CSR)

JavaScript はブラウザーでコンテンツをレンダリングします。```html
<div id="root"></div>
<script>
  // React renders app here
  ReactDOM.render(<App />, document.getElementById('root'));
</script>
```### 水分補給

サーバーでレンダリングされた HTML に JavaScript を添付します。```javascript
// React hydration
ReactDOM.hydrate(<App />, document.getElementById('root'));
```### 部分的な水分補給

インタラクティブなコンポーネントのみをハイドレートします。

**ツール**: Astro、Qwik

### 諸島の建築

静的 HTML 内の独立した対話型コンポーネント。

**コンセプト**: 最小限のJavaScriptを出荷し、インタラクティブ性の「島」のみをハイドレートします。

**フレームワーク**: Astro、イレブンティ ウィズ アイランド

## デザインパターン

### MVC (モデル-ビュー-コントローラー)

データ、プレゼンテーション、ロジックを分離します。

- **モデル**: データとビジネス ロジック
- **表示**: UI プレゼンテーション
- **コントローラー**: 入力を処理し、モデル/ビューを更新します

### MVVM (モデル-ビュー-ビューモデル)

データバインディングを備えた MVC に似ています。

- **モデル**: データ
- **表示**: UI
- **ViewModel**: ロジックと状態を表示します。

**使用場所**: Vue.js、Angular、Knockout

### コンポーネントベースのアーキテクチャ

再利用可能なコンポーネントから UI を構築します。```javascript
// React component
function Button({ onClick, children }) {
  return (
    <button onClick={onClick} className="btn">
      {children}
    </button>
  );
}

// Usage
<Button onClick={handleClick}>Click me</Button>
```### マイクロフロントエンド

フロントエンドをより小さな独立したアプリに分割します。

**アプローチ**:
- ビルド時の統合
- ランタイム統合 (iframe、Web コンポーネント)
- エッジ側には以下が含まれます

## 状態管理

### 地方州

コンポーネントレベルの状態。```javascript
// React useState
function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```### 世界の状態

アプリケーション全体の状態。

**解決策**:
- **Redux**: 予測可能な状態コンテナー
- **MobX**: 監視可能な状態
- **Zustand**: 最小限の状態管理
- **反動**: アトミック状態管理```javascript
// Redux example
import { createSlice, configureStore } from '@reduxjs/toolkit';

const counterSlice = createSlice({
  name: 'counter',
  initialState: { value: 0 },
  reducers: {
    increment: state => { state.value += 1; }
  }
});

const store = configureStore({
  reducer: { counter: counterSlice.reducer }
});
```### コンテキスト API

プロペラの穴あけなしで状態を共有します。```javascript
// React Context
const ThemeContext = React.createContext('light');

function App() {
  return (
    <ThemeContext.Provider value="dark">
      <Toolbar />
    </ThemeContext.Provider>
  );
}

function Toolbar() {
  const theme = useContext(ThemeContext);
  return <div className={theme}>...</div>;
}
```## API アーキテクチャ パターン

### REST (表現状態転送)

リソースベースの API 設計。```javascript
// RESTful API
GET    /api/users      // List users
GET    /api/users/1    // Get user
POST   /api/users      // Create user
PUT    /api/users/1    // Update user
DELETE /api/users/1    // Delete user
```### グラフQL

API のクエリ言語。```graphql
# Query
query {
  user(id: "1") {
    name
    email
    posts {
      title
    }
  }
}

# Mutation
mutation {
  createUser(name: "John", email: "john@example.com") {
    id
    name
  }
}
```

```javascript
// Apollo Client
import { useQuery, gql } from '@apollo/client';

const GET_USER = gql`
  query GetUser($id: ID!) {
    user(id: $id) {
      name
      email
    }
  }
`;

function User({ id }) {
  const { loading, error, data } = useQuery(GET_USER, {
    variables: { id }
  });
  
  if (loading) return <p>Loading...</p>;
  return <p>{data.user.name}</p>;
}
```### tRPC

エンドツーエンドのタイプセーフ API。```typescript
// Server
const appRouter = router({
  getUser: publicProcedure
    .input(z.string())
    .query(async ({ input }) => {
      return await db.user.findUnique({ where: { id: input } });
    })
});

// Client (fully typed!)
const user = await trpc.getUser.query('1');
```## マイクロサービス アーキテクチャ

アプリケーションを小さな独立したサービスに分割します。

**特徴**:
- 独立した展開
- サービス固有のデータベース
- API通信
- 分散型ガバナンス

**利点**:
- スケーラビリティ
- テクノロジーの柔軟性
- 障害の切り分け

**課題**:
- 複雑さ
- ネットワーク遅延
- データの一貫性

## モノリシック アーキテクチャ

単一の統合アプリケーション。

**長所**:
- よりシンプルな開発
- デバッグが容易になりました
- 単一の展開

**短所**:
- スケーリングの課題
- テクノロジーのロックイン
- 密結合

## サーバーレス アーキテクチャ

サーバーを管理せずにコードを実行します。

**プラットフォーム**: AWS Lambda、Vercel Functions、Netlify Functions、Cloudflare Workers```javascript
// Vercel serverless function
export default function handler(req, res) {
  res.status(200).json({ message: 'Hello from serverless!' });
}
```**利点**:
- 自動スケーリング
- 使用ごとに支払い
- サーバー管理なし

**使用例**:
- API
- バックグラウンドジョブ
- Webhook
- 画像処理

## アーキテクチャのベスト プラクティス

### 懸念事項の分離

さまざまな側面を分けておいてください。
- プレゼンテーション層
- ビジネスロジック層
- データアクセス層

### DRY (同じことを繰り返さないでください)

コードの重複を避けてください。

### 確固たる原則

- **単一の責任
- **O**ペン/クローズ
- **L**イスコフの交代
- **I**インターフェースの分離
- **D**依存関係の反転

### 継承よりも構成

クラス階層よりもオブジェクトを構成することを好みます。```javascript
// Composition
function withLogging(Component) {
  return function LoggedComponent(props) {
    console.log('Rendering', Component.name);
    return <Component {...props} />;
  };
}

const LoggedButton = withLogging(Button);
```## モジュールシステム

### ES モジュール (ESM)

最新の JavaScript モジュール。```javascript
// export
export const name = 'John';
export function greet() {}
export default App;

// import
import App from './App.js';
import { name, greet } from './utils.js';
import * as utils from './utils.js';
```### CommonJS

Node.js モジュール システム。```javascript
// export
module.exports = { name: 'John' };
exports.greet = function() {};

// import
const { name } = require('./utils');
```## ビルドの最適化

### コード分割

コードをより小さなチャンクに分割します。```javascript
// React lazy loading
const OtherComponent = React.lazy(() => import('./OtherComponent'));

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <OtherComponent />
    </Suspense>
  );
}
```### 木の揺れ

未使用のコードを削除します。```javascript
// Only imports 'map', not entire lodash
import { map } from 'lodash-es';
```### バンドルの分割

- **ベンダー バンドル**: サードパーティの依存関係
- **アプリバンドル**: アプリケーションコード
- **ルート バンドル**: ルートごとのコード

## 用語集の用語

**対象となる重要な用語**:
- 抽象化
- API
- アプリケーション
- 建築
- 非同期
- バインディング
- ブロック（CSS、JS）
- コールスタック
- クラス
- クライアント側
- 制御フロー
- デルタ
- デザインパターン
- イベント
- フェッチ
- 第一級関数
- 機能
- ガベージコレクション
- グリッド
- 吊り上げ
- 水分補給
- べき等
- インスタンス
- 遅延ロード
- メインスレッド
-MVC

- ポリフィル
- 段階的な強化
- プログレッシブウェブアプリ
- プロパティ
- プロトタイプ
- プロトタイプベースのプログラミング
- 休憩
- リフロー
- 往復時間 (RTT)
- スパ
- セマンティクス
- サーバー
- 総合モニタリング
- スレッド
- タイプ

## 追加のリソース

- [パターン.dev](https://www.patterns.dev/)
- [React Patterns](https://reactpatterns.com/)
- [JAMstack](https://jamstack.org/)
- [マイクロフロントエンド](https://micro-frontends.org/)