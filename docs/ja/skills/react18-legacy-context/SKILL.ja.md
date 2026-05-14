---
name: react18-legacy-context
description: 'Provides the complete migration pattern for React legacy context API (contextTypes, childContextTypes, getChildContext) to the modern createContext API. Use this skill whenever migrating legacy context in class components - this is always a cross-file migration requiring the provider AND all consumers to be updated together. Use it before touching any contextTypes or childContextTypes code, because migrating only the provider without the consumers (or vice versa) will cause a runtime failure. Always read this skill before writing any context migration - the cross-file coordination steps here prevent the most common context migration bugs.'
---
# React 18 レガシーコンテキストの移行

レガシー コンテキスト (`contextTypes`、`childContextTypes`、`getChildContext`) は React 16.3 で非推奨となり、React 18.3.1 で警告します。 **React 19 では削除されました**。

## これは常にファイル間の移行です

一度に 1 つのファイルを処理する他のほとんどの移行とは異なり、コンテキストの移行では次の調整が必要です。
1. コンテキスト オブジェクト (通常は新しいファイル) を作成します。
2. **プロバイダ** コンポーネントを更新します
3. **すべてのコンシューマ** コンポーネントを更新する

コンシューマーが欠けていると、アプリが壊れたままになります。間違ったコンテキストから読み取られるか、`undefined` が取得されます。

## 移行手順 (常にこの順序に従ってください)「」
ステップ 1: プロバイダーを見つける (childContextTypes + getChildContext)
ステップ 2: すべてのコンシューマー (contextTypes) を検索する
ステップ 3: コンテキスト ファイルを作成する
ステップ 4: プロバイダーを更新する
ステップ 5: 各コンシューマーを更新します (クラス コンポーネント → contextType、関数コンポーネント → useContext)
ステップ 6: 確認 - アプリを実行し、従来のコンテキスト警告が残っていないことを確認します。
「」## スキャンコマンド「」バッシュ
# すべてのプロバイダーを検索
grep -rn "childContextTypes\|getChildContext" src/ --include="*.js" --include="*.jsx" | grep -v "\.test\."

# すべての消費者を検索
grep -rn "contextTypes\s*=" src/ --include="*.js" --include="*.jsx" | grep -v "\.test\."

# this.context の使用法を検索します (レガシーまたはモダンである可能性があります - どちらかを確認してください)
grep -rn "この\.context\." src/ --include="*.js" --include="*.jsx" | grep -v "\.test\."
「」## 参照ファイル

- **`references/single-context.md`** - プロバイダー + クラス コンシューマー + 関数コンシューマーによる 1 つのコンテキスト (テーマ、認証など) の移行を完了する
- **`references/multi-context.md`** - 複数のレガシー コンテキスト (ネストされたプロバイダー、異なるコンテキストの複数のコンシューマー) を持つアプリ
- **`references/context-file-template.md`** - 新しいコンテキスト モジュールの標準ファイル構造