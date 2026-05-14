#componentWillReceiveProps 移行リファレンス

## 核心的な決定「」
ComponentWillReceiveProps は非同期作業や副作用をトリガーしますか?
  はい → コンポーネントDidUpdate
  NO (純粋な状態導出のみ) → getDerivedStateFromProps
「」疑わしい場合は、`componentDidUpdate` を使用してください。いつでも安全です。
`getDerivedStateFromProps` には、ロジックが純粋な同期状態導出以外の場合に誤った選択をさせるトラップ (このファイルの下部を参照) があります。

---

## ケース A - 非同期の副作用 / プロップ変更時のフェッチ {#case-a}

このメソッドは、データのフェッチ、リクエストのキャンセル、外部状態の更新、またはプロパティが変更されたときに非同期操作を実行します。

**前に：**```jsx
class UserProfile extends React.Component {
  コンポーネントWillReceiveProps(nextProps) {
    if (nextProps.userId !== this.props.userId) {
      this.setState({ 読み込み: true、プロファイル: null });
      fetchProfile(nextProps.userId)
        .then(プロファイル => this.setState({ プロファイル、読み込み: false }))
        .catch(err => this.setState({ エラー: エラー、読み込み: false }));
    }
  }
}
「」**後 - コンポーネントDidUpdate:**```jsx
class UserProfile extends React.Component {
  コンポーネントDidUpdate(prevProps) {
    if (prevProps.userId !== this.props.userId) {
      // this.props を使用します (nextProps ではありません - 更新はすでに行われています)
      this.setState({ 読み込み: true、プロファイル: null });
      fetchProfile(this.props.userId)
        .then(プロファイル => this.setState({ プロファイル、読み込み: false }))
        .catch(err => this.setState({ エラー: エラー、読み込み: false }));
    }
  }
}
「」**主な違い:** `componentDidUpdate` は `prevProps` を受け取ります - `this.props.x !== nextProps.x` ではなく `prevProps.x !== this.props.x` を比較します。アップデートはすでに適用されています。

**キャンセル パターン** (非同期にとって重要):```jsx
class UserProfile extends React.Component {
  _requestId = 0;

  コンポーネントDidUpdate(prevProps) {
    if (prevProps.userId !== this.props.userId) {
      const requestId = ++this._requestId;
      this.setState({読み込み中: true });
      fetchProfile(this.props.userId).then(profile => {
        // userId が再び変更された場合は古い応答を無視します
        if (requestId === this._requestId) {
          this.setState({ プロファイル、読み込み中: false });
        }
      });
    }
  }
}
「」---

## ケース B - 小道具からの純粋な状態の導出 {#case-b}

このメソッドは、新しいプロパティから状態値を同期的に導出するだけです。非同期作業、副作用、外部呼び出しはありません。

**前に：**```jsx
class SortedList extends React.Component {
  コンポーネントWillReceiveProps(nextProps) {
    if (nextProps.items !== this.props.items) {
      this.setState({
        sortedItems: [...nextProps.items].sort((a, b) => a.name.localeCompare(b.name)),
      });
    }
  }
}
「」**後 - getDerivedStateFromProps:**```jsx
class SortedList extends React.Component {
  // 変更を検出するには前の prop を追跡する必要があります
  static getDerivedStateFromProps(props, state) {
    if (props.items !== state.prevItems) {
      戻り値 {
        sortedItems: [...props.items].sort((a, b) => a.name.localeCompare(b.name)),
        prevItems: props.items, // ← 比較しているプロップを常に保存します
      };
    }
    null を返します。 // null = 状態変化なし
  }

  コンストラクター(小道具) {
    スーパー(小道具);
    this.state = {
      sortedItems: [...props.items].sort((a, b) => a.name.localeCompare(b.name)),
      prevItems: props.items, // ← コンストラクターでも初期化します
    };
  }
}
「」---

## getDerivedStateFromProps - トラップと警告

### トラップ 1: プロップの変更だけでなく、すべてのレンダリングで発生します。

`componentWillReceiveProps` とは異なり、`getDerivedStateFromProps` は、`setState` 呼び出しを含むすべてのレンダリングの前に呼び出されます。状態に保存されている以前の値と常に比較します。```jsx
// 間違っています - setState トリガーを含むすべてのレンダリングで起動します
static getDerivedStateFromProps(props, state) {
  return {sortedItems:sort(props.items)}; // setState ごとに再ソートします!
}

// 正しい - 項目の参照が変更された場合にのみ更新されます
static getDerivedStateFromProps(props, state) {
  if (props.items !== state.prevItems) {
    return {sortedItems:sort(props.items)、prevItems:props.items};
  }
  null を返します。
}
「」### トラップ 2: `this` にアクセスできません

`getDerivedStateFromProps` は静的メソッドです。 `this.props`、`this.state`、インスタンス メソッドはありません。```jsx
// 間違っています - 静的メソッドにはこれはありません
static getDerivedStateFromProps(props, state) {
  return { 値: this.computeValue(props) }; // 参照エラー
}

// 正しい - props + state の純粋な関数
static getDerivedStateFromProps(props, state) {
  return { 値: computeValue(props) }; // スタンドアロン関数
}
「」### 罠 3: 副作用のために使用しないでください

プロップが変更されたときにフェッチする必要がある場合は、`componentDidUpdate` を使用します。 `getDerivedStateFromProps` は純粋でなければなりません。

### getDerivedStateFromProps が実際には間違ったツールである場合

`getDerivedStateFromProps` で複雑なロジックを実行していることに気付いた場合は、使用するコンポーネントが、代わりに前処理されたデータを prop として受け取る必要があるかどうかを検討してください。このパターンは、一般的な prop-to-state 同期ではなく、狭いユースケースに存在します。