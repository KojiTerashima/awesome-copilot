# パフォーマンスと最適化のリファレンス

Web パフォーマンス メトリクス、最適化テクニック、および Core Web Vitals に関する包括的なリファレンス。

## コア ウェブ バイタル

ユーザーエクスペリエンスを測定するためのGoogleの指標。

### 最大のコンテンツフル ペイント (LCP)

最大のコンテンツ要素が表示されるときの読み込みパフォーマンスを測定します。

**目標**: < 2.5 秒

**最適化**:
- サーバーの応答時間を短縮します
- 画像の最適化
- レンダリングをブロックするリソースを削除する
- CDN を使用する
- 遅延読み込みの実装
- 重要なリソースをプリロードする```html
<link rel="preload" href="hero-image.jpg" as="image">
```### 最初の入力遅延 (FID) → 次のペイントへのインタラクション (INP)

FID (非推奨) は入力応答性を測定します。 INP は新しい指標です。

**INP ターゲット**: < 200ms

**最適化**:
- JavaScriptの実行時間を最小限に抑える
- 長いタスクを分割する
- Web ワーカーを使用する
- サードパーティのスクリプトを最適化します。
- `requestIdleCallback` を使用します

### 累積レイアウト シフト (CLS)

視覚的な安定性を測定します - 予期しないレイアウトの変化。

**目標**: < 0.1

**最適化**:
- 画像/ビデオのサイズを指定する
- 既存のコンテンツの上にコンテンツを挿入しないでください
- CSS アスペクト比を使用する
- 動的コンテンツ用のスペースを予約する```html
<img src="image.jpg" width="800" height="600" alt="Photo">

<style>
  .video-container {
    aspect-ratio: 16 / 9;
  }
</style>
```## その他のパフォーマンス指標

### 最初のコンテンツフル ペイント (FCP)
最初のコンテンツ要素がレンダリングされる時間。  
**目標**: < 1.8秒

### 最初のバイトまでの時間 (TTFB)
ブラウザが応答の最初のバイトを受信する時間。  
**ターゲット**: < 600ms

### インタラクティブまでの時間 (TTI)
ページが完全にインタラクティブになったとき。  
**目標**: < 3.8秒

### 速度指数
コンテンツが視覚的に表示される速度。  
**目標**: < 3.4秒

### 合計ブロッキング時間 (TBT)
すべての長いタスクのブロック時間の合計。  
**目標**: < 200ms

## 画像の最適化

### フォーマットの選択

|フォーマット |最適な用途 |長所 |短所 |
|------|----------|------|------|
| JPEG |写真 |小型で広くサポートされています |損失があり、透明性がない |
| PNG |グラフィックス、透明度 |ロスレス、透明性 |大きいサイズ |
|ウェブP |最新のブラウザ |小さいサイズ、透明 |古いブラウザのサポートが制限されている |
| AVIF |最新のフォーマット |最高の圧縮 |限定的なサポート |
| SVG |アイコン、ロゴ |スケーラブル、小型 |写真用ではありません |

### レスポンシブ画像```html
<!-- Picture element for art direction -->
<picture>
  <source media="(min-width: 1024px)" srcset="large.webp" type="image/webp">
  <source media="(min-width: 768px)" srcset="medium.webp" type="image/webp">
  <source media="(min-width: 1024px)" srcset="large.jpg">
  <source media="(min-width: 768px)" srcset="medium.jpg">
  <img src="small.jpg" alt="Responsive image">
</picture>

<!-- Srcset for resolution switching -->
<img
  src="image-800.jpg"
  srcset="image-400.jpg 400w,
          image-800.jpg 800w,
          image-1200.jpg 1200w"
  sizes="(max-width: 600px) 400px,
         (max-width: 1000px) 800px,
         1200px"
  alt="Image">

<!-- Lazy loading -->
<img src="image.jpg" loading="lazy" alt="Lazy loaded">
```### 画像圧縮

- ImageOptim、Squoosh、Sharp などのツールを使用する
- JPEG の品質は 80 ～ 85% を目標にします
- プログレッシブ JPEG を使用する
- メタデータの除去

## コードの最適化

### 縮小化

空白、コメントを削除し、名前を短縮します。```javascript
// Before
function calculateTotal(price, tax) {
  const total = price + (price * tax);
  return total;
}

// After minification
function t(p,x){return p+p*x}
```**ツール**: Terser (JS)、cssnano (CSS)、html-minifier

### コード分割

コードを小さなチャンクに分割し、オンデマンドでロードします。```javascript
// Dynamic import
button.addEventListener('click', async () => {
  const module = await import('./heavy-module.js');
  module.run();
});

// React lazy loading
const HeavyComponent = React.lazy(() => import('./HeavyComponent'));

// Webpack code splitting
import(/* webpackChunkName: "lodash" */ 'lodash').then(({ default: _ }) => {
  // Use lodash
});
```### 木の揺れ

バンドル中に未使用のコードを削除します。```javascript
// Only imports what's used
import { debounce } from 'lodash-es';

// ESM exports enable tree shaking
export { function1, function2 };
```### 圧縮

gzip または Brotli 圧縮を有効にします。```nginx
# nginx config
gzip on;
gzip_types text/plain text/css application/json application/javascript;
gzip_min_length 1000;

# brotli (better compression)
brotli on;
brotli_types text/plain text/css application/json application/javascript;
```## キャッシュ戦略

### キャッシュ制御ヘッダー```http
# Immutable assets (versioned URLs)
Cache-Control: public, max-age=31536000, immutable

# HTML (always revalidate)
Cache-Control: no-cache

# API responses (short cache)
Cache-Control: private, max-age=300

# No caching
Cache-Control: no-store
```### サービスワーカー

高度なキャッシュ制御:```javascript
// Cache-first strategy
self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request).then((response) => {
      return response || fetch(event.request);
    })
  );
});

// Network-first strategy
self.addEventListener('fetch', (event) => {
  event.respondWith(
    fetch(event.request).catch(() => {
      return caches.match(event.request);
    })
  );
});

// Stale-while-revalidate
self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.open('dynamic').then((cache) => {
      return cache.match(event.request).then((response) => {
        const fetchPromise = fetch(event.request).then((networkResponse) => {
          cache.put(event.request, networkResponse.clone());
          return networkResponse;
        });
        return response || fetchPromise;
      });
    })
  );
});
```## 戦略の読み込み

### クリティカルレンダリングパス

1. HTMLからDOMを構築する
2. CSSからCSSOMを構築する
3. DOM + CSSOM を結合してレンダー ツリーを作成する
4. レイアウトを計算する
5. ピクセルをペイントする

### リソースのヒント```html
<!-- DNS prefetch -->
<link rel="dns-prefetch" href="//example.com">

<!-- Preconnect (DNS + TCP + TLS) -->
<link rel="preconnect" href="https://fonts.googleapis.com">

<!-- Prefetch (low priority for next page) -->
<link rel="prefetch" href="next-page.js">

<!-- Preload (high priority for current page) -->
<link rel="preload" href="font.woff2" as="font" type="font/woff2" crossorigin>

<!-- Prerender (next page in background) -->
<link rel="prerender" href="next-page.html">
```### 遅延読み込み

#### 画像 - ネイティブの遅延読み込み

    <img src="image.jpg"loading="lazy">```javascript
// Intersection Observer for custom lazy loading
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      const img = entry.target;
      img.src = img.dataset.src;
      observer.unobserve(img);
    }
  });
});

document.querySelectorAll('img[data-src]').forEach(img => {
  observer.observe(img);
});
```### 重要な CSS

インラインのスクロールせずに見える CSS を使用し、残りを延期します。```html
<head>
  <style>
    /* Critical CSS inlined */
    body { margin: 0; font-family: sans-serif; }
    .header { height: 60px; background: #333; }
  </style>
  
  <!-- Non-critical CSS deferred -->
  <link rel="preload" href="styles.css" as="style" onload="this.onload=null;this.rel='stylesheet'">
  <noscript><link rel="stylesheet" href="styles.css"></noscript>
</head>
```## JavaScript のパフォーマンス

### デバウンスとスロットリング```javascript
// Debounce - execute after delay
function debounce(func, delay) {
  let timeoutId;
  return function(...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => func.apply(this, args), delay);
  };
}

// Usage
const handleSearch = debounce((query) => {
  // Search logic
}, 300);

// Throttle - execute at most once per interval
function throttle(func, limit) {
  let inThrottle;
  return function(...args) {
    if (!inThrottle) {
      func.apply(this, args);
      inThrottle = true;
      setTimeout(() => inThrottle = false, limit);
    }
  };
}

// Usage
const handleScroll = throttle(() => {
  // Scroll logic
}, 100);
```### 長いタスク

`requestIdleCallback` と別れる:```javascript
function processLargeArray(items) {
  let index = 0;
  
  function processChunk() {
    const deadline = performance.now() + 50; // 50ms budget
    
    while (index < items.length && performance.now() < deadline) {
      // Process item
      processItem(items[index]);
      index++;
    }
    
    if (index < items.length) {
      requestIdleCallback(processChunk);
    }
  }
  
  requestIdleCallback(processChunk);
}
```### ウェブワーカー

負荷の高い計算をオフロードします。```javascript
// main.js
const worker = new Worker('worker.js');
worker.postMessage({ data: largeDataset });

worker.onmessage = (event) => {
  console.log('Result:', event.data);
};

// worker.js
self.onmessage = (event) => {
  const result = heavyComputation(event.data);
  self.postMessage(result);
};
```## パフォーマンスの監視

### パフォーマンス API```javascript
// Navigation timing
const navTiming = performance.getEntriesByType('navigation')[0];
console.log('DOM loaded:', navTiming.domContentLoadedEventEnd);
console.log('Page loaded:', navTiming.loadEventEnd);

// Resource timing
const resources = performance.getEntriesByType('resource');
resources.forEach(resource => {
  console.log(resource.name, resource.duration);
});

// Mark and measure custom timings
performance.mark('start-task');
// Do work
performance.mark('end-task');
performance.measure('task-duration', 'start-task', 'end-task');

const measure = performance.getEntriesByName('task-duration')[0];
console.log('Task took:', measure.duration, 'ms');

// Observer for performance entries
const observer = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    console.log('Performance entry:', entry);
  }
});
observer.observe({ entryTypes: ['measure', 'mark', 'resource'] });
```### ウェブバイタルライブラリ```javascript
import { getLCP, getFID, getCLS } from 'web-vitals';

getLCP(console.log);
getFID(console.log);
getCLS(console.log);
```## CDN (コンテンツ配信ネットワーク)

コンテンツをグローバル サーバーに分散して、配信を高速化します。

**利点**:
- 待ち時間の短縮
- ロード時間の改善
- 可用性の向上
- 帯域幅コストの削減

**人気の CDN**:
- クラウドフレア
- Amazon CloudFront
- 早く
- アカマイ

## ベストプラクティス

### やるべきこと
- ✅ 画像の最適化 (フォーマット、圧縮、サイズ)
- ✅ コードを縮小して圧縮します
- ✅ キャッシュ戦略を実装する
- ✅ 静的アセットには CDN を使用する
- ✅ 非クリティカルなリソースの遅延読み込み
- ✅ 重要でない JavaScript を延期する
- ✅ インラインクリティカル CSS
- ✅ HTTP/2 または HTTP/3 を使用します
- ✅ コアウェブバイタルを監視
- ✅ パフォーマンスの予算を設定する

### やってはいけないこと
- ❌ 最適化されていない画像を提供する
- ❌ スクリプトによるブロックレンダリング
- ❌ レイアウトがずれる原因となる
- ❌ 過剰な HTTP リクエストを行う
- ❌ 未使用のコードをロードする
- ❌ メインスレッドで同期操作を使用する
- ❌ パフォーマンス指標を無視する
- ❌ モバイルのパフォーマンスのことは忘れてください

## 用語集の用語

**対象となる重要な用語**:
- bfcキャッシュ
- 帯域幅
- ブロトリ圧縮
- コード分割
- 圧縮辞書トランスポート
- 累積レイアウトシフト (CLS)
- デルタ
- ファーストコンテンツフルペイント(FCP)
- 最初の CPU アイドル状態
- 最初の入力遅延 (FID)
- 最初の意味のあるペイント (FMP)
- ファーストペイント(FP)
- グレースフルデグラデーション
- gzip圧縮
- 次のペイントへのインタラクション (INP)
- ジャンク
- ジッター
- 最大のコンテンツフル ペイント (LCP)
- レイテンシー
- 遅延ロード
- 長いタスク
- 可逆圧縮
- 非可逆圧縮
- 縮小化
- ネットワークスロットリング
- ページの読み込み時間
- ページ予測
- 知覚されたパフォーマンス
- プリフェッチ
- プリレンダリング
- 段階的な強化
- レール
- リアルユーザーモニタリング (RUM)
- リフロー
- レンダリングブロッキング
- リペイント
- リソースのタイミング
- 往復時間 (RTT)
- サーバーのタイミング
- スピードインデックス
- 推測的な解析
- 総合モニタリング
- 最初のバイトまでの時間 (TTFB)
- インタラクティブまでの時間 (TTI)
- 木の揺れ
- ウェブパフォーマンス
- Zstandard圧縮

## 追加のリソース

- [Web.devパフォーマンス](https://web.dev/performance/)
- [MDN パフォーマンス](https://developer.mozilla.org/en-US/docs/Web/Performance)
- [WebPageTest](https://www.webpagetest.org/)
- [ライトハウス](https://developers.google.com/web/tools/lighthouse)