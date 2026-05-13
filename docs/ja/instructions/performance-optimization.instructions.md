---
applyTo: '**'
description: 'Core Web Vitals (LCP、INP、CLS) に基づく包括的な web performance 標準。50 を超える anti-pattern、検出 regex、modern web framework 向け framework 別の修正方法、最新 API のガイダンスを含みます。'
---

# パフォーマンス標準

web application 開発向けの包括的な performance ルールです。すべての anti-pattern には、severity 分類、検出方法、影響を受ける Core Web Vitals 指標、修正用 code 例を含めています。

**Severity level:**

- **CRITICAL** — Core Web Vital を直接「poor」閾値より悪化させます。merge 前に必ず修正してください。
- **IMPORTANT** — user experience に測定可能な影響があります。同じ sprint 内で修正してください。
- **SUGGESTION** — 最適化の余地があります。将来の iteration で計画してください。

---

## Core Web Vitals クイック リファレンス

### LCP (Largest Contentful Paint)

**Good: < 2.5s | Needs Improvement: 2.5-4s | Poor: > 4s**

表示領域内で最大の content element が描画を完了する時点を測定します。4 つの連続した phase があります。

| Phase | Target | 測定内容 |
|-------|--------|---------|
| TTFB | 予算の約 40% | server 応答時間 |
| Resource Load Delay | < 10% | TTFB と LCP resource の fetch 開始の間の時間 |
| Resource Load Duration | 約 40% | LCP resource の download 時間 |
| Element Render Delay | < 10% | download と paint の間の時間 |

### INP (Interaction to Next Paint)

**Good: < 200ms | Needs Improvement: 200-500ms | Poor: > 500ms**

すべての user interaction の latency を測定し、その中で最悪のものを報告します。3 つの phase があります。

| Phase | 最適化 |
|-------|--------|
| Input Delay | long task を分割し、browser に制御を返す |
| Processing Time | handler を 50ms 未満に保つ |
| Presentation Delay | DOM size を最小化し、forced layout を避ける |

> **診断ツール:** INP の問題を調べるには Long Animation Frames (LoAF) API (Chrome 123+) を使ってください。LoAF は従来の Long Tasks API よりも優れた attribution を提供し、script source や rendering time も含みます。

### CLS (Cumulative Layout Shift)

**Good: < 0.1 | Needs Improvement: 0.1-0.25 | Poor: > 0.25**

layout shift の原因には、dimension を持たない image、動的に挿入される content、web font FOUT、遅れて読み込まれる ad があります。user interaction 後 500ms 以内の shift は除外されます。

---

## Loading と LCP の Anti-Pattern (L1-L10)

### L1: Critical Extraction なしの Render-Blocking CSS

- **Severity**: CRITICAL
- **Detection**: `<link.*rel="stylesheet"` in `<head>` loading large CSS
- **CWV**: LCP

```html
<!-- BAD -->
<link rel="stylesheet" href="/styles/main.css" />

<!-- GOOD — inline critical CSS (extracted at build time), preload the rest -->
<style>/* critical above-fold CSS, inlined by a tool like Critters/Beasties */</style>
<link rel="preload" href="/styles/main.css" as="style" />
<link rel="stylesheet" href="/styles/main.css" />
```

build 時の critical CSS extraction (例: Critters、Beasties、Next.js `experimental.optimizeCss`) と通常の `<link rel="stylesheet">` を優先してください。古い `media="print" onload="this.media='all'"` の技巧は避けてください。厳格な CSP (`'unsafe-inline'` なし / `script-src-attr 'unsafe-inline'` なし) では inline event handler がブロックされ、stylesheet が有効化されず style regression を起こす可能性があります。non-critical CSS を本当に遅延させる必要があるなら、inline handler ではなく、`media` を切り替える **external** script で読み込んでください。

### L2: Render-Blocking な同期 Script

- **Severity**: CRITICAL
- **Detection**: `<script.*src=` without `async|defer|type="module"`
- **CWV**: LCP

```html
<!-- BAD -->
<script src="/vendor/analytics.js"></script>

<!-- GOOD -->
<script src="/vendor/analytics.js" defer></script>
```

### L3: Critical Origin への Preconnect がない

- **Severity**: IMPORTANT
- **Detection**: third-party API / CDN URL without `<link rel="preconnect">`
- **CWV**: LCP

```html
<link rel="preconnect" href="https://api.example.com" />
<link rel="dns-prefetch" href="https://analytics.example.com" />
```

### L4: LCP Resource の Preload がない

- **Severity**: CRITICAL
- **Detection**: LCP image / font not preloaded
- **CWV**: LCP

```html
<link rel="preload" as="image" href="/hero.webp" fetchpriority="high" />
```

### L5: Main Content に対する Client-Side Data Fetching

- **Severity**: CRITICAL
- **Detection**: `useEffect.*fetch|useEffect.*axios|ngOnInit.*subscribe`
- **CWV**: LCP

```tsx
// BAD — content appears after JS execution + API call
'use client';
function Page() {
  const [data, setData] = useState(null);
  useEffect(() => { fetch('/api/data').then(r => r.json()).then(setData); }, []);
  return <div>{data?.title}</div>;
}

// GOOD — Server Component fetches data before HTML is sent
async function Page() {
  const data = await fetch('https://api.example.com/data').then(r => r.json());
  return <div>{data.title}</div>;
}
```

### L6: 過剰な Redirect Chain

- **Severity**: IMPORTANT
- **Detection**: 複数の連続 redirect (HTTP 301/302 chain)
- **CWV**: LCP

redirect は 1 回ごとに 200-300ms を追加します。最大でも 1 redirect にしてください。

### L7: LCP Element に fetchpriority がない

- **Severity**: IMPORTANT
- **Detection**: above-fold の hero image に `fetchpriority="high"` または `priority` prop がない
- **CWV**: LCP

```tsx
// Next.js
<Image src="/hero.webp" alt="Hero" width={1200} height={600} priority />

// Angular
<img ngSrc="/hero.webp" alt="Hero" width="1200" height="600" priority>

// Plain HTML
<img src="/hero.webp" alt="Hero" width="1200" height="600" fetchpriority="high" />
```

### L8: Async / Defer なしの Head 内 Third-Party Script

- **Severity**: IMPORTANT
- **Detection**: `<script.*src="https://` without `async|defer`
- **CWV**: LCP

必須でない script は defer してください。chat widget には facade pattern を使います。

### L9: 大きすぎる初期 HTML (>14KB)

- **Severity**: SUGGESTION
- **Detection**: server-rendered HTML larger than 14KB
- **CWV**: LCP

inline CSS / JS を減らし、whitespace を除去し、Suspense boundary を使う streaming SSR を使ってください。

### L10: Compression がない

- **Severity**: IMPORTANT
- **Detection**: server not returning `content-encoding: br` or `gzip`
- **CWV**: LCP

CDN / server level で Brotli (gzip より 15-25% 良い) を有効にしてください。

---

## Rendering と Hydration の Anti-Pattern (R1-R8)

### R1: Component Tree 全体が "use client"

- **Severity**: CRITICAL
- **Detection**: `"use client"` at top-level layout or page component
- **CWV**: LCP + INP

`"use client"` は、interactivity が必要な leaf component まで押し下げてください。

### R2: Async Data 向け Suspense Boundary がない

- **Severity**: IMPORTANT
- **Detection**: data fetching を行う Server Component に `<Suspense>` がない
- **CWV**: LCP

```tsx
// GOOD — stream shell immediately, fill in data progressively
async function Page() {
  const user = await getUser();
  return (
    <div>
      <Header user={user} />
      <Suspense fallback={<PostsSkeleton />}>
        <Posts />
      </Suspense>
    </div>
  );
}
```

### R3: 動的な Client Content による Hydration Mismatch

- **Severity**: IMPORTANT
- **Detection**: `Date.now()|Math.random()|window\.innerWidth` in SSR component
- **CWV**: CLS

client 専用の value には `useEffect` を使うか、既知の差分には `suppressHydrationWarning` を使ってください。

### R4: 遅い Data Source に対する Streaming 不足

- **Severity**: IMPORTANT
- **Detection**: すべての data を待ってから HTML を送る page
- **CWV**: LCP (TTFB)

Suspense boundary を使う streaming SSR を使ってください。shell を先に stream し、遅い data は段階的に埋めます。

### R5: Re-render を引き起こす不安定な Reference

- **Severity**: IMPORTANT
- **Detection**: `style=\{\{|onClick=\{\(\) =>` inline in JSX
- **CWV**: INP

React 19+ で React Compiler が有効なら auto-memoize されます。Compiler がない場合は `useMemo` / `useCallback` で抽出・memoize してください。Angular では OnPush、Vue では `computed()` を使います。

### R6: Long List の Virtualization 不足

- **Severity**: IMPORTANT
- **Detection**: `>100` item を `.map(` で描画しているのに virtual scroll がない
- **CWV**: INP

TanStack Virtual、react-window、Angular CDK Virtual Scroll、vue-virtual-scroller を使ってください。

### R7: 直後に非表示になる Content の SSR

- **Severity**: SUGGESTION
- **Detection**: `display: none` component の server rendering
- **CWV**: LCP (TTFB)

modal、drawer、dropdown は client-side rendering を使ってください。Angular なら `@defer`、React なら `React.lazy` を使います。

### R8: List Item に `key` Prop がない

- **Severity**: IMPORTANT
- **Detection**: `.map(` without `key=` prop
- **CWV**: INP

```tsx
// GOOD — stable unique key
{items.map(item => <Row key={item.id} data={item} />)}
```

list が並び替わる可能性があるなら、array index を key に使ってはいけません。

---

## JavaScript Runtime と INP の Anti-Pattern (J1-J8)

### J1: Event Handler 内の長い同期 Task

- **Severity**: CRITICAL
- **Detection**: heavy computation (>50ms) を含む event handler
- **CWV**: INP

```typescript
// GOOD — yield to browser
async function handleClick() {
  setLoading(true);
  await (globalThis.scheduler?.yield?.() ?? new Promise(r => setTimeout(r, 0)));
  const result = expensiveComputation(data);
  setResult(result);
}
```

最善策は heavy work を Web Worker に移すことです。

> **Note:** `scheduler.yield()` は Chrome 129+、Firefox 129+ で利用できますが、2026 年 4 月時点で Safari は未対応です。fallback として `await (globalThis.scheduler?.yield?.() ?? new Promise(r => setTimeout(r, 0)))` を使ってください。

### J2: Layout Thrashing

- **Severity**: CRITICAL
- **Detection**: loop 内の `offsetHeight|offsetWidth|getBoundingClientRect|clientHeight`
- **CWV**: INP

```typescript
// GOOD — batch reads then batch writes
const heights = elements.map(el => el.offsetHeight);
elements.forEach((el, i) => { el.style.height = `${heights[i] + 10}px`; });
```

### J3: Cleanup なしの setInterval / setTimeout

- **Severity**: IMPORTANT
- **Detection**: cleanup のない `setInterval|setTimeout`
- **Impact**: Memory

```tsx
useEffect(() => {
  const id = setInterval(() => fetchData(), 5000);
  return () => clearInterval(id);
}, []);
```

### J4: removeEventListener なしの addEventListener

- **Severity**: IMPORTANT
- **Detection**: cleanup のない `addEventListener`
- **Impact**: Memory

```tsx
useEffect(() => {
  const controller = new AbortController();
  window.addEventListener('resize', handleResize, { signal: controller.signal });
  return () => controller.abort();
}, []);
```

### J5: Detached DOM Node Reference

- **Severity**: SUGGESTION
- **Detection**: 削除済み DOM element への reference を保持する variable
- **Impact**: Memory

element が削除されたら reference を `null` に設定してください。

### J6: 同期 XHR

- **Severity**: CRITICAL
- **Detection**: synchronous flag を使う `XMLHttpRequest`
- **CWV**: INP

常に非同期の `fetch()` を使ってください。

### J7: Main Thread 上の Heavy Computation

- **Severity**: IMPORTANT
- **Detection**: component code 内の CPU-intensive operation
- **CWV**: INP

Web Worker に移すか、`scheduler.yield()` で chunk に分割してください。

### J8: Effect Cleanup がない

- **Severity**: IMPORTANT
- **Detection**: return cleanup のない `useEffect`、unsubscribe のない `subscribe`
- **Impact**: Memory

React では `useEffect` から cleanup を返します。Angular では `takeUntilDestroyed()`、Vue では `onUnmounted` を使います。

---

## CSS Performance の Anti-Pattern (C1-C7)

### C1: Layout を引き起こす Property を使う Animation

- **Severity**: CRITICAL
- **Detection**: `animation:|transition:` with `top|left|width|height|margin|padding`
- **CWV**: INP

```css
/* BAD — main thread, <60fps */
.card { transition: width 0.3s, height 0.3s; }

/* GOOD — GPU compositor, 60fps */
.card { transition: transform 0.3s, opacity 0.3s; }
.card:hover { transform: scale(1.05); }
```

### C2: Off-Screen Section に content-visibility がない

- **Severity**: SUGGESTION
- **Detection**: `content-visibility: auto` のない長い page
- **CWV**: INP

```css
.below-fold-section {
  content-visibility: auto;
  contain-intrinsic-size: auto 500px;
}
```

### C3: 恒久的に適用された will-change

- **Severity**: SUGGESTION
- **Detection**: base CSS にある `will-change:` (`:hover|:focus` ではない)
- **Impact**: Memory

interaction 時だけ適用するか、browser の自動最適化に任せてください。

### C4: 大量の未使用 CSS

- **Severity**: IMPORTANT
- **Detection**: rule の 50% 超が未使用の CSS
- **CWV**: LCP

PurgeCSS、Tailwind purge、critters を使ってください。route ごとに CSS を code-split します。

### C5: Hot Path における Universal Selector

- **Severity**: SUGGESTION
- **Detection**: CSS 内の `\* \{`
- **CWV**: INP

```css
/* GOOD — zero-specificity reset */
:where(*, *::before, *::after) { box-sizing: border-box; }
```

### C6: CSS Containment 不足

- **Severity**: SUGGESTION
- **Detection**: `contain` property のない複雑 component
- **CWV**: INP

```css
.sidebar { contain: layout style paint; }
```

### C7: View Transitions API なしの Route Transition

- **Severity**: SUGGESTION
- **Detection**: View Transitions API を使っていない SPA route change
- **CWV**: CLS (perceived)

```javascript
// Use View Transitions for smooth route changes (with feature check)
if (document.startViewTransition) {
  document.startViewTransition(() => {
    // update DOM / navigate
  });
} else {
  // fallback: update DOM directly
}
```

same-document transition は主要 browser すべてでサポートされています。cross-document は Chrome / Edge 126+、Safari 18.5+ でサポートされます。呼び出し前に必ず feature check を行ってください。未対応 browser では guard なしだと throw されます。

---

## Image、Media、Font の Anti-Pattern (I1-I8)

### I1: Dimension を持たない Image

- **Severity**: CRITICAL
- **Detection**: `width=` と `height=` がない `<img`
- **CWV**: CLS

image には常に `width` と `height` を設定するか、CSS の `aspect-ratio` を使ってください。

### I2: Above-Fold Image の Lazy Loading

- **Severity**: CRITICAL
- **Detection**: hero / banner image に `loading="lazy"`
- **CWV**: LCP

```html
<!-- GOOD — eager load with high priority -->
<img src="/hero.webp" alt="Hero" fetchpriority="high" />
```

### I3: Legacy Format のみ (JPEG / PNG)

- **Severity**: IMPORTANT
- **Detection**: WebP / AVIF alternative のない image
- **CWV**: LCP

```html
<picture>
  <source srcset="/hero.avif" type="image/avif" />
  <source srcset="/hero.webp" type="image/webp" />
  <img src="/hero.jpg" alt="Hero" width="1200" height="600" />
</picture>
```

### I4: Responsive srcset / sizes がない

- **Severity**: IMPORTANT
- **Detection**: `srcset` のない `<img`
- **CWV**: LCP

```html
<img src="/hero-800.jpg" alt="Hero"
     srcset="/hero-400.jpg 400w, /hero-800.jpg 800w, /hero-1200.jpg 1200w"
     sizes="(max-width: 600px) 400px, (max-width: 1024px) 800px, 1200px" />
```

### I5: font-display のない Font

- **Severity**: IMPORTANT
- **Detection**: `@font-face` without `font-display`
- **CWV**: CLS

```css
@font-face {
  font-family: 'CustomFont';
  src: url('/fonts/custom.woff2') format('woff2');
  font-display: swap; /* or "optional" for best CLS */
}
```

### I6: Critical Font の Preload がない

- **Severity**: IMPORTANT
- **Detection**: `<link rel="preload">` のない custom font
- **CWV**: LCP + CLS

```html
<link rel="preload" href="/fonts/main.woff2" as="font" type="font/woff2" crossorigin />
```

### I7: Subset で十分なのに Full Font を読み込む

- **Severity**: SUGGESTION
- **Detection**: 50KB を超える WOFF2 font file
- **CWV**: LCP

`unicode-range`、glyphhanger、または `next/font` (Google Fonts を自動 subset) を使ってください。

### I8: 最適化されていない SVG

- **Severity**: SUGGESTION
- **Detection**: editor metadata を含む SVG
- **CWV**: LCP (minor)

```bash
npx svgo input.svg -o output.svg
```

---

## Bundle と Tree Shaking の Anti-Pattern (B1-B6)

### B1: Module 全体を読み込む Barrel File Import

- **Severity**: IMPORTANT
- **Detection**: `from '\.\/(?:.*\/index|components)'`
- **CWV**: INP

```typescript
// BAD
import { Button } from './components';

// GOOD — direct import
import { Button } from './components/Button';
```

### B2: Tree Shaking を妨げる CommonJS require()

- **Severity**: IMPORTANT
- **Detection**: frontend code 内の `require(`
- **CWV**: INP

ESM の `import/export` を使ってください。`require` は `import` に置き換えます。

### B3: 小さな Utility に対して大きな Dependency を使う

- **Severity**: IMPORTANT
- **Detection**: `from "moment"|from "lodash"` (full import)
- **CWV**: INP

```typescript
// GOOD — tree-shakeable alternatives
import { format } from 'date-fns';
import { pick } from 'lodash-es';

// BEST — native JS
const formatted = new Intl.DateTimeFormat('en').format(date);
```

### B4: Route Splitting のための Dynamic Import 不足

- **Severity**: CRITICAL
- **Detection**: route component がすべて静的 import
- **CWV**: INP

```tsx
// Next.js: automatic with file-based routing
// React:
const Page = React.lazy(() => import('./pages/Page'));
// Angular:
{ path: 'settings', loadComponent: () => import('./pages/settings.component') }
// Vue:
const Page = defineAsyncComponent(() => import('./pages/Page.vue'));
```

### B5: package.json に sideEffects がない

- **Severity**: SUGGESTION
- **Detection**: `"sideEffects"` field のない library package.json
- **CWV**: INP

```json
{ "sideEffects": false }
```

### B6: 重複 Dependency

- **Severity**: SUGGESTION
- **Detection**: 同じ library の複数 version
- **CWV**: INP

```bash
npm dedupe
```

---

## Framework-Specific: Next.js (NX1-NX6)

### NX1: next/image を使っていない

- **Severity**: IMPORTANT
- **Detection**: `.tsx` 内で `<Image>` ではなく `<img `
- **CWV**: LCP + CLS

```tsx
import Image from 'next/image';
<Image src="/hero.jpg" alt="Hero" width={1200} height={600} priority />
```

### NX2: Partial Prerendering 向け Cache Components を使っていない

- **Severity**: IMPORTANT
- **Detection**: Next.js 16+ project で `"use cache"` directive のない page
- **CWV**: LCP

```typescript
// BAD — entire page is dynamic
export default async function Page() {
  const data = await fetchData(); // blocks full page render
  return <div>{data.title}</div>;
}

// GOOD — enable Partial Prerendering with "use cache"
// next.config.ts: { cacheComponents: true }
"use cache";
export default async function Page() {
  const data = await fetchData(); // static shell renders instantly, dynamic holes stream
  return <div>{data.title}</div>;
}
```

`next.config.ts` で `cacheComponents: true` を有効にします。`"use cache"` は file、component、function level で使えます。static shell は即座に読み込まれ、dynamic content は Suspense boundary 経由で stream されます。

### NX3: Server Rendering 可能な Component に不要な "use client"

- **Severity**: IMPORTANT
- **Detection**: hook や browser API を使わない component に `"use client"`
- **CWV**: INP

静的 content だけを render する component からは `"use client"` を削除してください。

### NX4: Server-Side ではなく useEffect で Data Fetching

- **Severity**: CRITICAL
- **Detection**: Next.js App Router page における `useEffect` + `fetch`
- **CWV**: LCP

data は Server Component の本体 (async function body) で直接 fetch してください。

### NX5: next/font がない

- **Severity**: IMPORTANT
- **Detection**: CSS / HTML 内の `fonts.googleapis|fonts.gstatic`
- **CWV**: CLS + LCP

```tsx
import { Inter } from 'next/font/google';
const inter = Inter({ subsets: ['latin'] });
```

### NX6: Cacheable な Server Function に "use cache" がない

- **Severity**: IMPORTANT
- **Detection**: `cacheComponents: true` な Next.js 16+ で `"use cache"` のない async server function
- **CWV**: LCP

```typescript
// BAD — data fetched on every request
async function getProducts() {
  return await db.products.findMany();
}

// GOOD — cached with revalidation
"use cache";
import { cacheLife } from 'next/cache';
async function getProducts() {
  cacheLife('hours');
  return await db.products.findMany();
}
```

`"use cache"` は古い `unstable_cache` と `fetch` cache option の代替です。細かい制御には `cacheLife()` と `cacheTag()` を使ってください。

---

## Framework-Specific: Angular (NG1-NG6)

### NG1: Presentational Component で既定の Change Detection を使う

- **Severity**: IMPORTANT
- **Detection**: `ChangeDetectionStrategy.OnPush` のない component (Angular <19)、または signal を使っていない component (Angular 19+)
- **CWV**: INP

```typescript
// Angular <19: Use OnPush
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush,
  ...
})

// Angular 19+: Prefer zoneless with signals
// app.config.ts: provideZonelessChangeDetection()
@Component({ ... })
export class ProductCard {
  product = input.required<Product>(); // signal input
  price = computed(() => this.product().price * 1.19); // derived signal
}
```

Angular 19+ では signal を使う zoneless change detection を優先してください。signal-based reactivity を使うなら OnPush は不要です。Angular 20+ では zoneless support が stable です。

### NG2: NgOptimizedImage を使っていない

- **Severity**: IMPORTANT
- **Detection**: `.component.html` 内で `ngSrc` のない `<img`
- **CWV**: LCP + CLS

```html
<img ngSrc="/hero.jpg" alt="Hero" width="1200" height="600" priority />
```

### NG3: Below-Fold Content に @defer がない

- **Severity**: SUGGESTION
- **Detection**: eagerly loaded される heavy な below-fold component (Angular 17+)
- **CWV**: INP

```html
@defer (on viewport) {
  <app-heavy-chart [data]="chartData" />
} @placeholder {
  <div class="chart-skeleton"></div>
}
```

### NG4: Reactive State に Signal を使っていない

- **Severity**: SUGGESTION
- **Detection**: Angular 19+ で signal を使わない class property
- **CWV**: INP

reactive state には `signal()`、派生値には `computed()` を使ってください。signal API (`signal()`, `computed()`, `effect()`) は Angular 20 以降 stable です。

### NG5: Incremental Hydration なしの Full Hydration

- **Severity**: IMPORTANT
- **Detection**: Angular 19+ で `withIncrementalHydration()` のない SSR app
- **CWV**: LCP, INP

```typescript
// BAD — full hydration blocks interactivity
provideClientHydration()

// GOOD — incremental hydration with triggers
provideClientHydration(withIncrementalHydration())
```

`@defer` trigger (`on viewport`, `on interaction`) を使って、必要時に component を hydrate してください。critical でない component hydration を遅らせることで TTI を削減できます。

### NG6: Angular 20+ Project でまだ zone.js を使っている

- **Severity**: SUGGESTION
- **Detection**: Angular 20+ で polyfills array に `zone.js` があり、`provideZonelessChangeDetection()` がない
- **CWV**: INP

```typescript
// app.config.ts
export const appConfig = {
  providers: [
    provideZonelessChangeDetection(), // removes ~15-30KB from bundle
    // ...
  ]
};
```

signal を使う zoneless change detection は bundle size を減らし、runtime performance を改善します。Angular 20 以降 stable です。

---

## Framework-Specific: React (RX1-RX4)

### RX1: React Compiler の採用不足

- **Severity**: SUGGESTION
- **Detection**: React 19+ project 内の手動 `useMemo|useCallback`
- **CWV**: INP

auto-memoization のために React Compiler (v19+) を有効にし、手動 wrapper を削除してください。

### RX2: 高コストな Update に useTransition がない

- **Severity**: IMPORTANT
- **Detection**: `useTransition` なしで高コストな re-render を起こす state update
- **CWV**: INP

```tsx
const [isPending, startTransition] = useTransition();
function handleFilter(value) {
  startTransition(() => setFilter(value));
}
```

### RX3: 高コストな Rendering に useDeferredValue がない

- **Severity**: IMPORTANT
- **Detection**: rapidly-changing input 由来の高コスト rendering
- **CWV**: INP

```tsx
const deferredQuery = useDeferredValue(query);
const results = expensiveFilter(items, deferredQuery);
```

### RX4: Route Splitting に React.lazy がない

- **Severity**: IMPORTANT
- **Detection**: route component が静的 import
- **CWV**: INP

```tsx
const Settings = React.lazy(() => import('./pages/Settings'));
```

---

## Framework-Specific: Vue (VU1-VU4)

### VU1: 大きな Data Structure に対する reactive()

- **Severity**: IMPORTANT
- **Detection**: 大きな array や深い object に対する `reactive(`
- **CWV**: INP

大きな data には `shallowRef()` または `shallowReactive()` を使ってください。

### VU2: 高コストな List Render に v-memo がない

- **Severity**: SUGGESTION
- **Detection**: `v-memo` のない large list
- **CWV**: INP

```vue
<div v-for="item in items" :key="item.id" v-memo="[item.id, item.updatedAt]">
  <ExpensiveItem :data="item" />
</div>
```

### VU3: defineAsyncComponent がない

- **Severity**: IMPORTANT
- **Detection**: heavy component の静的 import
- **CWV**: INP

```typescript
const HeavyChart = defineAsyncComponent(() => import('./HeavyChart.vue'));
```

### VU4: Performance-Critical Component で Vapor Mode を使っていない

- **Severity**: SUGGESTION
- **Detection**: Vue 3.6+ で virtual DOM を使う performance-critical component
- **CWV**: INP

Vue 3.6+ の Vapor Mode は template を仮想 DOM を介さない直接 DOM 操作に compile します。performance-critical な subtree で使ってください。standard component と混在できます。

---

## Resource Hints クイック リファレンス

| Hint | 目的 | 使用タイミング |
|------|------|---------------|
| `preconnect` | DNS + TCP + TLS を先行実行 | critical な third-party origin (API、CDN、font) |
| `preload` | 即時取得、高 priority | LCP image、critical font |
| `prefetch` | 将来 navigation 用の低 priority 読み込み | 次 page の asset |
| `dns-prefetch` | DNS resolution のみ | critical ではない third-party origin |
| `modulepreload` | ES module の preload + parse | critical JS module |
| `<script type="speculationrules">` | 次 navigation の prefetch / prerender | 遷移可能性が高い次 page (Chrome 121+、progressive enhancement) |

---

## Image Optimization クイック リファレンス

| 項目 | 推奨 |
|------|------|
| Format | WebP (25-34% 小さい)、AVIF (50% 小さい) |
| LCP image | `fetchpriority="high"` または framework の `priority` prop |
| Below-fold | `loading="lazy"` |
| Dimensions | 常に `width` + `height` を設定 |
| Responsive | `srcset` + `sizes` または framework Image component |
| Compression | photo は quality 75-85 |

---

## Font Loading クイック リファレンス

| Strategy | 最適な用途 | CLS への影響 |
|----------|-----------|-------------|
| `font-display: swap` | body text | わずかな FOUT、最小限の CLS |
| `font-display: optional` | すべての font (最良の CLS) | FOUT なし、CLS なし |
| `next/font` | Next.js project | CLS 0 |
| Variable fonts | 複数 weight | すべての weight を 1 file に集約 |

ルール: preload する critical font は 1-2 個だけにし、WOFF2 を使い、必要な文字だけに subset し、可能であれば self-host してください。

---

## Performance Checklist (CWV)

### LCP (< 2.5s)
- [ ] LCP image に `fetchpriority="high"` または `priority` prop がある
- [ ] HTML source に含まれない場合、LCP image が preload されている
- [ ] above-fold image に `loading="lazy"` がない
- [ ] critical CSS が inline 化または抽出されている
- [ ] render-blocking script がない (`defer` または `async` を使う)
- [ ] critical third-party origin に preconnect している
- [ ] main content が server-rendered されている (client-side fetch ではない)
- [ ] image が modern format (WebP / AVIF) と responsive `srcset` を使っている
- [ ] compression が有効 (Brotli 推奨)
- [ ] font が `font-display: swap` または `optional` で preload されている

### INP (< 200ms)
- [ ] event handler が 50ms 未満で完了する
- [ ] long task が小さな chunk に分割されている
- [ ] route-based code splitting が実装されている
- [ ] heavy computation が Web Worker に移されている
- [ ] 100 件超の item を持つ list が virtualized されている
- [ ] barrel file import がない (direct component import を使う)
- [ ] ESM import を使っている (CommonJS `require` ではない)
- [ ] `"use client"` が interactivity が必要な component にのみ付いている
- [ ] layout-triggering な CSS property を animation していない
- [ ] effect cleanup が実装されている (listener / timer leak がない)

### CLS (< 0.1)
- [ ] すべての image に `width` と `height` attribute がある
- [ ] font が `font-display: swap` または `optional` を使っている
- [ ] 既存 content の上に動的 content が挿入されない
- [ ] ad / embed に予約済み space がある
- [ ] hydration mismatch がない
- [ ] `content-visibility: auto` に `contain-intrinsic-size` がある
