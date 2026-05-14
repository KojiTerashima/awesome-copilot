---
name: react19-source-patterns
description: 'Reference for React 19 source-file migration patterns, including API changes, ref handling, and context updates.'
---
# React 19 のソース移行パターン

React 19 に必要なすべてのソースファイルの移行に関するリファレンス。

## クイックリファレンステーブル

|パターン |アクション |参考資料 |
|---|---|---|
| `ReactDOM.render(...)` | → `createRoot().render()` | References/api-migrations.md を参照してください。
| `ReactDOM.hydrate(...)` | → `hydrateRoot(...)` | References/api-migrations.md を参照してください。
| `unmountComponentAtNode` | →`root.unmount()` |インライン修正 |
| `ReactDOM.findDOMNode` | → 直接参照 |インライン修正 |
| `forwardRef(...)` ラッパー | → 直接プロップとして参照 | References/api-migrations.md を参照してください。
| `Component.defaultProps = {}` | → ES6 デフォルトパラメータ | References/api-migrations.md を参照してください。
| `useRef()` 引数なし | →`useRef(null)` |インライン修正を追加 `null` |
|レガシーコンテキスト | →`createContext` | [→ api-migrations.md#legacy-context](references/api-migrations.md#legacy-context) |
|文字列参照 `this.refs.x` | →`createRef()` | [→ api-migrations.md#string-refs](references/api-migrations.md#string-refs) |
| `import React from 'react'` (未使用) |削除 |ファイル内で `React.` が使用されていない場合のみ |

## PropTypes ルール

`.propTypes` の割り当てを**削除しないでください**。 `prop-types` パッケージは引き続きスタンドアロンのバリデーターとして機能します。 React 19 では、React パッケージから組み込みのランタイム チェックが削除されるだけで、パッケージ自体は有効なままです。

このコメントを `.propTypes` ブロックの上に追加します。```jsx
// NOTE: React 19 no longer runs propTypes validation at runtime.
// PropTypes kept for documentation and IDE tooling only.
```## Read the Reference

各移行の完全な前後のコードについては、**`references/api-migrations.md`** を参照してください。これには、`forwardRef` と `useImperativeHandle`、`defaultProps` の null と未定義の動作、レガシー コンテキスト プロバイダー/コンシューマーのファイル間の移行などのエッジ ケースを含む完全なパターンが含まれています。