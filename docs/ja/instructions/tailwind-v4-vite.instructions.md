---
description: '公式の @tailwindcss/vite plugin を使った、Vite project 向け Tailwind CSS v4+ のインストールと設定'
applyTo: 'vite.config.ts, vite.config.js, **/*.css, **/*.tsx, **/*.ts, **/*.jsx, **/*.js'
---

# Vite での Tailwind CSS v4+ インストール

公式 Vite plugin を使って Tailwind CSS version 4 以降をインストールおよび設定するための指示です。Tailwind CSS v4 では設定が簡素化され、多くの場合 PostCSS 設定や tailwind.config.js が不要になります。

## Tailwind CSS v4 の主な変更点

- Vite plugin を使う場合、**PostCSS 設定は不要**
- **tailwind.config.js は不要**。設定は CSS 経由で行う
- **新しい `@tailwindcss/vite` plugin** が PostCSS ベースの方式を置き換える
- `@theme` directive を使う **CSS-first 設定**
- **自動 content 検出** により content path の指定が不要

## インストール手順

### Step 1: 依存関係をインストールする

`tailwindcss` と `@tailwindcss/vite` plugin をインストールする:

```bash
npm install tailwindcss @tailwindcss/vite
```

### Step 2: Vite plugin を設定する

Vite の設定ファイルに `@tailwindcss/vite` plugin を追加する:

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import tailwindcss from '@tailwindcss/vite'

export default defineConfig({
  plugins: [
    tailwindcss(),
  ],
})
```

Vite を使う React project の場合:

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import tailwindcss from '@tailwindcss/vite'

export default defineConfig({
  plugins: [
    react(),
    tailwindcss(),
  ],
})
```

### Step 3: Tailwind CSS を import する

メイン CSS file (例: `src/index.css` または `src/App.css`) に Tailwind CSS の import を追加する:

```css
@import "tailwindcss";
```

### Step 4: entry point で CSS import を確認する

アプリケーションの entry point でメイン CSS file が import されていることを確認する:

```typescript
// src/main.tsx or src/main.ts
import './index.css'
```

### Step 5: 開発サーバーを起動する

開発サーバーを実行してインストールを確認する:

```bash
npm run dev
```

## Tailwind v4 でやってはいけないこと

### tailwind.config.js を作成しない

Tailwind v4 は CSS-first 設定を使う。特別な legacy 要件がない限り、`tailwind.config.js` file は作成しない。

```javascript
// ❌ Tailwind v4 では不要
module.exports = {
  content: ['./src/**/*.{js,ts,jsx,tsx}'],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

### Tailwind 用の postcss.config.js を作成しない

`@tailwindcss/vite` plugin を使う場合、Tailwind 向けの PostCSS 設定は不要である。

```javascript
// ❌ @tailwindcss/vite 使用時は不要
module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
}
```

### 古い directive を使わない

旧来の `@tailwind` directive は、単一の import に置き換えられている:

```css
/* ❌ 古い形式 - Tailwind v4 では使わない */
@tailwind base;
@tailwind components;
@tailwind utilities;

/* ✅ 新しい形式 - Tailwind v4 ではこれを使う */
@import "tailwindcss";
```

## CSS-first 設定 (Tailwind v4)

### カスタム theme 設定

design token をカスタマイズするには、CSS で `@theme` directive を使う:

```css
@import "tailwindcss";

@theme {
  --color-primary: #3b82f6;
  --color-secondary: #64748b;
  --font-sans: 'Inter', system-ui, sans-serif;
  --radius-lg: 0.75rem;
}
```

### カスタム utility を追加する

カスタム utility は CSS で直接定義する:

```css
@import "tailwindcss";

@utility content-auto {
  content-visibility: auto;
}

@utility scrollbar-hidden {
  scrollbar-width: none;
  &::-webkit-scrollbar {
    display: none;
  }
}
```

### カスタム variant を追加する

カスタム variant は CSS で定義する:

```css
@import "tailwindcss";

@variant hocus (&:hover, &:focus);
@variant group-hocus (:merge(.group):hover &, :merge(.group):focus &);
```

## 確認チェックリスト

インストール後に次を確認する:

- [ ] `tailwindcss` と `@tailwindcss/vite` が `package.json` の dependencies にある
- [ ] `vite.config.ts` に `tailwindcss()` plugin が含まれている
- [ ] メイン CSS file に `@import "tailwindcss";` がある
- [ ] CSS file がアプリケーション entry point で import されている
- [ ] 開発サーバーがエラーなく起動する
- [ ] Tailwind utility class (例: `text-blue-500`、`p-4`) が正しく描画される

## 使用例

単純な component でインストールを確認する:

```tsx
export function TestComponent() {
  return (
    <div className="min-h-screen bg-gray-100 flex items-center justify-center">
      <h1 className="text-3xl font-bold text-blue-600 underline">
        Hello, Tailwind CSS v4!
      </h1>
    </div>
  )
}
```

## トラブルシューティング

### style が適用されない

1. CSS import 文が `@import "tailwindcss";` であることを確認する (古い directive ではないこと)
2. CSS file が entry point で import されていることを確認する
3. Vite 設定に `tailwindcss()` plugin が含まれていることを確認する
4. Vite cache をクリアする: `rm -rf node_modules/.vite && npm run dev`

### Plugin Not Found Error

"Cannot find module '@tailwindcss/vite'" と表示される場合:

```bash
npm install @tailwindcss/vite
```

### TypeScript Error

TypeScript が Vite plugin の型を見つけられない場合は、import が正しいことを確認する:

```typescript
import tailwindcss from '@tailwindcss/vite'
```

## Tailwind v3 からの移行

Tailwind v3 から移行する場合:

1. `tailwind.config.js` を削除する (`@theme` を使う CSS へカスタマイズを移す)
2. `postcss.config.js` を削除する (Tailwind のためだけに使っていた場合)
3. 古い package をアンインストールする: `npm uninstall postcss autoprefixer`
4. 新しい package をインストールする: `npm install tailwindcss @tailwindcss/vite`
5. `@tailwind` directive を `@import "tailwindcss";` に置き換える
6. Vite 設定を更新して `@tailwindcss/vite` plugin を使う

## 参照

- Official Documentation: https://tailwindcss.com/docs/installation/using-vite
- Tailwind CSS v4 Upgrade Guide: https://tailwindcss.com/docs/upgrade-guide
