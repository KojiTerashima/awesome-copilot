# フレームワーク固有の修正ガイド

このドキュメントでは、各フレームワークとスタイル設定方法の具体的な修正テクニックについて説明します。

---

## 純粋な CSS / SCSS

### レイアウトのオーバーフローを修正する```css
/* Before: Overflow occurs */
.container {
  width: 100%;
}

/* After: Control overflow */
.container {
  width: 100%;
  max-width: 100%;
  overflow-x: hidden;
}
```### テキストクリッピングの防止```css
/* Single line truncation */
.text-truncate {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

/* Multi-line truncation */
.text-clamp {
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

/* Word wrapping */
.text-wrap {
  word-wrap: break-word;
  overflow-wrap: break-word;
  hyphens: auto;
}
```### 間隔の統一```css
/* Unify spacing with CSS custom properties */
:root {
  --spacing-xs: 0.25rem;
  --spacing-sm: 0.5rem;
  --spacing-md: 1rem;
  --spacing-lg: 1.5rem;
  --spacing-xl: 2rem;
}

.card {
  padding: var(--spacing-md);
  margin-bottom: var(--spacing-lg);
}
```### コントラストの向上```css
/* Before: Insufficient contrast */
.text {
  color: #999999;
  background-color: #ffffff;
}

/* After: Meets WCAG AA standards */
.text {
  color: #595959; /* Contrast ratio 7:1 */
  background-color: #ffffff;
}
```---

## 追い風 CSS

### レイアウトの修正```jsx
{/* Before: Overflow */}
<div className="w-full">
  <img src="..." />
</div>

{/* After: Overflow control */}
<div className="w-full max-w-full overflow-hidden">
  <img src="..." className="w-full h-auto object-contain" />
</div>
```### テキストクリッピングの防止```jsx
{/* Single line truncation */}
<p className="truncate">Long text...</p>

{/* Multi-line truncation */}
<p className="line-clamp-3">Long text...</p>

{/* Allow wrapping */}
<p className="break-words">Long text...</p>
```### レスポンシブサポート```jsx
{/* Mobile-first responsive */}
<div className="
  flex flex-col gap-4
  md:flex-row md:gap-6
  lg:gap-8
">
  <div className="w-full md:w-1/2 lg:w-1/3">
    Content
  </div>
</div>
```### 間隔の統一 (Tailwind 構成)```javascript
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      spacing: {
        '18': '4.5rem',
        '22': '5.5rem',
      },
    },
  },
}
```### アクセシビリティの改善```jsx
{/* Add focus state */}
<button className="
  bg-blue-500 text-white
  hover:bg-blue-600
  focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2
">
  Button
</button>

{/* Improve contrast */}
<p className="text-gray-700 bg-white"> {/* Changed from text-gray-500 */}
  Readable text
</p>
```---

## React + CSS モジュール

### モジュール範囲内の修正```css
/* Component.module.css */

/* Before */
.container {
  display: flex;
}

/* After: Add overflow control */
.container {
  display: flex;
  flex-wrap: wrap;
  overflow: hidden;
  max-width: 100%;
}
```### コンポーネント側の修正```jsx
// Component.jsx
import styles from './Component.module.css';

// Before
<div className={styles.container}>

// After: Add conditional class
<div className={`${styles.container} ${isOverflow ? styles.overflow : ''}`}>
```---

## スタイルコンポーネント / 感情

### スタイルの修正```jsx
// Before
const Container = styled.div`
  width: 100%;
`;

// After
const Container = styled.div`
  width: 100%;
  max-width: 100%;
  overflow-x: hidden;
  
  @media (max-width: 768px) {
    padding: 1rem;
  }
`;
```### レスポンシブサポート```jsx
const Card = styled.div`
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 1.5rem;
  
  @media (max-width: 1024px) {
    grid-template-columns: repeat(2, 1fr);
  }
  
  @media (max-width: 640px) {
    grid-template-columns: 1fr;
    gap: 1rem;
  }
`;
```### テーマとの一貫性```jsx
// theme.js
export const theme = {
  colors: {
    primary: '#2563eb',
    text: '#1f2937',
    textLight: '#4b5563', // Improved contrast
  },
  spacing: {
    sm: '0.5rem',
    md: '1rem',
    lg: '1.5rem',
  },
};

// Usage
const Text = styled.p`
  color: ${({ theme }) => theme.colors.text};
  margin-bottom: ${({ theme }) => theme.spacing.md};
`;
```---

## Vue (スコープ付きスタイル)

### スコープ付きスタイルの修正```vue
<template>
  <div class="container">
    <p class="text">Content</p>
  </div>
</template>

<style scoped>
/* Applied only to this component */
.container {
  max-width: 100%;
  overflow: hidden;
}

.text {
  /* Fix: Improve contrast */
  color: #374151; /* Was: #9ca3af */
}

/* Responsive */
@media (max-width: 768px) {
  .container {
    padding: 1rem;
  }
}
</style>
```### ディープセレクター (子コンポーネントに影響)```vue
<style scoped>
/* Override child component styles (use cautiously) */
:deep(.child-class) {
  margin-bottom: 1rem;
}
</style>
```---

## Next.js / アプリルーター

### グローバルスタイルの修正```css
/* app/globals.css */
:root {
  --foreground: #171717;
  --background: #ffffff;
}

/* Prevent layout overflow */
html, body {
  max-width: 100vw;
  overflow-x: hidden;
}

/* Prevent image overflow */
img {
  max-width: 100%;
  height: auto;
}
```### レイアウトコンポーネントの修正```tsx
// app/layout.tsx
export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body className="min-h-screen flex flex-col">
        <header className="sticky top-0 z-50">
          {/* Header */}
        </header>
        <main className="flex-1 container mx-auto px-4 py-8">
          {children}
        </main>
        <footer>
          {/* Footer */}
        </footer>
      </body>
    </html>
  );
}
```---

## 一般的なパターン

### フレックスボックスのレイアウトの問題を修正する```css
/* Before: Items overflow */
.flex-container {
  display: flex;
  gap: 1rem;
}

/* After: Wrap and size control */
.flex-container {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
}

.flex-item {
  flex: 1 1 300px; /* grow, shrink, basis */
  min-width: 0; /* Prevent flexbox overflow issues */
}
```### グリッドレイアウトの問題の修正```css
/* Before: Fixed column count */
.grid-container {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
}

/* After: Auto-adjust */
.grid-container {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1.5rem;
}
```### Z-インデックスの整理```css
/* Systematize z-index */
:root {
  --z-dropdown: 100;
  --z-sticky: 200;
  --z-modal-backdrop: 300;
  --z-modal: 400;
  --z-tooltip: 500;
}

.modal {
  z-index: var(--z-modal);
}
```### フォーカス状態の追加```css
/* Add focus state to all interactive elements */
button:focus-visible,
a:focus-visible,
input:focus-visible,
select:focus-visible,
textarea:focus-visible {
  outline: 2px solid #2563eb;
  outline-offset: 2px;
}

/* Customize focus ring */
.custom-focus:focus-visible {
  outline: none;
  box-shadow: 0 0 0 3px rgba(37, 99, 235, 0.5);
}
```---

## デバッグ手法

### 要素の境界を視覚化する```css
/* Use only during development */
* {
  outline: 1px solid red !important;
}
```### オーバーフローの検出```javascript
// Run in console to detect overflow elements
document.querySelectorAll('*').forEach(el => {
  if (el.scrollWidth > el.clientWidth) {
    console.log('Horizontal overflow:', el);
  }
  if (el.scrollHeight > el.clientHeight) {
    console.log('Vertical overflow:', el);
  }
});
```### コントラスト比の確認```javascript
// Use Chrome DevTools Lighthouse or axe DevTools
// Or check at the following site:
// https://webaim.org/resources/contrastchecker/
```
