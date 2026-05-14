# flashSync ガイド

## インポート```jsx
'react-dom' から { flashSync } をインポートします。
// 'react' からのものではありません - それはreact-domに存在します
「」ファイルが既に `react-dom` からインポートされている場合:```jsx
「react-dom」から ReactDOM をインポートします。
// 名前付きインポートを追加します。
import ReactDOM, { flashSync } から 'react-dom';
「」## 構文```jsx
flashSync(() => {
  this.setState({ ... });
});
// この行以降、再レンダリングは同期的に完了します
「」1 つの flashSync バッチ内の複数の setState 呼び出しが 1 つの同期レンダリングにまとめられます。```jsx
flashSync(() => {
  this.setState({ ステップ: '読み込み中' });
  this.setState({ 進行状況: 0 });
  // これらをまとめてバッチ → 1 回のレンダリング
});
「」## いつ使用するか

✅ 非同期操作が開始される前に、ユーザーが特定の UI 状態を確認する必要がある場合に使用します。```jsx
lushSync(() => this.setState({読み込み: true }));
高価なAsyncOperation()を待ちます;
「」✅ 各ステップが次のステップの前に視覚的に完了する必要がある、複数ステップの進行フローで使用します。```jsx
flashSync(() => this.setState({ ステータス: '検証中' }));
検証を待ちます();
flashSync(() => this.setState({ ステータス: '処理中' }));
プロセスを待ちます();
「」✅ 中間 UI 状態を同期的にアサートする必要があるテストで使用します (可能な場合は避けてください - `waitFor` を推奨します)。

## 使用しない場合

❌ reading-this.state-after-await のバグを「修正」するためにこれを使用しないでください。これはカテゴリ A です (代わりにリファクタリングします)。```jsx
// 間違っています - flashSync ではこれが解決されません
lushSync(() => this.setState({読み込み: true }));
const data = await fetchData();
if (this.state.loading) { ... } // まだ競合状態です
「」❌ 「安全にする」ためにすべての setState にこれを使用しないでください。これは React 18 の同時レンダリングを無効にします。```jsx
// 間違っています - 過剰な flashSync
非同期ハンドルクリック() {
  flashSync(() => this.setState({ クリック: true }));   // 不要
  flashSync(() => this.setState({ 処理: true })); // 不要
  const result = doWork() を待ちます。
  flashSync(() => this.setState({ 結果、完了: true })); // 不要
}
「」❌ 即時状態をトリガーするために `useEffect` または `componentDidMount` 内で使用しないでください。ネストされたレンダリング サイクルが発生します。

## パフォーマンスに関するメモ

`flushSync` は同期レンダリングを強制し、レンダリングが完了するまでブラウザのスレッドをブロックします。遅いデバイスや複雑なコンポーネント ツリーでは、非同期メソッドで `flushSync` を複数回呼び出すと、目に見えるジャンクが発生します。控えめに使用してください。

1 つのメソッドに 2 つを超える `flushSync` 呼び出しを追加している場合は、コンポーネントの状態モデルを再設計する必要があるかどうかを再検討してください。