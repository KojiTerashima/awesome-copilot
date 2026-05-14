---
name: react18-batching-patterns
description: 'Provides exact patterns for diagnosing and fixing automatic batching regressions in React 18 class components. Use this skill whenever a class component has multiple setState calls in an async method, inside setTimeout, inside a Promise .then() or .catch(), or in a native event handler. Use it before writing any flushSync call - the decision tree here prevents unnecessary flushSync overuse. Also use this skill when fixing test failures caused by intermediate state assertions that break after React 18 upgrade.'
---
# React 18 の自動バッチ処理パターン

クラスコンポーネントのコードベースに対する React 18 の最も危険なサイレント破壊的変更を診断して修正するためのリファレンス。

## 核となる変化

| setState の場所 |反応17 |反応18 |
|---|---|---|
|反応イベントハンドラ |バッチ処理 |バッチ処理 (同じ) |
|セットタイムアウト | **即時再レンダリング** | **バッチ処理** |
| Promise .then() / .catch() | **即時再レンダリング** | **バッチ処理** |
|非同期/待機 | **即時再レンダリング** | **バッチ処理** |
|ネイティブ addEventListener コールバック | **即時再レンダリング** | **バッチ処理** |

**バッチ** とは、その実行コンテキスト内のすべての setState 呼び出しが、最後に 1 回の再レンダリングでまとめてフラッシュされることを意味します。中間レンダリングは発生しません。

## クイック診断

すべての非同期クラス メソッドを読み取ります。質問: `await` の後に `this.state` を読み取って決定を行うコードはありますか?「」
コードは await? の後に this.state を読み取ります。
  YES → カテゴリ A (サイレント状態読み取りバグ)
  いいえ、しかし中間レンダリングはユーザーに表示される必要がありますか?
    はい → カテゴリ C (flushSync が必要)
    NO → カテゴリ B (リファクタリング、flushSync なし)
「」各カテゴリの完全なパターンについては、以下をお読みください。
- **`references/batching-categories.md`** - カテゴリ A、B、C と完全な前後のコード
- **`references/flushSync-guide.md`** - flashSync を使用する場合と使用しない場合、インポート構文

## flashSync ルール

**`flushSync` は慎重に使用してください。** React 18 の同時スケジューラをバイパスして、同期再レンダリングを強制します。これを使いすぎると、React 18 のパフォーマンス上の利点が無効になります。

`flushSync` は、次の場合にのみ使用します。
- 非同期操作を開始する前に、ユーザーは中間の UI 状態を確認する必要があります。
- フェッチを開始する前に、スピナー/ローディング状態をレンダリングする必要があります
- 連続した UI ステップには明確に表示される状態があります (進行状況ウィザード、複数ステップのフロー)

ほとんどの場合、修正は **リファクタリング** です。つまり、`await` の後の `this.state` を読み取らないようにコードを再構築します。カテゴリごとの正しいアプローチについては、`references/batching-categories.md` をお読みください。