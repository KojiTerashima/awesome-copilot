# Framer Motion (Motion v12) — 完全リファレンス

> Framer Motion は 2025 年 منتصفに **Motion** へ名称変更されました。npm パッケージは現在 `motion`、
> import パスは `motion/react` です。API はすべて同一です。`framer-motion` も引き続き動作します。

## 目次
1. [パッケージと Import パス](#package--import-paths)
2. [スクロールアニメーションの2種類](#two-types-of-scroll-animation)
3. [useScroll — オプションリファレンス](#usescroll--options-reference)
4. [useTransform — 完全リファレンス](#usetransform--full-reference)
5. [滑らかさのための useSpring](#usespring-for-smoothing)
6. [Copilot プロンプト付きレシピ](#recipes-with-copilot-prompts)
   - スクロール進捗バー
   - 再利用可能な ScrollReveal ラッパー
   - パララックスレイヤー
   - 横スクロールセクション
   - clipPath を使った画像リビール
   - スクロール連動ナビバー（非表示/表示）
   - スタガー付きカードグリッド
   - スクロール時の 3D チルト
7. [スタガーのための Variants パターン](#variants-pattern-for-stagger)
8. [Motion Value イベント](#motion-value-events)
9. [Next.js & App Router の注意点](#nextjs--app-router-notes)
10. [アクセシビリティ](#accessibility)
11. [Copilot のよくある落とし穴](#common-copilot-pitfalls)

---

## パッケージと Import パス

```bash
npm install motion          # 推奨（2025年に改名）
npm install framer-motion   # 引き続き利用可能 — APIは同一
```

```js
// 推奨（Motion v12+）
import { motion, useScroll, useTransform, useSpring, useMotionValueEvent } from 'motion/react';

// 旧来の書き方 — いまも有効
import { motion, useScroll, useTransform } from 'framer-motion';
```

**Motion v12 の新機能（2025）：**
- ブラウザの ScrollTimeline API によるハードウェアアクセラレーション付きスクロール
- `useScroll` と `scroll()` はデフォルトで GPU アクセラレーション
- 新しい色型：`oklch`、`oklab`、`color-mix` を直接アニメーション可能
- React 19 + concurrent rendering をフルサポート

---

## スクロールアニメーションの2種類

### Scroll-triggered（要素がビューポートに入ったときに一度だけ発火）

```jsx
<motion.div
  initial={{ opacity: 0, y: 50 }}
  whileInView={{ opacity: 1, y: 0 }}
  viewport={{ once: true, margin: '-80px' }}
  transition={{ duration: 0.6, ease: [0.21, 0.47, 0.32, 0.98] }}
>
  Content
</motion.div>
```

`viewport.margin` — 負の値にすると、要素が完全に入る前にアニメーションが始まります。  
`viewport.once` — `true` なら一度だけアニメーションし、逆再生しません。

### Scroll-linked（連続的に、スクロール位置に連動）

```jsx
const { scrollYProgress } = useScroll();
const opacity = useTransform(scrollYProgress, [0, 1], [0, 1]);
return <motion.div style={{ opacity }}>Content</motion.div>;
```

値はスクロールの各フレームで更新されます — `animate` ではなく `style` prop を使う必要があります。

---

## useScroll — オプションリファレンス

```js
const {
  scrollX,          // 絶対水平スクロール量（ピクセル）
  scrollY,          // 絶対垂直スクロール量（ピクセル）
  scrollXProgress,  // offset 間の水平進捗 0→1
  scrollYProgress,  // offset 間の垂直進捗 0→1
} = useScroll({
  // ビューポートではなくスクロール可能要素を追跡
  container: containerRef,

  // コンテナ内での要素位置を追跡
  target: targetRef,

  // 追跡の開始・終了タイミングを定義
  // 形式: ["target position container position", "target position container position"]
  offset: ['start end', 'end start'],
  // よく使う offset ペア:
  // ['start end', 'end start']    = 要素が画面内のどこかにある間を追跡
  // ['start end', 'end end']      = 要素が入ってからページ下端までを追跡
  // ['start start', 'end start']  = 要素が上へ抜ける間を追跡
  // ['center center', 'end start']= 中央一致から退出までを追跡

  // コンテンツサイズ変更時にも更新（わずかなパフォーマンスコスト、デフォルト false）
  trackContentSize: false,
});
```

**Offset 文字列の値:**
- `start` = `0` = 上端/左端
- `center` = `0.5` = 中央
- `end` = `1` = 下端/右端
- 0〜1 の数値も使用可能: `[0, 1]` = `['start', 'end']`

---

## useTransform — 完全リファレンス

```js
// MotionValue をある範囲から別の範囲へマッピング
const y = useTransform(scrollYProgress, [0, 1], [0, -200]);

// 複数停止点の補間
const opacity = useTransform(
  scrollYProgress,
  [0, 0.2, 0.8, 1],
  [0, 1, 1, 0]
);

// 非数値（色、文字列）
const color = useTransform(
  scrollYProgress,
  [0, 0.5, 1],
  ['#6366f1', '#ec4899', '#f97316']
);

// CSS 文字列値
const clipPath = useTransform(
  scrollYProgress,
  [0, 1],
  ['inset(0% 100% 0% 0%)', 'inset(0% 0% 0% 0%)']
);

// クランプを無効化（出力範囲外の値も許可）
const y = useTransform(scrollYProgress, [0, 1], [0, -200], { clamp: false });

// 複数入力から変換
const combined = useTransform(
  [scrollX, scrollY],
  ([x, y]) => Math.sqrt(x * x + y * y)
);
```

**ルール:** `useTransform` の出力は `MotionValue` です。`motion.*` 要素の `style` prop に渡す必要があります。通常の `<div style={{ y }}>` は動作しません — 必ず `<motion.div style={{ y }}>` を使ってください。

---

## 滑らかさのための useSpring

任意の MotionValue を `useSpring` でラップすると、スプリング物理が加わります — 生きたような動きの進捗バーに最適です。

```js
const { scrollYProgress } = useScroll();

const smooth = useSpring(scrollYProgress, {
  stiffness: 100,   // 高いほど速くキビキビ反応
  damping: 30,      // 高いほどバウンスが少ない
  restDelta: 0.001  // 停止判定の精度しきい値
});

return <motion.div style={{ scaleX: smooth }} />;
```

（物理ではなく）わずかな遅れ感を出したい場合は、`useTransform` を `clamp: false` とイージング付き範囲で使ってください。

---

## Copilot プロンプト付きレシピ

### 1. スクロール進捗バー

**Copilot Chat Prompt:**
```
Framer Motion: fixed scroll progress bar at top of page.
useScroll for page scroll progress, useSpring to smooth scaleX.
stiffness 100, damping 30. Grows left to right.
```

```tsx
'use client';
import { useScroll, useSpring, motion } from 'motion/react';

export function ScrollProgressBar() {
  const { scrollYProgress } = useScroll();
  const scaleX = useSpring(scrollYProgress, {
    stiffness: 100, damping: 30, restDelta: 0.001,
  });

  return (
    <motion.div
      style={{ scaleX }}
      className="fixed top-0 left-0 right-0 h-1 bg-indigo-500 origin-left z-50"
    />
  );
}
```

---

### 2. 再利用可能な ScrollReveal ラッパー

**Copilot Chat Prompt:**
```
Framer Motion: reusable ScrollReveal component that wraps children with 
fade-in-up entrance animation using whileInView. Props: delay (default 0), 
duration (default 0.6), once (default true). viewport margin -80px.
TypeScript. 'use client'.
```

```tsx
'use client';
import { motion } from 'motion/react';

interface ScrollRevealProps {
  children: React.ReactNode;
  delay?: number;
  duration?: number;
  once?: boolean;
  className?: string;
}

export function ScrollReveal({
  children, delay = 0, duration = 0.6, once = true, className
}: ScrollRevealProps) {
  return (
    <motion.div
      initial={{ opacity: 0, y: 40 }}
      whileInView={{ opacity: 1, y: 0 }}
      viewport={{ once, margin: '-80px' }}
      transition={{ duration, delay, ease: [0.21, 0.47, 0.32, 0.98] }}
      className={className}
    >
      {children}
    </motion.div>
  );
}

// 使用例:
// <ScrollReveal delay={0.2}><h2>Section Title</h2></ScrollReveal>
```

---

### 3. パララックスレイヤー

**Copilot Chat Prompt:**
```
Framer Motion parallax section: background moves y from 0% to 30% (slow),
foreground text moves y from 50 to -50px (fast). 
Both use target ref with offset ['start end', 'end start'].
Fade out at top and bottom using opacity useTransform [0, 0.3, 0.7, 1] → [0,1,1,0].
```

```tsx
'use client';
import { useRef } from 'react';
import { motion, useScroll, useTransform } from 'motion/react';

export function ParallaxSection() {
  const ref = useRef<HTMLDivElement>(null);
  const { scrollYProgress } = useScroll({
    target: ref,
    offset: ['start end', 'end start'],
  });

  const backgroundY = useTransform(scrollYProgress, [0, 1], ['0%', '30%']);
  const textY        = useTransform(scrollYProgress, [0, 1], [50, -50]);
  const opacity      = useTransform(scrollYProgress, [0, 0.3, 0.7, 1], [0, 1, 1, 0]);

  return (
    <section ref={ref} className="relative h-screen overflow-hidden flex items-center justify-center">
      <motion.div
        className="absolute inset-0 bg-cover bg-center"
        style={{ backgroundImage: 'url(/hero-bg.jpg)', y: backgroundY, scale: 1.2 }}
      />
      <motion.div style={{ y: textY, opacity }} className="relative z-10 text-center text-white">
        <h2 className="text-6xl font-bold">Parallax Title</h2>
        <p className="text-xl mt-4">Scrolls at a different speed</p>
      </motion.div>
    </section>
  );
}
```

---

### 4. 横スクロールセクション

**Copilot Chat Prompt:**
```
Framer Motion horizontal scroll: 4 cards scroll horizontally as user scrolls vertically.
Outer container ref height 300vh controls speed (sticky pattern).
useScroll tracks outer container, useTransform maps scrollYProgress to x '0%' → '-75%'.
```

```tsx
'use client';
import { useRef } from 'react';
import { motion, useScroll, useTransform } from 'motion/react';

const cards = [
  { id: 1, title: 'Card One',   color: 'bg-indigo-500' },
  { id: 2, title: 'Card Two',   color: 'bg-pink-500'   },
  { id: 3, title: 'Card Three', color: 'bg-amber-500'  },
  { id: 4, title: 'Card Four',  color: 'bg-teal-500'   },
];

export function HorizontalScroll() {
  const containerRef = useRef<HTMLDivElement>(null);
  const { scrollYProgress } = useScroll({
    target: containerRef,
    offset: ['start start', 'end end'],
  });

  const x = useTransform(scrollYProgress, [0, 1], ['0%', '-75%']);

  return (
    <div ref={containerRef} className="relative h-[300vh]">
      <div className="sticky top-0 h-screen overflow-hidden">
        <motion.div
          style={{ x, width: `${cards.length * 100}vw` }}
          className="flex gap-6 h-full items-center px-8"
        >
          {cards.map(card => (
            <div
              key={card.id}
              className={`${card.color} w-screen h-[70vh] rounded-2xl flex items-center justify-center flex-shrink-0`}
            >
              <h3 className="text-white text-4xl font-bold">{card.title}</h3>
            </div>
          ))}
        </motion.div>
      </div>
    </div>
  );
}
```

---

### 5. clipPath を使った画像リビール

**Copilot Chat Prompt:**
```
Framer Motion: image reveals left to right as it scrolls into view.
useScroll target ref, offset ['start end', 'center center'].
useTransform clipPath from 'inset(0% 100% 0% 0%)' to 'inset(0% 0% 0% 0%)'.
Also scale from 1.15 to 1.
```

```tsx
'use client';
import { useRef } from 'react';
import { motion, useScroll, useTransform } from 'motion/react';

export function ImageReveal({ src, alt }: { src: string; alt: string }) {
  const ref = useRef<HTMLDivElement>(null);
  const { scrollYProgress } = useScroll({
    target: ref,
    offset: ['start end', 'center center'],
  });

  const clipPath = useTransform(
    scrollYProgress,
    [0, 1],
    ['inset(0% 100% 0% 0%)', 'inset(0% 0% 0% 0%)']
  );
  const scale = useTransform(scrollYProgress, [0, 1], [1.15, 1]);

  return (
    <div ref={ref} className="overflow-hidden rounded-xl">
      <motion.img
        src={src} alt={alt}
        style={{ clipPath, scale }}
        className="w-full h-full object-cover"
      />
    </div>
  );
}
```

---

### 6. スクロール連動ナビバー（下スクロールで隠す）

**Copilot Chat Prompt:**
```
Framer Motion navbar: transparent when at top, white with shadow after 80px.
Hide by sliding up when scrolling down, reveal when scrolling up.
Use useScroll, useMotionValueEvent to detect direction.
Animate y, backgroundColor, boxShadow with motion.nav.
```

```tsx
'use client';
import { useRef, useState } from 'react';
import { motion, useScroll, useMotionValueEvent } from 'motion/react';

export function Navbar() {
  const { scrollY } = useScroll();
  const [scrolled, setScrolled] = useState(false);
  const [hidden,   setHidden]   = useState(false);
  const prevRef = useRef(0);

  useMotionValueEvent(scrollY, 'change', latest => {
    const nextScrolled = latest > 80;
    const nextHidden = latest > prevRef.current && latest > 200;
    setScrolled(current => (current === nextScrolled ? current : nextScrolled));
    setHidden(current => (current === nextHidden ? current : nextHidden));
    prevRef.current = latest;
  });

  return (
    <motion.nav
      animate={{
        y: hidden ? -80 : 0,
        backgroundColor: scrolled ? 'rgba(255,255,255,0.95)' : 'rgba(255,255,255,0)',
        boxShadow: scrolled ? '0 1px 24px rgba(0,0,0,0.08)' : 'none',
      }}
      transition={{ duration: 0.3, ease: 'easeInOut' }}
      className="fixed top-0 left-0 right-0 z-50 backdrop-blur-sm"
    >
      {/* nav links */}
    </motion.nav>
  );
}
```

---

### 7. スタガー付きカードグリッド

**Copilot Chat Prompt:**
```
Framer Motion: card grid with stagger entrance. Use variants: 
container has staggerChildren 0.1, delayChildren 0.2.
Each card: hidden (opacity 0, y 40, scale 0.96) → visible (opacity 1, y 0, scale 1).
Trigger with whileInView on the container. Once.
```

```tsx
'use client';
import { motion } from 'motion/react';

const containerVariants = {
  hidden: {},
  visible: {
    transition: { staggerChildren: 0.1, delayChildren: 0.2 }
  }
};

const cardVariants = {
  hidden:  { opacity: 0, y: 40, scale: 0.96 },
  visible: {
    opacity: 1, y: 0, scale: 1,
    transition: { duration: 0.5, ease: [0.21, 0.47, 0.32, 0.98] }
  }
};

export function CardGrid({ cards }: { cards: { id: number; title: string }[] }) {
  return (
    <motion.div
      variants={containerVariants}
      initial="hidden"
      whileInView="visible"
      viewport={{ once: true, margin: '-50px' }}
      className="grid grid-cols-3 gap-6"
    >
      {cards.map(card => (
        <motion.div key={card.id} variants={cardVariants}
          className="bg-white rounded-xl p-6 shadow-sm border"
        >
          <h3>{card.title}</h3>
        </motion.div>
      ))}
    </motion.div>
  );
}
```

---

### 8. スクロール時の 3D チルト

**Copilot Chat Prompt:**
```
Framer Motion: 3D perspective card that rotates on X axis as it scrolls through view.
rotateX 15→0→-15, scale 0.9→1→0.9, opacity 0→1→0.
Target ref with offset ['start end', 'end start']. Wrap in perspective container.
```

```tsx
'use client';
import { useRef } from 'react';
import { motion, useScroll, useTransform } from 'motion/react';

export function TiltCard({ children }: { children: React.ReactNode }) {
  const ref = useRef<HTMLDivElement>(null);
  const { scrollYProgress } = useScroll({
    target: ref,
    offset: ['start end', 'end start'],
  });

  const rotateX = useTransform(scrollYProgress, [0, 0.5, 1], [15,  0, -15]);
  const scale   = useTransform(scrollYProgress, [0, 0.5, 1], [0.9, 1,  0.9]);
  const opacity = useTransform(scrollYProgress, [0, 0.2, 0.8, 1], [0, 1, 1, 0]);

  return (
    <div ref={ref} style={{ perspective: '1000px' }}>
      <motion.div
        style={{ rotateX, scale, opacity }}
        className="bg-white rounded-2xl p-8 shadow-lg"
      >
        {children}
      </motion.div>
    </div>
  );
}
```

---

## スタガーのための Variants パターン

Variants は親から子へ自動で伝播するため、手動で渡す必要はありません。

```tsx
const parent = {
  hidden: {},
  visible: {
    transition: {
      staggerChildren: 0.1,   // 各子要素の間隔
      delayChildren: 0.2,      // 最初の子要素の前に入る初期待機
      when: 'beforeChildren',  // 親が先にアニメーション
    }
  }
};

const child = {
  hidden: { opacity: 0, y: 20 },
  visible: { opacity: 1, y: 0, transition: { duration: 0.5 } }
};

// `variants={child}` を持つ子要素には自動でスタガーが適用される
// 親が 'hidden' と 'visible' の間で遷移するとき
```

---

## Motion Value イベント

```tsx
import { useScroll, useMotionValueEvent } from 'motion/react';

const { scrollY } = useScroll();

// 変化ごとに発火 — 命令的な副作用に使う
useMotionValueEvent(scrollY, 'change', latest => {
  console.log('scroll position:', latest);
});

// スクロール方向を検出
const [direction, setDirection] = useState<'up' | 'down'>('down');

useMotionValueEvent(scrollY, 'change', current => {
  const diff = current - scrollY.getPrevious()!;
  setDirection(diff > 0 ? 'down' : 'up');
});
```

**`useMotionValueEvent` と `useTransform` の使い分け:**
- `useTransform` は滑らかにアニメーションする CSS 値（y, opacity, color）が欲しいときに使う
- `useMotionValueEvent` は React state の更新や副作用を発火したいときに使う

---

## Next.js & App Router の注意点

```tsx
// motion hooks を使うファイルはすべて Client Component である必要がある
'use client';

// App Router でページ全体のスクロール追跡をする場合は、すでに client component の layout で useScroll を使う
// Server Components で使おうとしないこと

// SSR 安全なスクロールアニメーションが必要なら、次のようにガードする:
import { useEffect, useState } from 'react';
const [mounted, setMounted] = useState(false);
useEffect(() => setMounted(true), []);
if (!mounted) return null; // または skeleton
```

**Next.js App Router の推奨パターン:**
1. すべての `motion.*` コンポーネントを別々の `'use client'` ファイルに置く
2. それらを Server Components から import する（自動的に client-rendered される）
3. ページ遷移にはレイアウト階層で `AnimatePresence` を使う

---

## アクセシビリティ

```tsx
import { useReducedMotion } from 'motion/react';

export function AnimatedCard() {
  const prefersReducedMotion = useReducedMotion();

  return (
    <motion.div
      initial={{ opacity: 0, y: prefersReducedMotion ? 0 : 50 }}
      whileInView={{ opacity: 1, y: 0 }}
      transition={{ duration: prefersReducedMotion ? 0 : 0.6 }}
    >
      Content
    </motion.div>
  );
}
```

または、reduced motion が推奨されるときはスクロール連動 transform をすべて無効化します:
```tsx
const prefersReducedMotion = useReducedMotion();
const y = useTransform(
  scrollYProgress, [0, 1],
  prefersReducedMotion ? [0, 0] : [100, -100]  // reduced motion の場合は移動なし
);
```

---

## Copilot のよくある落とし穴

**'use client' の不足:** Copilot は Next.js App Router ファイルでこれを追加し忘れることがあります。  
`useScroll`、`useTransform`、`motion.*`、または任意の hook を使うファイルには、先頭に `'use client'` が必要です。

**通常の div に style prop を使ってしまう:** Copilot は `y` が MotionValue なのに `<div style={{ y }}>` と書くことがあります。  
これは黙って何も起きません。`<motion.div style={{ y }}>` である必要があります。

**古い import パス:** Copilot はまだ `from 'framer-motion'` を生成することがあります（有効ですが旧来）。  
現在の正規形は `from 'motion/react'` です。

**useScroll で offset を忘れる:** `offset` なしでは `scrollYProgress` はページ全体を  
0 から 1 で追跡し、要素位置にはなりません。要素単位で追跡するなら、常に `target` + `offset` を渡してください。

**target の ref 未接続:** Copilot は `target: ref` だけ書いて DOM 要素への `ref` 接続を忘れることがあります。  
```tsx
const ref = useRef(null);
const { scrollYProgress } = useScroll({ target: ref }); // ← ref を渡す
return <div ref={ref}>...</div>;                          // ← ref を接続する
```

**スクロール連動値に animate prop を使う:** スクロール連動値は `animate` ではなく `style` を使う必要があります。  
`animate` はマウント/アンマウント時に動作し、スクロールでは動きません。
```tsx
// ❌ 間違い
<motion.div animate={{ opacity }} />

// ✅ 正しい
<motion.div style={{ opacity }} />
```

**スクロール進捗を平滑化していない:** 生の `scrollYProgress` は細かな動きで機械的に感じることがあります。  
進捗バーや質感が重要な UI 要素には `useSpring` でラップしてください。

