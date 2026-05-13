---
description: 'Next.js + Tailwind 開発標準と指示'
applyTo: '**/*.tsx, **/*.ts, **/*.jsx, **/*.js, **/*.css'
---

# Next.js + Tailwind 開発指示

TypeScript と Tailwind CSS を用いた高品質な Next.js アプリケーションのための指示です。

## プロジェクトコンテキスト

- 最新の Next.js (App Router)
- 型安全性のための TypeScript
- スタイリングのための Tailwind CSS

## 開発標準

### アーキテクチャ
- Server Component と Client Component を使う App Router
- route は feature/domain ごとにグループ化する
- 適切な error boundary を実装する
- デフォルトでは React Server Components を使う
- 可能な限り static optimization を活用する

### TypeScript
- strict mode を有効にする
- 明確な型定義を行う
- type guard を用いた適切な error handling を行う
- runtime 型検証には Zod を使う

### スタイリング
- 一貫した color palette を備えた Tailwind CSS を使う
- responsive design pattern を採用する
- dark mode をサポートする
- container queries のベストプラクティスに従う
- semantic HTML structure を維持する

### 状態管理
- server state には React Server Components を使う
- client state には React hooks を使う
- 適切な loading state と error state を用意する
- 適切な場合は optimistic updates を使う

### データ取得
- 直接の database query には Server Components を使う
- loading state には React Suspense を使う
- 適切な error handling と retry logic を実装する
- cache invalidation strategy を設計する

### セキュリティ
- 入力検証とサニタイズを行う
- 適切な認証チェックを行う
- CSRF protection を実装する
- rate limiting を実装する
- API route を安全に処理する

### パフォーマンス
- 画像最適化には next/image を使う
- フォント最適化には next/font を使う
- route prefetching を活用する
- 適切な code splitting を行う
- bundle size を最適化する

## 実装プロセス
1. component hierarchy を設計する
2. type と interface を定義する
3. server-side logic を実装する
4. client component を構築する
5. 適切な error handling を追加する
6. responsive styling を実装する
7. loading state を追加する
8. test を書く
