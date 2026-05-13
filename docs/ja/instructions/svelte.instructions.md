---
description: 'コンポーネントベースのユーザーインターフェイスとフルスタックアプリケーションのための、Svelte 5 と SvelteKit の開発標準およびベストプラクティス'
applyTo: '**/*.svelte, **/*.ts, **/*.js, **/*.css, **/*.scss, **/*.json'
---

# Svelte 5 と SvelteKit の開発指示

最新の runes ベース reactivity、TypeScript、パフォーマンス最適化を用いて、高品質な Svelte 5 / SvelteKit アプリケーションを構築するための指示です。

## プロジェクトコンテキスト
- runes system ($state、$derived、$effect、$props、$bindable) を備えた Svelte 5.x
- file-based routing を用いたフルスタックアプリケーション向け SvelteKit
- 型安全性と開発者体験向上のための TypeScript
- CSS custom properties を使った component-scoped styling
- progressive enhancement と performance-first のアプローチ
- 最適化を備えた最新 build tooling (Vite)

## 中核概念

### アーキテクチャー
- すべての reactivity には legacy store ではなく Svelte 5 の runes system を使う
- 拡張しやすいよう、component は feature または domain ごとに整理する
- 見た目中心の component とロジックが重い component を分離する
- 再利用可能なロジックは composable function に抽出する
- slot と snippet による適切な component composition を実装する
- 適切な load function を伴う SvelteKit の file-based routing を使う

### コンポーネント設計
- component には単一責任原則を適用する
- 既定では runes syntax を使う `<script lang="ts">` を使う
- component は小さく保ち、1 つの関心事に集中させる
- TypeScript annotation による適切な prop validation を実装する
- component 内で再利用する template logic には `{#snippet}` block を使う
- component composition と content projection には slots を使う
- 柔軟な親子 composition のため `children` snippet を渡す
- component はテストしやすく再利用可能に設計する

## Reactivity と状態

### Svelte 5 Runes System
- reactive なローカル state 管理には `$state()` を使う
- 計算値や高コスト計算には `$derived()` を実装する
- 単純な式を超える複雑な計算には `$derived.by()` を使う
- `$effect()` は控えめに使う。state 同期には `$derived` や function binding を優先する
- DOM 更新前にコードを動かすには `$effect.pre()` を実装する
- effect 内で同じ state を読み書きして無限ループになるのを防ぐには `untrack()` を使う
- component props は `$props()` と TypeScript annotation 付き destructuring で定義する
- component 間の双方向 data binding には `$bindable()` を使う
- より良いパフォーマンスのため、legacy store から runes へ移行する
- 楽観的 UI パターンでは derived value を直接上書きする (Svelte 5.25+)

### 状態管理
- ローカル component state には `$state()` を使う
- 生の `setContext` / `getContext` より、`createContext()` helper を使った型安全な context を実装する
- reactive な state を component tree 全体に共有するには context API を使う
- SSR では global `$state` module を避ける。request 間の data leak を防ぐため context を使う
- 必要な場合はグローバルアプリケーション state に SvelteKit store を使う
- 複雑な data structure に対して state は正規化して保つ
- 計算値には `$effect()` より `$derived()` を優先する
- client-side data には適切な state persistence を実装する

### Effect のベストプラクティス
- state を同期するために `$effect()` を使うのは **避ける**。代わりに `$derived()` を使う
- analytics、logging、DOM manipulation のような side effect には `$effect()` を **使う**
- 適切に teardown するため、effect から cleanup function を **返す**
- DOM 更新前に動く必要があるコード (例: scroll position) には `$effect.pre()` を使う
- component lifecycle 外で手動制御する effect には `$effect.root()` を使う
- effect 内で依存関係を作らずに state を読むには `untrack()` を使う
- 注意: effect 内の async code は `await` 以降の依存関係を追跡しない

## SvelteKit パターン

### ルーティングと Layout
- 適切な SEO を伴う page component には `+page.svelte` を使う
- 共有 layout と navigation には `+layout.svelte` を実装する
- ルーティングは SvelteKit の file-based system で扱う

### データ読み込みと変更
- server-side data loading と API call には `+page.server.ts` を使う
- data mutation には `+page.server.ts` で form action を実装する
- API endpoint と server-side logic には `+server.ts` を使う
- server-side と universal data fetching には SvelteKit の load function を使う
- 適切な loading、error、success state を実装する
- server load function では promise を使って streaming data を処理する
- cache 管理には `invalidate()` と `invalidateAll()` を使う
- より良いユーザー体験のため、optimistic update を実装する
- offline scenario や network error を適切に処理する

### フォームと検証
- server-side form handling には SvelteKit の form action を使う
- `use:enhance` で progressive enhancement を実装する
- 制御された form input には `bind:value` を使う
- data は client-side と server-side の両方で検証する
- file upload や複雑な form scenario を処理する
- label と ARIA attribute による適切な accessibility を実装する

## UI とスタイリング

### スタイル
- `<style>` block による component-scoped style を使う
- theming と design system には CSS custom properties を実装する
- 条件付き style には `class:` directive を使う
- BEM または utility-first CSS convention に従う
- mobile-first アプローチで responsive design を実装する
- 本当に global style が必要な場合にのみ `:global()` を控えめに使う

### トランジションとアニメーション
- enter / exit animation (fade、slide、scale、fly) には `transition:` directive を使う
- enter / exit を分けるには `in:` と `out:` を使う
- リストの滑らかな並び替えには `flip` と `animate:` directive を実装する
- ブランドに合わせた motion design には custom transition を作る
- 直接の変更時だけ transition を起動するには `|local` modifier を使う
- リスト animation には keyed `{#each}` block と transition を組み合わせる

## TypeScript と Tooling

### TypeScript 統合
- 最大限の型安全性のため、`tsconfig.json` では strict mode を有効にする
- prop には TypeScript で annotation する: `let { name }: { name: string } = $props()`
- event handler、ref、SvelteKit が生成する型を適切に型付けする
- 再利用可能な component には generic type を使う
- SvelteKit が生成する `$types.ts` file を活用する
- 適切な型検査には `svelte-check` を実装する
- boilerplate を減らすため、可能な箇所では型推論を使う

### 開発ツール
- コード一貫性のため、ESLint と eslint-plugin-svelte、および Prettier を使う
- debugging と performance analysis には Svelte DevTools を使う
- 依存関係は常に最新に保ち、セキュリティ脆弱性を監査する
- 複雑な component やロジックは JSDoc で文書化する
- Svelte の命名規則 (component は PascalCase、function は camelCase) に従う

## 本番運用への備え

### パフォーマンス最適化
- 効率的なリスト描画には keyed `{#each}` block を使う
- dynamic import と `<svelte:component>` で lazy loading を実装する
- 不要な再計算を避けるため、高コスト計算には `$derived()` を使う
- 複数文を要する複雑な derived value には `$derived.by()` を使う
- derived state に `$effect()` を使うのは避ける。`$derived()` より効率が悪い
- SvelteKit の自動 code splitting と preloading を活用する
- tree shaking と適切な import で bundle size を最適化する
- performance bottleneck の特定には Svelte DevTools で profile する
- 条件付きで reactive listener を作る abstraction には `$effect.tracking()` を使う

### エラーハンドリング
- route-level error boundary には `+error.svelte` page を実装する
- load function と form action では try/catch block を使う
- 意味のある error message と fallback UI を提供する
- debugging と monitoring のため、適切に error を記録する
- form の validation error は適切な user feedback とともに処理する
- 適切な response には SvelteKit の `error()` と `redirect()` helper を使う
- loading state のため、保留中 promise は `$effect.pending()` で追跡する

### テスト
- component の unit test には Vitest と Testing Library を使う
- 実装詳細ではなく component の振る舞いをテストする
- user workflow の end-to-end test には Playwright を使う
- SvelteKit の load function と store は適切に mock する
- form action と API endpoint は十分にテストする
- axe-core による accessibility test を実装する

### セキュリティ
- XSS 攻撃を防ぐため、user input をサニタイズする
- `@html` directive は慎重に使い、HTML content を検証する
- SvelteKit で適切な CSRF protection を実装する
- load function と form action 内の data を検証しサニタイズする
- すべての外部 API call と本番デプロイには HTTPS を使う
- 機密 data は適切な session management とともに安全に保管する

### アクセシビリティ
- semantic HTML 要素と適切な heading hierarchy を使う
- すべての interactive 要素に keyboard navigation を実装する
- 適切な ARIA label と description を提供する
- color contrast が WCAG guideline を満たすことを保証する
- screen reader と accessibility tool でテストする
- 動的 content に focus management を実装する

### デプロイ
- 異なる deployment stage 間の設定には environment variable を使う
- SvelteKit の meta tag と structured data で適切な SEO を実装する
- hosting platform に応じて適切な SvelteKit adapter でデプロイする

## 実装プロセス
1. TypeScript と必要な adapter で SvelteKit project を初期化する
2. 適切な folder 構成で project structure を整える
3. TypeScript interface と component prop を定義する
4. Svelte 5 runes を使って中核 component を実装する
5. SvelteKit で routing、layout、navigation を追加する
6. data loading と form handling を実装する
7. custom properties と responsive design による styling system を追加する
8. error handling と loading state を実装する
9. 包括的な test coverage を追加する
10. performance と bundle size を最適化する
11. accessibility compliance を確保する
12. 適切な SvelteKit adapter でデプロイする

## よくあるパターン
- 柔軟な UI composition のための slot を使った renderless component
- 横断的関心事と DOM manipulation のための custom action (`use:` directive)
- component 内で再利用する template logic のための `{#snippet}` block
- component tree の state 共有のための `createContext()` を使った型安全な context
- form や interactive feature に対する `use:enhance` を使った progressive enhancement
- 最適なパフォーマンスのための server-side rendering と client-side hydration
- 双方向 binding のための function binding (`bind:value={() => value, setValue}`)
- state 同期には `$effect()` を避け、代わりに `$derived()` または callback を使う
