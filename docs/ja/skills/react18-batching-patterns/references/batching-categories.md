# バッチ処理カテゴリ - パターンの前/後

## カテゴリ A - this.state 待機後の読み取り (サイレント バグ) {#category-a}

このメソッドは、`await` の後に `this.state` を読み取り、条件付きの決定を行います。 React 18 では、中間の setState はまだフラッシュされていません。`this.state` はまだ更新前の値を保持しています。

**以前 (React 18 で壊れた):**```jsx
非同期 handleLoadClick() {
  this.setState({読み込み中: true });       // バッチ処理 - まだフラッシュされていません
  const data = await fetchData();
  if (this.state.loading) { // ← まだ FALSE (古い値)
    this.setState({ データ、読み込み: false });  // ← 呼び出されることはありません
  }
}
「」**後 - this.state 読み取り全体を削除します:**```jsx
非同期 handleLoadClick() {
  this.setState({読み込み中: true });
  {を試してください
    const data = await fetchData();
    this.setState({ データ、読み込み: false }); // 常に呼び出されます - 条件は必要ありません
  } キャッチ (エラー) {
    this.setState({ エラー: エラー、読み込み: false });
  }
}
「」**パターン:** `this.state` の条件がその時点で常に true になる場合 (true に設定しただけです)、条件を削除します。 `await` の前に呼び出した setState は最終的にフラッシュされます。それを確認する必要はありません。

---

## カテゴリ A バリアント - 複数ステップの条件付きチェーン```jsx
// 前 (壊れた):
非同期初期化() {
  this.setState({ ステップ: '認証' });
  const トークン = await 認証();
  if (this.state.step === 'auth') { // ← 間違っています: まだ初期値です
    this.setState({ ステップ: '読み込み中', トークン });
    const data = awaitloadData(token);
    if (this.state.step === 'loading') { // ← また間違っています
      this.setState({ ステップ: '準備完了', データ });
    }
  }
}
「」

```jsx
// 後 - this.state ではなくローカル変数を使用してフローを追跡します。
非同期初期化() {
  this.setState({ ステップ: '認証' });
  {を試してください
    const トークン = await 認証();
    this.setState({ ステップ: '読み込み中', トークン });
    const data = awaitloadData(token);
    this.setState({ ステップ: '準備完了', データ });
  } キャッチ (エラー) {
    this.setState({ ステップ: 'エラー', エラー: エラー });
  }
}
「」---

## カテゴリ B - 独立した setState 呼び出し (リファクタリング、flushSync なし) {#category-b}

Promise チェーン内の複数の setState 呼び出し。順序は重要ですが、中間状態の読み取りは発生しません。呼び出しを再構築する必要があるだけです。

**前に：**```jsx
handleSubmit() {
  this.setState({ 送信中: true });
  submitForm(this.state.formData)
    .then(結果 => {
      this.setState({ 結果 });
      this.setState({ 送信中: false });  // .then() 内の 2 つの setState
    });
}
「」**後 - setState 呼び出しを統合します:**```jsx
非同期ハンドルSubmit() {
  this.setState({ 送信中: true、結果: null、エラー: null });
  {を試してください
    const result = await submitForm(this.state.formData);
    this.setState({ 結果、送信中: false });
  } キャッチ (エラー) {
    this.setState({ エラー: エラー、送信中: false });
  }
}
「」ルール: 同じ非同期コンテキスト内の複数の `setState` 呼び出しは、React 18 ですでにバッチ処理されています。より少ない呼び出しに統合する方がクリーンですが、厳密に必要というわけではありません。

---

## カテゴリ C - 中間レンダリングが表示される必要がある (flushSync) {#category-c}

ユーザーは、非同期操作を開始する前に、中間の UI 状態 (スピナーの読み込み、進行ステップ) を確認する必要があります。これは、`flushSync` が正しい答えとなる唯一のケースです。

**診断の質問:** 「フェッチが返されるまでローディング スピナーが表示されなかった場合、UX は間違っていますか?」

- はい → `flushSync`
- いいえ → リファクタリング (カテゴリー A または B)

**前:**```jsx
非同期 processOrder() {
  this.setState({ ステータス: '検証中' });   // ユーザーはこれを参照する必要があります
  await validateOrder(this.props.order);
  this.setState({ ステータス: '充電中' });     // ユーザーはこれを参照する必要があります
  ChargeCard(this.props.card)を待ちます;
  this.setState({ ステータス: '完了' });
}
「」**後 - 必要な中間レンダリングごとに flashSync:**```jsx
'react-dom' から { flashSync } をインポートします。

非同期 processOrder() {
  flashSync(() => {
    this.setState({ ステータス: '検証中' });  // すぐにレンダリングされます
  });
  await validateOrder(this.props.order);

  flashSync(() => {
    this.setState({ ステータス: '充電中' });    // すぐにレンダリングされます
  });
  ChargeCard(this.props.card)を待ちます;

  this.setState({ ステータス: '完了' });      // 最後 - flashSync は必要ありません
}
「」**シンプルなローディング スピナー ケース** (最も一般的):```jsx
'react-dom' から { flashSync } をインポートします。

非同期ハンドル検索() {
  // ユーザーはフェッチを開始する前にスピナーを確認する必要があります
  lushSync(() => this.setState({読み込み: true }));
  const results = await searchAPI(this.state.query);
  this.setState({ 結果、読み込み: false });
}
「」---

## setTimeout パターン```jsx
// 前 (React 17 - setTimeout による即時再レンダリング):
handleAutoSave() {
  setTimeout(() => {
    this.setState({保存: true });
    // React 17: ここで再レンダリングが発生しました
    saveToServer(this.state.formData).then(() => {
      this.setState({ 保存: false, lastSaved: Date.now() });
    });
  }、2000);
}
「」

```jsx
// 後 (React 18 - setTimeout バッチ内のすべての setState):
handleAutoSave() {
  setTimeout(async () => {
    // 読み込み状態をフェッチ前に表示する必要がある場合 - flashSync
    flashSync(() => this.setState({ 保存: true }));
    await saveToServer(this.state.formData);
    this.setState({ 保存: false, lastSaved: Date.now() });
  }、2000);
}
「」---

## バッチ処理により壊れるテスト パターン```jsx
// 以前 (React 17 - 中間状態が同期的に表示されていました):
it('保存インジケーターを表示', () => {
  render(<AutoSaveForm />);
  fireEvent.change(input, { target: { value: '新しいテキスト' } });
  Expect(screen.getByText('保存中...')).toBeInTheDocument(); // ← 同期チェック
});

// 後 (React 18 - 中間状態には waitFor を使用):
it('保存インジケーターを表示', async () => {
  render(<AutoSaveForm />);
  fireEvent.change(input, { target: { value: '新しいテキスト' } });
  await waitFor(() => Expect(screen.getByText('Saving...')).toBeInTheDocument());
  await waitFor(() => Expect(screen.getByText('Saved')).toBeInTheDocument());
});
「」
