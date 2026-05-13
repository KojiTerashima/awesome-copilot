---
description: 'Azure Functions の TypeScript パターン'
applyTo: '**/*.ts, **/*.js, **/*.json'
---

## コード生成のガイダンス
- Node.js 用の最新の TypeScript コードを生成する
- 非同期コードには `async/await` を使用します
- 可能な限り、外部パッケージの代わりに Node.js v20 組み込みモジュールを使用してください。
- イベントループのブロックを避けるために、`fs` の代わりに `node:fs/promises` などの Node.js 非同期関数を常に使用してください。
- プロジェクトに追加の依存関係を追加する前に質問する
- API は、`@azure/functions@4` パッケージを使用して Azure Functions を使用して構築されます。
- 各エンドポイントには独自の関数ファイルが必要であり、次の命名規則を使用する必要があります: `src/functions/<resource-name>-<http-verb>.ts`
- API に変更を加える場合は、必ず OpenAPI スキーマ (存在する場合) と `README.md` ファイルを適宜更新してください。
