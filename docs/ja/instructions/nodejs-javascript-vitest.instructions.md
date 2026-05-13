---
description: "Vitest テストを伴う Node.js と JavaScript コードを書くためのガイドライン"
applyTo: '**/*.js, **/*.mjs, **/*.cjs'
---

# コード生成ガイドライン

## コーディング標準
- JavaScript は ES2022 の機能と Node.js (20+) の ESM modules を使う
- 可能な限り Node.js built-in modules を使い、外部依存は避ける
- 追加の依存関係が必要な場合は、追加する前にユーザーへ確認する
- 非同期コードには常に async/await を使い、callback を避けるために 'node:util' の promisify function を使う
- コードはシンプルで保守しやすく保つ
- 変数名と関数名は説明的にする
- コメントは本当に必要な場合を除いて追加しない。コード自体が自明であるべき
- `null` は決して使わず、オプショナル値には常に `undefined` を使う
- class より function を優先する

## テスト
- テストには Vitest を使う
- すべての新機能とバグ修正に対してテストを書く
- テストでは edge case と error handling をカバーする
- テストしやすくするために元のコードを変更してはいけない。元のコードをそのまま対象にするテストを書くこと

## ドキュメント
- 新機能の追加や大きな変更を行う場合は、必要に応じて README.md を更新する

## ユーザーとのやり取り
- 実装の詳細、設計の選択、要件が不明な場合は質問する
- 常に質問と同じ言語で回答するが、生成する内容（code、comments、docs）は english を使う
