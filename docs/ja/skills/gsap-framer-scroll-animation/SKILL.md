---
name: gsap-framer-scroll-animation
description: >-
  ユーザーがスクロールアニメーション、スクロール効果、
  パララックス、スクロールトリガーによるリビール、ピン留めセクション、横スクロール、テキストアニメーション、
  またはスクロール位置に連動するあらゆるモーションを作りたいときは、vanilla JS、React、Next.js を問わず
  このスキルを使ってください。
  GSAP ScrollTrigger（pinning、scrubbing、snapping、timelines、horizontal scroll、
  ScrollSmoother、matchMedia）と Framer Motion / Motion v12（useScroll、useTransform、
  useSpring、whileInView、variants）をカバーします。ユーザーが単に
  「animate on scroll」「fade in as I scroll」「make it scroll like Apple」
  「parallax effect」「sticky section」「scroll progress bar」「entrance animation」
  と言っただけでもこのスキルを使ってください。
  また、GSAP または Framer Motion のコード生成に関する Copilot のプロンプトパターンにも対応します。
  クリエイティブ哲学やデザイン品質の磨き込みには premium-frontend-ui スキルと組み合わせてください。
metadata:
  author: 'Utkarsh Patrikar'
  author_url: 'https://github.com/utkarsh232005'
---

# GSAP & Framer Motion — スクロールアニメーションスキル

GitHub Copilot のプロンプト、すぐに使えるコードレシピ、詳細な API リファレンスを備えた、本番品質のスクロールアニメーション。

> **デザインコンパニオン:** このスキルは、スクロール駆動モーションの*技術実装*を提供します。  
> アニメーションの**方法**と**タイミング**を導くべき*クリエイティブ哲学*、デザイン原則、プレミアムな美的感覚については、
> 必ず **premium-frontend-ui** スキルをあわせて参照してください。  
> 両者を組み合わせることで完全なアプローチになります: premium-frontend-ui が **what** と **why** を決め、
> このスキルが **how** を実現します。

## クイックライブラリ選択

| Need | Use |
|---|---|
| Vanilla JS, Webflow, Vue | **GSAP** |
| Pinning, horizontal scroll, complex timelines | **GSAP** |
| React / Next.js, declarative style | **Framer Motion** |
| whileInView entrance animations | **Framer Motion** |
| Both in same Next.js app | See notes in references |

完全なレシピと Copilot プロンプトは、該当するリファレンスファイルを参照してください:

- **GSAP** → `references/gsap.md` — ScrollTrigger API、全レシピ、React 統合
- **Framer Motion** → `references/framer.md` — useScroll、useTransform、全レシピ

## セットアップ（必ず最初に実施）

### GSAP
```bash
npm install gsap
```
```js
import gsap from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';
gsap.registerPlugin(ScrollTrigger); // MUST call before any ScrollTrigger usage
```

### Framer Motion (Motion v12, 2025)
```bash
npm install motion   # new package name since mid-2025
# or: npm install framer-motion  — still works, same API
```
```js
import { motion, useScroll, useTransform, useSpring } from 'motion/react';
// legacy: import { motion } from 'framer-motion'  — also valid
```

## ワークフロー

1. ユーザーの意図を解釈し、GSAP と Framer Motion のどちらが最適かを判断します。
2. 詳細な API とパターンを確認するため、`references/` 内の該当リファレンス文書を読みます。
3. 必要なパッケージが未導入であれば、インストールを提案します。
4. 要求された形式（React コンポーネント、フック要件、または vanilla JS）に従って、アニメーション構造の骨組みを実装します。
5. 適切な手法（スクロール連動か in-view 要素か）を適用し、アクセシビリティ対応があること、フックが無限再レンダーを起こさないことを確認します。

## 最もよく使うスクロールパターン 5 選

クイックリファレンス — Copilot プロンプト付きの完全なレシピはリファレンスファイルにあります。

### 1. 画面進入時フェードイン（GSAP）
```js
gsap.from('.card', {
  opacity: 0, y: 50, stagger: 0.15, duration: 0.8,
  scrollTrigger: { trigger: '.card', start: 'top 85%' }
});
```

### 2. 画面進入時フェードイン（Framer Motion）
```jsx
<motion.div
  initial={{ opacity: 0, y: 40 }}
  whileInView={{ opacity: 1, y: 0 }}
  viewport={{ once: true, margin: '-80px' }}
  transition={{ duration: 0.6 }}
/>
```

### 3. Scrub / スクロール連動（GSAP）
```js
gsap.to('.hero-img', {
  scale: 1.3, opacity: 0, ease: 'none',
  scrollTrigger: { trigger: '.hero', start: 'top top', end: 'bottom top', scrub: true }
});
```

### 4. スクロール連動（Framer Motion）
```jsx
const { scrollYProgress } = useScroll({ target: ref, offset: ['start end', 'end start'] });
const y = useTransform(scrollYProgress, [0, 1], [0, -100]);
return <motion.div style={{ y }} />;
```

### 5. ピン留めタイムライン（GSAP）
```js
const tl = gsap.timeline({
  scrollTrigger: { trigger: '.section', pin: true, scrub: 1, start: 'top top', end: '+=200%' }
});
tl.from('.title', { opacity: 0, y: 60 }).from('.img', { scale: 0.85 });
```

## 重要ルール（常に適用）

- **GSAP**: 使用前に必ず `gsap.registerPlugin(ScrollTrigger)` を呼び出す
- **GSAP scrub**: 必ず `ease: 'none'` を使う — scrub 有効時にイージングは不自然になる
- **GSAP React**: `@gsap/react` の `useGSAP` を使い、素の `useEffect` は使わない — ScrollTrigger を自動でクリーンアップする
- **GSAP debug**: 開発中は `markers: true` を追加し、本番前に削除する
- **Framer**: `useTransform` の出力は通常の div ではなく `motion.*` 要素の `style` prop に渡す
- **Framer Next.js**: motion フックを使うファイルの先頭には必ず `'use client'` を追加する
- **両方**: `transform` と `opacity` のみをアニメーション対象にする — `width`、`height`、`box-shadow` は避ける
- **アクセシビリティ**: 必ず `prefers-reduced-motion` を確認する — パターンは各リファレンスファイルを参照
- **プレミアムな仕上げ**: モーションのタイミング、イージングカーブ、抑制の効いた演出には **premium-frontend-ui** スキルの原則を適用する — アニメーションは主張しすぎず、体験を高めるべき

## Copilot プロンプトのコツ

- セレクター、ベース画像、スクロール範囲を最初から Copilot に渡す — あいまいなプロンプトはあいまいなコードを生む
- GSAP では必ず指定する: セレクター、start/end 文字列、scrub か toggleActions か
- Framer では必ず指定する: どのフックを使うか（useScroll vs whileInView）、offset 値、何を transform するか
- `/fix` を依頼するときはエラーメッセージをそのまま貼る — 実際のエラーがあると Copilot の修正精度は大幅に上がる
- Copilot Chat では `@workspace` スコープを使い、既存コンポーネント構造を読ませる

## リファレンスファイル

| File | Contents |
|---|---|
| `references/gsap.md` | ScrollTrigger API 完全リファレンス、10 個のレシピ、React（useGSAP）、Lenis、matchMedia、アクセシビリティ |
| `references/framer.md` | useScroll / useTransform API 完全リファレンス、8 個のレシピ、variants、Motion v12 の注意点、Next.js のヒント |

## 関連スキル

| Skill | Relationship |
|---|---|
| **premium-frontend-ui** | クリエイティブ哲学、デザイン原則、美的ガイドライン — アニメーションの *when* と *why* を定義 |

