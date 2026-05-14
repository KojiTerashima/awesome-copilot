---
name: react18-string-refs
description: 'Provides exact migration patterns for React string refs (ref="name" + this.refs.name) to React.createRef() in class components. Use this skill whenever migrating string ref usage - including single element refs, multiple refs in a component, refs in lists, callback refs, and refs passed to child components. Always use this skill before writing any ref migration code - the multiple-refs-in-list pattern is particularly tricky and this skill prevents the most common mistakes. Use it for React 18.3.1 migration (string refs warn) and React 19 migration (string refs removed).'
---
# React 18 文字列参照の移行

文字列参照 (`ref="myInput"` + `this.refs.myInput`) は React 16.3 で非推奨になり、React 18.3.1 で警告され、**React 19 で削除されました**。

## クイックパターンマップ

|パターン |参考資料 |
|---|---|
| DOM 要素の単一参照 | [→ pattern.md#single-ref](references/patterns.md#single-ref) |
| 1 つのコンポーネント内の複数の参照 | [→ pattern.md#multiple-refs](references/patterns.md#multiple-refs) |
|リスト内の参照 / 動的参照 | [→ pattern.md#list-refs](references/patterns.md#list-refs) |
|コールバック参照 (代替アプローチ) | [→ pattern.md#callback-refs](references/patterns.md#callback-refs) |
|子コンポーネントに渡される Ref | [→ pattern.md#forwarded-refs](references/patterns.md#forwarded-refs) |

## スキャンコマンド「」バッシュ
# JSX 内のすべての文字列参照割り当てを検索します
grep -rn 'ref="' src/ --include="*.js" --include="*.jsx" | grep -v "\.test\."

# すべての this.refs アクセサを検索します
grep -rn "this\.refs\." src/ --include="*.js" --include="*.jsx" | grep -v "\.test\."
「」両方を一緒に移行する必要があります。各コンポーネントの `ref="name"` アクセスと `this.refs.name` アクセスをペアとして見つけます。

## 移行ルール

すべての文字列参照は `React.createRef()` に移行されます。

1. `refName = React.createRef();` をクラスフィールドとして (またはコンストラクター内に) 追加します。
2. JSX の `ref="refName"` → `ref={this.refName}` を置き換えます
3. `this.refs.refName` → `this.refName.current` をどこでも置き換えます

各ケースの前後の完全な内容については、`references/patterns.md` を参照してください。