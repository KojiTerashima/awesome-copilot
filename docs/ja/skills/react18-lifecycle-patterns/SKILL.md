---
name: react18-lifecycle-patterns
description: 'Provides exact before/after migration patterns for the three unsafe class component lifecycle methods - componentWillMount, componentWillReceiveProps, and componentWillUpdate - targeting React 18.3.1. Use this skill whenever a class component needs its lifecycle methods migrated, when deciding between getDerivedStateFromProps vs componentDidUpdate, when adding getSnapshotBeforeUpdate, or when fixing React 18 UNSAFE_ lifecycle warnings. Always use this skill before writing any lifecycle migration code - do not guess the pattern from memory, the decision trees here prevent the most common migration mistakes.'
---
# React 18 ライフサイクル パターン

3 つの安全でないクラス コンポーネントのライフサイクル メソッドを React 18.3.1 準拠のパターンに移行するためのリファレンス。

## クイック意思決定ガイド

ライフサイクル メソッドを移行する前に、メソッドの動作の **セマンティック カテゴリ**を特定します。間違ったカテゴリ = 間違った移行。以下の表は、正しい参照ファイルへのルートを示しています。

###componentWillMount - 何をするものですか?

|何をするのか |正しい移行 |参考資料 |
|---|---|---|
|初期状態を設定します (`this.setState(...)`) | `constructor` に移動 | [→componentWillMount.md](references/componentWillMount.md#case-a) |
|副作用 (フェッチ、サブスクリプション、DOM) を実行します。 `componentDidMount` に移動 | [→componentWillMount.md](references/componentWillMount.md#case-b) |
| props | から初期状態を取得します。小道具を使用して `constructor` に移動 | [→componentWillMount.md](references/componentWillMount.md#case-c) |

###componentWillReceiveProps - 何をするものですか?

|何をするのか |正しい移行 |参考資料 |
|---|---|---|
|プロパティの変更 (フェッチ、キャンセル) によって引き起こされる非同期の副作用 | `componentDidUpdate` | [→componentWillReceiveProps.md](references/componentWillReceiveProps.md#case-a) |
|新しいプロパティからの純粋な状態の派生 (副作用なし) | `getDerivedStateFromProps` | [→componentWillReceiveProps.md](references/componentWillReceiveProps.md#case-b) |

###componentWillUpdate - 何をするのですか?

|何をするのか |正しい移行 |参考資料 |
|---|---|---|
|更新前に DOM を読み取ります (スクロール、サイズ、位置) | `getSnapshotBeforeUpdate` | [→componentWillUpdate.md](references/componentWillUpdate.md#case-a) |
|更新前にリクエストをキャンセル/実行効果を実行します。 `componentDidUpdate` と前の比較 | [→componentWillUpdate.md](references/componentWillUpdate.md#case-b) |

---

## UNSAFE_ プレフィックス ルール

**`UNSAFE_componentWillMount`、`UNSAFE_componentWillReceiveProps`、または `UNSAFE_componentWillUpdate` を永続的な修正として使用しないでください。**

プレフィックスを付けると React 18.3.1 の警告が抑制されますが、次のことは抑制されません。
- 同時モードの安全性の問題を修正
- React 19 用のコードベースを準備します (これらはプレフィックスの有無にかかわらず削除されます)
- 移行によって解決される根本的なセマンティック上の問題を修正する

UNSAFE_ プレフィックスは、実際の移行スプリントをスケジュールする際の一時的な保留としてのみ適切です。 UNSAFE_ プレフィックスの追加には次のマークを付けます。```jsx
// TODO: React 19 ではこれが削除されます。 React 19 にアップグレードする前に移行してください。
// UNSAFE_ プレフィックスが一時的に追加されました。componentDidMount / getDerivedStateFromProps / などに置き換えます。
「」---

## 参照ファイル

移行するライフサイクル メソッドの完全な参照ファイルをお読みください。

- **`references/componentWillMount.md`** - 完全な前後コードを含む 3 つのケース
- **`references/componentWillReceiveProps.md`** - getDerivedStateFromProps トラップ警告、完全な例
- **`references/componentWillUpdate.md`** - getSnapshotBeforeUpdate +componentDidUpdate のペア

移行コードを記述する前に、関連するファイルを読んでください。