---
description: "モダンな caching、tooling、server/client boundary を備えた Next.js (App Router) アプリ構築のベストプラクティス（Next.js 16.1.1 に準拠）。"
applyTo: "**/*.tsx, **/*.ts, **/*.jsx, **/*.js, **/*.css"
---

# LLM 向け Next.js ベストプラクティス (2026)

_最終更新: 2026年1月（Next.js 16.1.1 に準拠）_

このドキュメントは、Next.js アプリケーションを構築・構造化・保守するための、最新かつ信頼できるベストプラクティスをまとめたものです。コード品質、保守性、拡張性を確保するために、LLM と開発者の両方を対象としています。

---

## 1. プロジェクト構造と整理

- 新しいプロジェクトでは **`app/` ディレクトリ**（App Router）を使ってください。旧来の `pages/` ディレクトリより優先します。
- **トップレベルのフォルダー:**
  - `app/` — routing、layout、page、route handler
  - `public/` — 静的 asset（画像、font など）
  - `lib/` — 共通 utility、API client、business logic
  - `components/` — 再利用可能な UI component
  - `contexts/` — React context provider
  - `styles/` — global / modular stylesheet
  - `hooks/` — custom React hook
  - `types/` — TypeScript の型定義
- **Colocation:** file（component、style、test）は使用箇所の近くに置いてよいですが、深すぎるネストは避けてください。
- **Route Groups:** URL path に影響させず route をグループ化するには丸括弧（例: `(admin)`）を使ってください。
- **Private Folders:** routing から除外し実装詳細であることを示すには、`_` プレフィックス（例: `_internal`）を使ってください。
- **Feature Folders:** 大規模アプリでは、`app/dashboard/`、`app/auth/` のように feature ごとにグループ化してください。
- **`src/` の使用**（任意）: source code を config file と分離したい場合は、すべて `src/` 配下に置いてください。

## 2. Next.js 16+ App Router のベストプラクティス

### 2.1. Server Component と Client Component の統合（App Router）

**Server Component 内で `{ ssr: false }` を付けた `next/dynamic` を使ってはいけません。** これはサポートされておらず、build/runtime error の原因になります。

**正しいアプローチ:**

- Server Component の中で Client Component（hook、browser API、client-only library を使う component など）を使いたい場合は、次のようにします。
  1. client-only の logic / UI を、先頭に `'use client'` を持つ専用の Client Component に移してください。
  2. その Client Component を Server Component に直接 import して使ってください（`next/dynamic` は不要です）。
  3. navbar と profile dropdown のように複数の client-only 要素を組み合わせる必要がある場合は、それらを含む単一の Client Component を作ってください。

**例:**

```tsx
// Server Component
import DashboardNavbar from "@/components/DashboardNavbar";

export default async function DashboardPage() {
  // ...server logic...
  return (
    <>
      <DashboardNavbar /> {/* This is a Client Component */}
      {/* ...rest of server-rendered page... */}
    </>
  );
}
```

**理由:**

- Server Component は client-only feature や SSR 無効化つき dynamic import を使えません。
- Client Component は Server Component の中で描画できますが、その逆はできません。

**要点:**
client-only UI は必ず Client Component に移し、Server Component から直接 import してください。Server Component 内で `{ ssr: false }` を使った `next/dynamic` は決して使わないでください。

### 2.2. Next.js 16+ の async request API（App Router）

- **App Router の Server Component と Route Handler では、request に紐づくデータは async であると考えてください。** Next.js 16 では `cookies()`, `headers()`, `draftMode()` のような API は async です。
- **route props に注意してください:** Server Component では `params` / `searchParams` が Promise の場合があります。通常の object として扱うのではなく、`await` することを優先してください。
- **意図せず dynamic rendering にしないでください:** request data（cookies / headers / searchParams）へアクセスすると route は dynamic になります。必要性を意識して読み取り、適切なら `Suspense` boundary の後ろに dynamic な部分を分離してください。

---

## 3. Component のベストプラクティス

- **Component の種類:**
  - **Server Components**（デフォルト）: data fetching、重い logic、非インタラクティブ UI に使う
  - **Client Components:** 先頭に `'use client'` を追加。interactivity、state、browser API に使う
- **Component を作るタイミング:**
  - 同じ UI pattern が 2 回以上再利用されるとき
  - page の一部が複雑、または自己完結しているとき
  - 可読性や testability が向上するとき
- **命名規約:**
  - component の file 名と export 名には `PascalCase` を使う（例: `UserCard.tsx`）
  - hook には `camelCase` を使う（例: `useUser.ts`）
  - 静的 asset には `snake_case` または `kebab-case` を使う（例: `logo_dark.svg`）
  - context provider は `XyzProvider` と命名する（例: `ThemeProvider`）
- **ファイル命名:**
  - component 名と file 名を一致させる
  - 単一 export の file では default export を使う
  - 複数の関連 component では `index.ts` の barrel file を使う
- **配置場所:**
  - 共有 component は `components/` に置く
  - route 固有の component は対応する route folder の中に置く
- **Props:**
  - props には TypeScript interface を使う
  - 明示的な prop type と default value を優先する
- **Testing:**
  - test は component の近くに配置する（例: `UserCard.test.tsx`）

## 4. 命名規約（全般）

- **Folder:** `kebab-case`（例: `user-profile/`）
- **File:** component は `PascalCase`、utility / hook は `camelCase`、静的 asset は `kebab-case`
- **Variable / Function:** `camelCase`
- **Type / Interface:** `PascalCase`
- **Constant:** `UPPER_SNAKE_CASE`

## 5. API Route（Route Handler）

- 超低レイテンシや地理的分散が必要な場合を除き、**Edge Function より API Route を優先** してください。
- **場所:** API route は `app/api/` に配置してください（例: `app/api/users/route.ts`）。
- **HTTP Method:** HTTP verb 名の async function（`GET`, `POST` など）を export してください。
- **Request / Response:** Web 標準の `Request` と `Response` API を使ってください。高度な機能が必要な場合は `NextRequest` / `NextResponse` を使ってください。
- **Dynamic Segment:** 動的 API route には `[param]` を使ってください（例: `app/api/users/[id]/route.ts`）。
- **Validation:** 入力は必ず検証とサニタイズを行ってください。`zod` や `yup` のような library を使ってください。
- **Error Handling:** 適切な HTTP status code と error message を返してください。
- **Authentication:** middleware や server-side session check で、機密性の高い route を保護してください。

### Route Handler の使い方に関する注意（パフォーマンス）

- Server Component から共通 logic を再利用する目的だけで、自分自身の Route Handler（例: `fetch('/api/...')`）を呼び出してはいけません。余計な server hop を避けるため、共有 logic は `lib/` などの module に切り出し、直接呼び出してください。

## 6. 全般的なベストプラクティス

- **TypeScript:** すべてのコードに TypeScript を使ってください。`tsconfig.json` では `strict` mode を有効にしてください。
- **ESLint & Prettier:** code style と linting を強制してください。公式 Next.js ESLint config を使ってください。Next.js 16 では `next lint` ではなく ESLint CLI で ESLint を実行することを優先してください。
- **Environment Variables:** secret は `.env.local` に保存し、version control に commit しないでください。
  - Next.js 16 では `serverRuntimeConfig` / `publicRuntimeConfig` は削除されています。代わりに environment variable を使ってください。
  - `NEXT_PUBLIC_` 変数は **build 時に埋め込まれます**（build 後に値を変更してもデプロイ済み build には反映されません）。
  - dynamic context で真に runtime 評価が必要なら、Next.js のガイドに従ってください（例: `process.env` を読む前に `connection()` を呼ぶ）。
- **Testing:** Jest、React Testing Library、Playwright を使ってください。重要な logic と component には必ず test を書いてください。
- **Accessibility:** semantic HTML と ARIA attribute を使ってください。screen reader でもテストしてください。
- **Performance:**
  - 画像と font の最適化には組み込み機能を使う
  - legacy caching pattern より **Cache Components**（`cacheComponents` + `use cache`）を優先する
  - async data には Suspense と loading state を使う
  - client bundle を大きくしすぎない。大半の logic は Server Component に置く
- **Security:**
  - すべての user input をサニタイズする
  - 本番では HTTPS を使う
  - 安全な HTTP header を設定する
  - Server Action と Route Handler の認可は server-side で行う。client input を信用しない
- **Documentation:**
  - 明確な README と code comment を書く
  - public API と component を文書化する

## 7. Caching と Revalidation（Next.js 16 Cache Components）

- App Router における memoization / caching には **Cache Components を優先** してください。
  - `next.config.*` で `cacheComponents: true` を有効にする
  - component / function を cache 対象にするには **`use cache` directive** を使う
- cache tag と有効期間は意図を持って使ってください。
  - `cacheTag(...)` を使って cache に tag を関連づける
  - `cacheLife(...)` を使って cache の lifetime（preset または configured profile）を制御する
- **revalidation の指針:**
  - 多くの場合は `revalidateTag(tag, 'max')`（stale-while-revalidate）を優先する
  - 単一引数の `revalidateTag(tag)` は legacy / deprecated
  - “read-your-writes” のように即時整合性が必要なら、**Server Action** 内で `updateTag(...)` を使う
- 新しいコードでは **`unstable_cache` を避ける** こと。legacy とみなし、Cache Components へ移行してください。

## 8. Tooling の更新（Next.js 16）

- **Turbopack は default の dev bundler** です。設定は `next.config.*` のトップレベル `turbopack` field で行ってください（削除済みの `experimental.turbo` は使わないでください）。
- **typed routes は安定機能** で、`typedRoutes` により利用できます（TypeScript 必須）。

## 9. 不要なサンプルファイルを避ける

ユーザーが live example、Storybook story、明示的な documentation component を特別に求めていない限り、ModalExample.tsx のような example / demo file を main codebase に作成しないでください。デフォルトでは、repository はクリーンで production-focused に保ってください。

## 10. 常に最新のドキュメントとガイドを使う

- Next.js 関連の依頼では、まず最新の Next.js documentation、guide、example を探してください。
- それらが利用可能なら、次の tool を使ってドキュメントを取得・検索してください。
  - `resolve_library_id` で docs 内の package / library 名を解決する
  - `get_library_docs` で最新のドキュメントを取得する
