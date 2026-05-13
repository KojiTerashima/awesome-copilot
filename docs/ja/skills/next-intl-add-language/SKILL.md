---
name: next-intl-add-language
description: 'Next.js + next-intlアプリケーションに新しい言語を追加する'
---

これは、Next.jsプロジェクトにnext-intlを使って新しい言語を追加するためのガイドです。

- i18nにはnext-intlを使用しています。
- すべての翻訳は`./messages`ディレクトリにあります。
- UIコンポーネントは`src/components/language-toggle.tsx`です。
- ルーティングとミドルウェアの設定は以下で管理されています：
  - `src/i18n/routing.ts`
  - `src/middleware.ts`

新しい言語を追加する際は：

- `en.json`の内容をすべて新しい言語に翻訳してください。目的は完全な翻訳のためにすべてのJSONエントリを新しい言語で揃えることです。
- `routing.ts`と`middleware.ts`にパスを追加してください。
- `language-toggle.tsx`に言語を追加してください。
