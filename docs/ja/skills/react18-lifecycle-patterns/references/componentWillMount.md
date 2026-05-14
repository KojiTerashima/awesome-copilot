#componentWillMount 移行リファレンス

## ケース A - 状態を初期化します {#case-a}

このメソッドは、非同期操作に依存しない静的値または計算値を使用して `this.setState()` のみを呼び出します。

**前に：**```jsx
class UserList extends React.Component {
  コンポーネントウィルマウント() {
    this.setState({ 項目: []、読み込み: false、ページ: 1 });
  }
  render() { ... }
}
「」**後 - コンストラクターに移動:**```jsx
class UserList extends React.Component {
  コンストラクター(小道具) {
    スーパー(小道具);
    this.state = { 項目: []、読み込み: false、ページ: 1 };
  }
  render() { ... }
}
「」**コンストラクターが既に存在する場合**、状態をマージします。```jsx
class UserList extends React.Component {
  コンストラクター(小道具) {
    スーパー(小道具);
    // 既存の状態は、componentWillMount 状態とマージされます。
    this.state = {
      ...this.existingState, // すでにここにあるものは何でも
      アイテム: []、
      ロード: false、
      ページ: 1、
    };
  }
}
「」---

## ケース B - 副作用が発生する {#case-b}

このメソッドは、データのフェッチ、サブスクリプションのセットアップ、外部 API との対話、または DOM の操作を行います。

**前に：**```jsx
class UserDashboard extends React.Component {
  コンポーネントウィルマウント() {
    this.subscription = this.props.eventBus.subscribe(this.handleEvent);
    フェッチ(`/api/users/${this.props.userId}`)
      .then(r => r.json())
      .then(user => this.setState({ user,loading: false }));
    this.setState({読み込み中: true });
  }
}
「」**後 - コンポーネントDidMountに移動:**```jsx
class UserDashboard extends React.Component {
  コンストラクター(小道具) {
    スーパー(小道具);
    this.state = { 読み込み中: true、ユーザー: null }; // ここでの初期状態
  }

  コンポーネントDidMount() {
    // すべての副作用はここに移動します - 最初のレンダリング後に実行されます
    this.subscription = this.props.eventBus.subscribe(this.handleEvent);
    フェッチ(`/api/users/${this.props.userId}`)
      .then(r => r.json())
      .then(user => this.setState({ user,loading: false }));
  }

  コンポーネントウィルアンマウント() {
    // サブスクリプションとクリーンアップを常に組み合わせます
    this.subscription?.unsubscribe();
  }
}
「」**これが安全な理由:** React 18 同時モードでは、マウントする前に `componentWillMount` を複数回呼び出すことができます。内部の副作用は複数回発生する可能性があります。 `componentDidMount` は、マウント後に 1 回だけ起動することが保証されています。

---

## ケース C - 小道具から初期状態を導出する {#case-c}

このメソッドは `this.props` を読み取り、初期状態値を計算します。

**前に：**```jsx
class PriceDisplay extends React.Component {
  コンポーネントウィルマウント() {
    this.setState({
      フォーマット済み価格: `$${this.props.price.toFixed(2)}`、
      isDiscount: this.props.price < this.props.originalPrice,
    });
  }
}
「」**後 - 小道具を備えたコンストラクター:**```jsx
class PriceDisplay extends React.Component {
  コンストラクター(小道具) {
    スーパー(小道具);
    this.state = {
      フォーマット済み価格: `$${props.price.toFixed(2)}`、
      isDiscount: props.price < props.originalPrice、
    };
  }
}
「」**注意:** プロパティが後で変更されたときにこの初期状態を更新する必要がある場合、それは `getDerivedStateFromProps` のケースです - `componentWillReceiveProps.md` ケース B を参照してください。

---

## 1 つのメソッドで複数のパターンを使用

単一の `componentWillMount` が状態の初期化と副作用の両方を行う場合:```jsx
// 混合 - 状態の初期化 + フェッチ
コンポーネントウィルマウント() {
  this.setState({ 読み込み: true、項目: [] });              // ケースA
  fetch('/api/items').then(r => r.json()) // ケース B
    .then(items => this.setState({ items, 読み込み中: false }));
}
「」それらを分割します。```jsx
コンストラクター(小道具) {
  スーパー(小道具);
  this.state = { 読み込み中: true、項目: [] }; // ケースA → コンストラクター
}

コンポーネントDidMount() {
  fetch('/api/items').then(r => r.json()) // ケース B →ComponentDidMount
    .then(items => this.setState({ items, 読み込み中: false }));
}
「」
