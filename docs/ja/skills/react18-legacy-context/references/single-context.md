# 単一コンテキストの移行 - 前後の完了

## 完全な例: ThemeContext

これは、1 つのプロバイダーと複数のコンシューマーを持つ 1 つのコンテキストという、最も一般的なパターンをカバーしています。

---

### ステップ 1 - 前の状態 (レガシー)

**ThemeProvider.js (プロバイダー):**```jsx
'prop-types' から PropTypes をインポートします。

class ThemeProvider extends React.Component {
  静的 childContextTypes = {
    テーマ: PropTypes.string、
    トグルテーマ: PropTypes.func、
  };

  状態 = { テーマ: 'ライト' };

  toggleTheme = () => {
    this.setState(s => ({ テーマ: s.theme === 'ライト' ? 'ダーク' : 'ライト' }));
  };

  getChildContext() {
    戻り値 {
      テーマ: this.state.theme、
      トグルテーマ: this.toggleTheme、
    };
  }

  render() {
    this.props.children を返します。
  }
}
「」**ThemedButton.js (クラスコンシューマ):**```jsx
'prop-types' から PropTypes をインポートします。

class ThemedButton extends React.Component {
  静的 contextTypes = {
    テーマ: PropTypes.string、
    トグルテーマ: PropTypes.func、
  };

  render() {
    const {テーマ、toggleTheme } = this.context;
    戻る (
      <button className={`btn btn-${theme}`} onClick={toggleTheme}>
        テーマの切り替え
      </ボタン>
    );
  }
}
「」**ThemedHeader.js (関数コンシューマー - 存在する場合):**```jsx
// 関数コンポーネントはレガシーコンテキストをきれいに使用できませんでした
// クラスラッパーを使用するか、プロップをレンダリングする必要がありました
「」---

### ステップ 2 - コンテキスト ファイルの作成

**src/contexts/ThemeContext.js (新しいファイル):**```jsx
「react」から React をインポートします。

// デフォルト値は getChildContext() の形状と一致します。
エクスポート const ThemeContext = React.createContext({
  テーマ：「光」、
  トグルテーマ: () => {},
});

// コンテキストの名前付きエクスポート - プロバイダーとコンシューマーの両方がここからインポートします
「」---

### ステップ 3 - プロバイダーの更新

**ThemeProvider.js (後):**```jsx
「react」から React をインポートします。
import { ThemeContext } から '../contexts/ThemeContext';

class ThemeProvider extends React.Component {
  状態 = { テーマ: 'ライト' };

  toggleTheme = () => {
    this.setState(s => ({ テーマ: s.theme === 'ライト' ? 'ダーク' : 'ライト' }));
  };

  render() {
    // React 19 JSX の略記: <ThemeContext value={...}>
    // React 18: <ThemeContext.Provider value={...}>
    戻る (
      <ThemeContext.Provider
        値={{
          テーマ: this.state.theme、
          トグルテーマ: this.toggleTheme、
        }}
      >
        {this.props.children}
      </ThemeContext.Provider>
    );
  }
}

デフォルトのThemeProviderをエクスポートします。
「」> **React 19 の注意:** React 19 では、`<ThemeContext value={...}>` を直接書くことができます (`.Provider` は不可)。 React 18.3.1 の場合は `<ThemeContext.Provider value={...}>` を使用します。

---

### ステップ 4 - クラス コンシューマを更新する

**ThemedButton.js (後):**```jsx
「react」から React をインポートします。
import { ThemeContext } から '../contexts/ThemeContext';

class ThemedButton extends React.Component {
  // 単数の contextType (contextTypes ではありません)
  静的 contextType = ThemeContext;

  render() {
    const {テーマ、toggleTheme } = this.context;
    戻る (
      <button className={`btn btn-${theme}`} onClick={toggleTheme}>
        テーマの切り替え
      </ボタン>
    );
  }
}

デフォルトのテーマボタンをエクスポートします。
「」**レガシーとの主な違い:**

- `contextTypes` (複数形) ではなく `static contextType` (単数形)
- PropTypes 宣言は必要ありません
- `this.context` は完全な値オブジェクトです (`value` に渡したものは部分的なものではありません)。
- `contextType` を介してクラス コンポーネントごとに 1 つのコンテキストのみ - 複数の場合は `Context.Consumer` レンダー プロップを使用

---

### ステップ 5 - 関数コンシューマを更新する

**ThemedHeader.js (後 - フックを使用して簡単になりました):**```jsx
import { useContext } から 'react';
import { ThemeContext } から '../contexts/ThemeContext';

function ThemedHeader({ タイトル }) {
  const {テーマ} = useContext(ThemeContext);
  <h1 className={`header-${theme}`}>{title}</h1> を返します。
}
「」---

### ステップ 6 - 1 つのクラス コンポーネント内の複数のコンテキスト

クラス コンポーネントが複数のレガシー コンテキストを消費する場合、クラス コンポーネントは複雑になります。クラス コンポーネントには `static contextType` を 1 つだけ含めることができます。複数のコンテキストの場合は、render prop フォームを使用します。```jsx
import { ThemeContext } から '../contexts/ThemeContext';
import { AuthContext } から '../contexts/AuthContext';

class Dashboard extends React.Component {
  render() {
    戻る (
      <ThemeContext.Consumer>
        {({ テーマ }) => (
          <AuthContext.Consumer>
            {({ ユーザー }) => (
              <div className={`dashboard-${theme}`}>
                ようこそ、{user.name}
              </div>
            )}
          </AuthContext.Consumer>
        )}
      </ThemeContext.Consumer>
    );
  }
}
「」または、`useContext` をクリーンに使用するために、クラス コンポーネントを関数コンポーネントに移行することを検討してください。

---

### 検証チェックリスト

1 つのコンテキストを移行した後:「」バッシュ
# プロバイダー - 従来のコンテキストのエクスポートは残りません
grep -n "childContextTypes\|getChildContext" src/ThemeProvider.js

# Consumers - レガシーコンテキストの消費は残りません
grep -rn "contextTypes\s*=" src/ --include="*.js" --include="*.jsx" | grep -v "ThemeContext\|\.test\."

# this.context の使用法 - レガシーではなく contextType から読み取られていることを確認します
grep -rn "この\.context\." src/ --include="*.js" | grep -v "\.test\."
「」それぞれが、移行されたコンテキストに対してゼロのヒットを返す必要があります。