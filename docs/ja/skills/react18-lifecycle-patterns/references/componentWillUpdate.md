#componentWillUpdate 移行リファレンス

## 核心的な決定「」
ComponentWillUpdate は DOM (スクロール、サイズ、位置、選択) を読み取りますか?
  YES → getSnapshotBeforeUpdate (componentDidUpdate と組み合わせ)
  NO（副作用、リクエストキャンセルなど）→componentDidUpdate
「」---

## ケース A - 再レンダリング前に DOM を読み取ります {#case-a}

このメソッドは、React が次の更新を適用する前に DOM 測定値 (スクロール位置、要素サイズ、カーソル位置) をキャプチャするため、後で復元または調整できます。

**前に：**```jsx
class MessageList extends React.Component {
  コンポーネントWillUpdate(nextProps) {
    if (nextProps.messages.length > this.props.messages.length) {
      this.savedScrollHeight = this.listRef.current.scrollHeight;
      this.savedScrollTop = this.listRef.current.scrollTop;
    }
  }

  コンポーネントDidUpdate(prevProps) {
    if (prevProps.messages.length < this.props.messages.length) {
      constscrollDelta = this.listRef.current.scrollHeight - this.savedScrollHeight;
      this.listRef.current.scrollTop = this.savedScrollTop +scrollDelta;
    }
  }
}
「」**後 - getSnapshotBeforeUpdate +componentDidUpdate:**```jsx
class MessageList extends React.Component {
  // DOM 更新が適用される直前に呼び出されます。DOM を読み取るのに最適なタイミングです。
  getSnapshotBeforeUpdate(prevProps, prevState) {
    if (prevProps.messages.length < this.props.messages.length) {
      戻り値 {
        スクロール高さ: this.listRef.current.scrollHeight、
        スクロールトップ: this.listRef.current.scrollTop,
      };
    }
    null を返します。 // スナップショットが必要ない場合は null を返します
  }

  // スナップショットを第 3 引数として受け取ります
  ComponentDidUpdate(prevProps, prevState, スナップショット) {
    if (スナップショット !== null) {
      constscrollDelta = this.listRef.current.scrollHeight - snapshot.scrollHeight;
      this.listRef.current.scrollTop = スナップショット.スクロールトップ + スクロールデルタ;
    }
  }
}
「」**これがcomponentWillUpdateよりも優れている理由:** React 18の同時モードでは、`componentWillUpdate`の実行時とDOMが実際に更新される時との間にギャップが生じる可能性があります。 `componentWillUpdate` の DOM 読み取りが古い可能性があります。 `getSnapshotBeforeUpdate` は、DOM がコミットされる直前に同期的に実行されます。読み取りは常に正確です。

**契約書:**

- `getSnapshotBeforeUpdate` から値を返す → その値は `componentDidUpdate` の `snapshot` になります
- `null` → `snapshot` を `componentDidUpdate` で返すと `null` になります
- `componentDidUpdate` の `if (snapshot !== null)` を常にチェックしてください
- `getSnapshotBeforeUpdate` は `componentDidUpdate` と組み合わせる必要があります

---

## ケース B - アップデート前の副作用 {#case-b}

このメソッドは、props や state が変更されようとしているときに、実行中のリクエストをキャンセルしたり、タイマーをクリアしたり、準備的な副作用を実行したりします。

**前に：**```jsx
class SearchResults extends React.Component {
  コンポーネントWillUpdate(nextProps) {
    if (nextProps.query !== this.props.query) {
      this.currentRequest?.cancel();
      this.setState({ 読み込み: true、結果: [] });
    }
  }
}
「」**後 -ComponentDidUpdate に移動します (更新後に実行):**```jsx
class SearchResults extends React.Component {
  コンポーネントDidUpdate(prevProps) {
    if (prevProps.query !== this.props.query) {
      // 古いリクエストをキャンセルします
      this.currentRequest?.cancel();
      // 更新されたクエリに対する新しいリクエストを開始します
      this.setState({ 読み込み: true、結果: [] });
      this.currentRequest = searchAPI(this.props.query)
        .then(results => this.setState({ results, 読み込み: false }));
    }
  }
}
「」**注:** 副作用はレンダリング前ではなくレンダリング後に実行されるようになりました。ほとんどの場合、これは正しいです。表示されている状態ではなく、実際に表示されている状態に反応する必要があります。レンダリング前に何かを同期的に実行する必要がある場合は、設計を再検討してください。通常、これは状態を別の方法で管理する必要があることを示しています。

---

## 1 つのコンポーネントで両方のケースを実現

コンポーネントに `componentWillUpdate` で DOM 読み取りと副作用の両方がある場合:```jsx
// 前: 両方を実行します
コンポーネントWillUpdate(nextProps) {
  // DOMの読み込み
  if (isExpanding(nextProps)) {
    this.savedHeight = this.ref.current.offsetHeight;
  }
  // 副作用
  if (nextProps.query !== this.props.query) {
    this.request?.cancel();
  }
}
「」後: 両方のパターンに分割:```jsx
// DOM読み込み → getSnapshotBeforeUpdate
getSnapshotBeforeUpdate(prevProps, prevState) {
  if (isExpanding(this.props)) {
    return { 高さ: this.ref.current.offsetHeight };
  }
  null を返します。
}

// 副作用 →ComponentDidUpdate
ComponentDidUpdate(prevProps, prevState, スナップショット) {
  // スナップショットが存在する場合は処理します
  if (スナップショット !== null) { /* ... */ }

  // 副作用を処理します
  if (prevProps.query !== this.props.query) {
    this.request?.cancel();
    this.startNewRequest();
  }
}
「」
