# GSAP ScrollTrigger — 完全リファレンス

## 目次
1. [インストールと登録](#installation--registration)
2. [ScrollTrigger 設定リファレンス](#scrolltrigger-config-reference)
3. [Start / End 構文の読み解き](#start--end-syntax-decoded)
4. [toggleActions の値](#toggleactions-values)
5. [Copilot プロンプト付きレシピ](#recipes-with-copilot-prompts)
   - フェードインのバッチ表示
   - スクラブアニメーション
   - ピン留めタイムライン
   - パララックスレイヤー
   - 水平スクロール
   - 文字単位のスタガーテキスト
   - スクロールスナップ
   - プログレスバー
   - ScrollSmoother
   - スクロールカウンター
6. [React 統合 (useGSAP)](#react-integration-usegsap)
7. [Lenis スムーススクロール](#lenis-smooth-scroll)
8. [matchMedia を使ったレスポンシブ対応](#responsive-with-matchmedia)
9. [アクセシビリティ](#accessibility)
10. [パフォーマンスとクリーンアップ](#performance--cleanup)
11. [Copilot でよくある落とし穴](#common-copilot-pitfalls)

---

## インストールと登録

```bash
npm install gsap
# React
npm install gsap @gsap/react
```

```js
import gsap from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';
import { ScrollSmoother } from 'gsap/ScrollSmoother'; // optional
gsap.registerPlugin(ScrollTrigger, ScrollSmoother);
```

CDN（vanilla）:
```html
<script src="https://cdn.jsdelivr.net/npm/gsap@3.14/dist/gsap.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/gsap@3.14/dist/ScrollTrigger.min.js"></script>
```

---

## ScrollTrigger 設定リファレンス

```js
gsap.to('.element', {
  x: 500,
  ease: 'none',          // スクラブアニメーションでは 'none' を使う
  scrollTrigger: {
    trigger: '.section',         // この要素の位置がアニメーション開始のトリガーになる
    start: 'top 80%',            // "[trigger edge] [viewport edge]"
    end: 'bottom 20%',           // アニメーションの終了位置
    scrub: 1,                    // 進行度をスクロールに連動; 即時連動は true、なめらかな遅延は数値
    pin: true,                   // スクロール中に trigger 要素を固定; 別要素を固定する場合は selector/element を指定
    pinSpacing: true,            // 固定要素の下に余白を追加（デフォルト: true）
    markers: true,               // デバッグ用マーカー — 本番では削除
    toggleActions: 'play none none reverse', // onEnter onLeave onEnterBack onLeaveBack
    toggleClass: 'active',       // アクティブ時に付与/解除される CSS クラス
    snap: { snapTo: 'labels', duration: 0.3, ease: 'power1.inOut' }, // 1 のような数値で増分スナップも可能
    fastScrollEnd: true,         // ユーザーが高速スクロールした場合に強制的に完了
    horizontal: false,           // 水平スクロールコンテナでは true
    anticipatePin: 1,            // pin 時のジャンプを軽減（先読み秒数）
    invalidateOnRefresh: true,   // リサイズ時に位置を再計算
    id: 'my-trigger',            // ScrollTrigger.getById() 用
    onEnter: () => {},
    onLeave: () => {},
    onEnterBack: () => {},
    onLeaveBack: () => {},
    onUpdate: self => console.log(self.progress), // 0 から 1
    onToggle: self => console.log(self.isActive),
  }
});
```

---

## Start / End 構文の読み解き

形式: `"[trigger position] [viewport position]"`

| 値 | 意味 |
|---|---|
| `"top bottom"` | trigger の上端が viewport の下端に達する — 画面内に入る |
| `"top 80%"` | trigger の上端が viewport 上端から 80% の位置に到達 |
| `"top center"` | trigger の上端が viewport の中央に到達 |
| `"top top"` | trigger の上端が viewport の上端に一致 |
| `"center center"` | 中央同士が一致 |
| `"bottom top"` | trigger の下端が viewport の上端に達する — 画面外に出る |
| `"+=200"` | trigger 位置から 200px 後 |
| `"-=100"` | trigger 位置から 100px 前 |
| `"+=200%"` | trigger 位置から viewport 高さの 200% 後 |

---

## toggleActions の値

```
toggleActions: "play pause resume reset"
                ^      ^      ^        ^
              onEnter onLeave onEnterBack onLeaveBack
```

| 値 | 効果 |
|---|---|
| `play` | 現在位置から再生 |
| `pause` | 現在位置で一時停止 |
| `resume` | 停止位置から再開 |
| `reverse` | 逆再生 |
| `reset` | 開始位置へジャンプ |
| `restart` | 最初から再生 |
| `none` | 何もしない |

入場アニメーションで最も一般的: `"play none none none"`（1回だけ再生し、逆再生しない）。

---

## Copilot プロンプト付きレシピ

### 1. フェードインのバッチ表示

**Copilot Chat Prompt:**
```
Using GSAP ScrollTrigger.batch, animate all .card elements: 
fade in from opacity 0, y 50 when they enter the viewport at 85%.
Stagger 0.15s between cards. Animate once (no reverse).
```

```js
gsap.registerPlugin(ScrollTrigger);

ScrollTrigger.batch('.card', {
  onEnter: elements => {
    gsap.from(elements, {
      opacity: 0,
      y: 50,
      stagger: 0.15,
      duration: 0.8,
      ease: 'power2.out',
    });
  },
  start: 'top 85%',
});
```

個別の ScrollTrigger ではなく `batch` を使う理由: 同時に入ってくる要素を1回のアニメーション呼び出しにまとめられるため、要素ごとに ScrollTrigger を作るより高パフォーマンスです。

---

### 2. スクラブアニメーション（スクロール連動）

**Copilot Chat Prompt:**
```
GSAP scrub: animate .hero-image scale from 1 to 1.3 and opacity to 0
as the user scrolls past .hero-section. 
Perfectly synced to scroll position, no pin.
```

```js
gsap.to('.hero-image', {
  scale: 1.3,
  opacity: 0,
  ease: 'none',   // 重要: scrub では線形イージング
  scrollTrigger: {
    trigger: '.hero-section',
    start: 'top top',
    end: 'bottom top',
    scrub: true,
  }
});
```

---

### 3. ピン留めタイムライン

**Copilot Chat Prompt:**
```
GSAP pinned timeline: pin .story-section while a sequence plays —
fade in .title (y: 60), scale .image to 1, slide .text from x: 80.
Total scroll distance 300vh. Scrub 1 for smoothness.
```

```js
const tl = gsap.timeline({
  scrollTrigger: {
    trigger: '.story-section',
    start: 'top top',
    end: '+=300%',
    pin: true,
    scrub: 1,
    anticipatePin: 1,
  }
});

tl
  .from('.title',  { opacity: 0, y: 60, duration: 1 })
  .from('.image',  { scale: 0.85, opacity: 0, duration: 1 }, '-=0.3')
  .from('.text',   { x: 80, opacity: 0, duration: 1 }, '-=0.3');
```

---

### 4. パララックスレイヤー

**Copilot Chat Prompt:**
```
GSAP parallax: background image moves yPercent -20 (slow),
foreground text moves yPercent -60 (fast). Both scrubbed to scroll, no pin.
Trigger is .parallax-section, start top bottom, end bottom top.
```

```js
// 遅い背景
gsap.to('.parallax-bg', {
  yPercent: -20,
  ease: 'none',
  scrollTrigger: {
    trigger: '.parallax-section',
    start: 'top bottom',
    end: 'bottom top',
    scrub: true,
  }
});

// 速い前景
gsap.to('.parallax-fg', {
  yPercent: -60,
  ease: 'none',
  scrollTrigger: {
    trigger: '.parallax-section',
    start: 'top bottom',
    end: 'bottom top',
    scrub: true,
  }
});
```

---

### 5. 水平スクロールセクション

**Copilot Chat Prompt:**
```
GSAP horizontal scroll: 4 .panel elements inside .panels-container.
Pin .horizontal-section, scrub 1, snap per panel.
End should use offsetWidth so it recalculates on resize.
```

```js
const sections = gsap.utils.toArray('.panel');

gsap.to(sections, {
  xPercent: -100 * (sections.length - 1),
  ease: 'none',
  scrollTrigger: {
    trigger: '.horizontal-section',
    pin: true,
    scrub: 1,
    snap: 1 / (sections.length - 1),
    end: () => `+=${document.querySelector('.panels-container').offsetWidth}`,
    invalidateOnRefresh: true,
  }
});
```

必要な HTML:
```html
<div class="horizontal-section">
  <div class="panels-container">
    <div class="panel">1</div>
    <div class="panel">2</div>
    <div class="panel">3</div>
    <div class="panel">4</div>
  </div>
</div>
```

必要な CSS:
```css
.horizontal-section { overflow: hidden; }
.panels-container   { display: flex; flex-wrap: nowrap; width: 400vw; }
.panel              { width: 100vw; height: 100vh; flex-shrink: 0; }
```

---

### 6. 文字単位のスタガーテキスト表示

**Copilot Chat Prompt:**
```
Split .hero-title into characters using SplitType.
Animate each char: opacity 0→1, y 80→0, rotateX -90→0.
Stagger 0.03s, ease back.out(1.7). Trigger when heading enters at 85%.
```

```bash
npm install split-type
```

```js
import SplitType from 'split-type';

const text = new SplitType('.hero-title', { types: 'chars' });

gsap.from(text.chars, {
  opacity: 0,
  y: 80,
  rotateX: -90,
  stagger: 0.03,
  duration: 0.6,
  ease: 'back.out(1.7)',
  scrollTrigger: {
    trigger: '.hero-title',
    start: 'top 85%',
    toggleActions: 'play none none none',
  }
});
```

---

### 7. スクロールスナップセクション

**Copilot Chat Prompt:**
```
GSAP: each full-height section scales from 0.9 to 1 when it enters view.
Also add global scroll snapping between sections using ScrollTrigger.create snap.
```

```js
const sections = gsap.utils.toArray('section');

sections.forEach(section => {
  gsap.from(section, {
    scale: 0.9,
    opacity: 0.6,
    scrollTrigger: {
      trigger: section,
      start: 'top 90%',
      toggleActions: 'play none none reverse',
    }
  });
});

ScrollTrigger.create({
  snap: {
    snapTo: (progress) => {
      const step = 1 / (sections.length - 1);
      return Math.round(progress / step) * step;
    },
    duration: { min: 0.2, max: 0.5 },
    ease: 'power1.inOut',
  }
});
```

---

### 8. スクロール進捗バー

**Copilot Chat Prompt:**
```
GSAP: fixed progress bar at top of page. scaleX 0→1 linked to 
full page scroll, scrub 0.3 for slight smoothing. transformOrigin left center.
```

```js
gsap.to('.progress-bar', {
  scaleX: 1,
  ease: 'none',
  transformOrigin: 'left center',
  scrollTrigger: {
    trigger: document.body,
    start: 'top top',
    end: 'bottom bottom',
    scrub: 0.3,
  }
});
```

```css
.progress-bar {
  position: fixed; top: 0; left: 0;
  width: 100%; height: 4px;
  background: #6366f1;
  transform-origin: left;
  transform: scaleX(0);
  z-index: 999;
}
```

---

### 9. ScrollSmoother のセットアップ

**Copilot Chat Prompt:**
```
Set up GSAP ScrollSmoother with smooth: 1.5, effects: true.
Show the required wrapper HTML structure.
Add data-speed and data-lag to parallax elements.
```

```bash
# ScrollSmoother は gsap の一部 — 追加インストール不要
```

```js
import { ScrollSmoother } from 'gsap/ScrollSmoother';
gsap.registerPlugin(ScrollTrigger, ScrollSmoother);

ScrollSmoother.create({
  wrapper: '#smooth-wrapper',
  content: '#smooth-content',
  smooth: 1.5,
  effects: true,
  smoothTouch: 0.1,
});
```

```html
<div id="smooth-wrapper">
  <div id="smooth-content">
    <img data-speed="0.5" src="bg.jpg" />      <!-- スクロール速度 50% -->
    <div data-lag="0.3" class="float">...</div> <!-- 0.3秒の遅延 -->
  </div>
</div>
```

---

### 10. アニメーション数値カウンター

**Copilot Chat Prompt:**
```
GSAP: animate .counter elements from 0 to their data-target value 
when they enter the viewport. Duration 2s, ease power2.out.
Format with toLocaleString. Animate once.
```

```js
document.querySelectorAll('.counter').forEach(el => {
  const obj = { val: 0 };
  gsap.to(obj, {
    val: parseInt(el.dataset.target, 10),
    duration: 2,
    ease: 'power2.out',
    onUpdate: () => { el.textContent = Math.round(obj.val).toLocaleString(); },
    scrollTrigger: {
      trigger: el,
      start: 'top 85%',
      toggleActions: 'play none none none',
    }
  });
});
```

```html
<span class="counter" data-target="12500">0</span>
```

---

## React 統合 (useGSAP)

```bash
npm install gsap @gsap/react
```

```jsx
import { useRef } from 'react';
import { useGSAP } from '@gsap/react';
import gsap from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';

gsap.registerPlugin(useGSAP, ScrollTrigger);
```

**useEffect ではなく useGSAP を使う理由:**
`useGSAP` はコンポーネントのアンマウント時に、その中で作成されたすべての ScrollTrigger を自動で kill するため、メモリリークを防げます。React strict mode の二重実行にも正しく対応します。GSAP を理解した `useLayoutEffect` の置き換えだと考えてください。

**Copilot Chat Prompt:**
```
React: use useGSAP from @gsap/react to animate .card elements inside containerRef.
Fade in from y 60, opacity 0, stagger 0.12, scrollTrigger start top 80%.
Scope to containerRef so selectors don't match outside this component.
```

```jsx
export function AnimatedSection() {
  const containerRef = useRef(null);

  useGSAP(() => {
    gsap.from('.card', {
      opacity: 0,
      y: 60,
      stagger: 0.12,
      duration: 0.7,
      ease: 'power2.out',
      scrollTrigger: {
        trigger: containerRef.current,
        start: 'top 80%',
        toggleActions: 'play none none none',
      }
    });
  }, { scope: containerRef });

  return (
    <div ref={containerRef}>
      <div className="card">One</div>
      <div className="card">Two</div>
    </div>
  );
}
```

**React でのピン留めタイムライン:**
```jsx
export function PinnedStory() {
  const sectionRef = useRef(null);

  useGSAP(() => {
    const tl = gsap.timeline({
      scrollTrigger: {
        trigger: sectionRef.current,
        pin: true, scrub: 1,
        start: 'top top', end: '+=200%',
      }
    });
    tl.from('.story-title', { opacity: 0, y: 40 })
      .from('.story-image', { scale: 0.85, opacity: 0 }, '-=0.2')
      .from('.story-text',  { opacity: 0, x: 40 }, '-=0.2');
  }, { scope: sectionRef });

  return (
    <section ref={sectionRef}>
      <h2 className="story-title">Chapter One</h2>
      <img className="story-image" src="/photo.jpg" alt="" />
      <p className="story-text">The story begins.</p>
    </section>
  );
}
```

**Next.js の注意点:** `gsap.registerPlugin(ScrollTrigger)` は `useGSAP` または `useLayoutEffect` 内で実行するか、次のようにガードしてください:
```js
if (typeof window !== 'undefined') gsap.registerPlugin(ScrollTrigger);
```

---

## Lenis スムーススクロール

```bash
npm install lenis
```

**Copilot Chat Prompt:**
```
Integrate Lenis smooth scroll with GSAP ScrollTrigger.
Add lenis.raf to gsap.ticker. Set lagSmoothing to 0.
Destroy lenis on unmount if in React.
```

```js
import Lenis from 'lenis';
import { useEffect } from 'react';

const lenis = new Lenis({ duration: 1.2, smoothWheel: true });

const raf = (time) => lenis.raf(time * 1000);
gsap.ticker.add(raf);
gsap.ticker.lagSmoothing(0);
lenis.on('scroll', ScrollTrigger.update);

// React クリーンアップ
useEffect(() => {
  return () => {
    lenis.destroy();
    gsap.ticker.remove(raf);
  };
}, []);
```

---

## matchMedia を使ったレスポンシブ対応

**Copilot Chat Prompt:**
```
Use gsap.matchMedia to animate x: 200 on desktop (min-width: 768px)
and y: 100 on mobile. Both should skip animation if prefers-reduced-motion is set.
```

```js
const mm = gsap.matchMedia();

mm.add({
  isDesktop: '(min-width: 768px)',
  isMobile:  '(max-width: 767px)',
  noMotion:  '(prefers-reduced-motion: reduce)',
}, context => {
  const { isDesktop, isMobile, noMotion } = context.conditions;
  if (noMotion) return;

  gsap.from('.box', {
    x: isDesktop ? 200 : 0,
    y: isMobile  ? 100 : 0,
    opacity: 0,
    scrollTrigger: { trigger: '.box', start: 'top 80%' }
  });
});
```

---

## アクセシビリティ

```js
// すべてのスクロールアニメーションを prefers-reduced-motion でガード
const prefersReducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)');

if (!prefersReducedMotion.matches) {
  gsap.from('.box', {
    opacity: 0, y: 50,
    scrollTrigger: { trigger: '.box', start: 'top 85%' }
  });
} else {
  // アニメーションせず、要素を即時表示
  gsap.set('.box', { opacity: 1, y: 0 });
}
```

または `prefers-reduced-motion: reduce` 条件付きで `gsap.matchMedia()` を使います（上記参照）。

---

## パフォーマンスとクリーンアップ

```js
// 特定のトリガーを kill
const st = ScrollTrigger.create({ ... });
st.kill();

// すべてのトリガーを kill（例: ページ遷移時）
ScrollTrigger.killAll();

// すべてのトリガー位置を再計算（動的コンテンツ読み込み後）
ScrollTrigger.refresh();
```

**パフォーマンスルール:**
- アニメーションは `transform` と `opacity` のみにする — GPU アクセラレーションされ、レイアウト再計算が発生しない
- `width`, `height`, `top`, `left`, `box-shadow`, `filter` のアニメーションは避ける
- 似た要素が多い場合は `ScrollTrigger.batch()` を使う — 要素ごとに1トリガーよりはるかに効率的
- `will-change: transform` は必要最小限に — 実際にアニメーション中の要素だけ
- `markers: true` は本番前に必ず削除

---

## Copilot でよくある落とし穴

**registerPlugin の書き忘れ:** Copilot は `gsap.registerPlugin(ScrollTrigger)` を省略しがちです。  
ScrollTrigger を使う前に必ず追加してください。

**scrub で不適切な ease:** Copilot は scrub アニメーションでも `power2.out` を既定にしがちです。  
`scrub: true` または `scrub: number` のときは、必ず `ease: 'none'` を使ってください。

**React で useGSAP ではなく useEffect を使う:** Copilot は `useEffect` を生成しがちです — 必ず `useGSAP` に置き換えてください。

**水平スクロールで end が固定値:** Copilot は `end: "+=" + container.offsetWidth` と書きがちです。  
正解: `end: () => "+=" + container.offsetWidth`（関数形式ならリサイズ時に再計算される）。

**本番に markers が残る:** Copilot は `markers: true` を追加したままにしがちです。必ず削除してください。

**長いアニメーションで pin なしの scrub:** 長いタイムラインを pin せずに scrub すると、  
要素が画面外へ流れてしまいます。`pin: true` を追加するか、スクロール距離を短くしてください。

