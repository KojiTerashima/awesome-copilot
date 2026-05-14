# 複数のレガシーコンテキスト - 移行リファレンス

## 複数のコンテキストの識別

React 16/17 コードベースには、さまざまな関心事に使用されるいくつかのレガシー コンテキストが含まれることがよくあります。「」バッシュ
# childContextTypes で使用される個別のコンテキスト名を検索します
grep -rn "childContextTypes" src/ --include="*.js" --include="*.jsx" | grep -v "\.test\."
# 各ヒットは移行する個別のコンテキストです
「」クラスヘビーのコードベースの一般的なパターン:

- **テーマのコンテキスト** - ダーク/ライト モード、カラー パレット
- **認証コンテキスト** - 現在のユーザー、ログイン/ログアウト機能
- **ルーターコンテキスト** - 現在のルート、ナビゲーション (古い反応ルーターを使用している場合)
- **ストア コンテキスト** - Redux ストア、ディスパッチ (古い接続パターンを使用している場合)
- **ロケール/i18n コンテキスト** - 言語、翻訳機能
- **トースト/通知コンテキスト** - 通知の表示/非表示

---

## 移行順序

コンテキストを一度に 1 つずつ移行します。それぞれは独立した移行です。「」
各レガシー コンテキストについて:
  1. src/contexts/[名前]Context.js を作成します。
  2.プロバイダーを更新する
  3. すべてのコンシューマーを更新する
  4. アプリを実行します - このコンテキストに対する警告がないことを確認します
  5. 次のコンテキストに移動します
「」最初にすべてのプロバイダーを移行し、次にすべてのコンシューマーを移行しないでください。移行すると、アプリが壊れた中間状態のままになります。

---

## 同じプロバイダー内の複数のコンテキスト

一部のアプリは、1 つのプロバイダー コンポーネントで複数のコンテキストを組み合わせていました。```jsx
// 前 - 1 つのプロバイダーが複数のコンテキスト値をエクスポートします。
class AppProvider extends React.Component {
  静的 childContextTypes = {
    テーマ: PropTypes.string、
    ユーザー: PropTypes.object、
    ロケール: PropTypes.string、
    通知: PropTypes.array、
  };

  getChildContext() {
    戻り値 {
      テーマ: this.state.theme、
      ユーザー: this.state.user、
      ロケール: this.state.locale、
      通知: this.state.notifications,
    };
  }
}
「」**移行アプローチ - 個別のコンテキストに分割:**```jsx
// src/contexts/ThemeContext.js
エクスポート const ThemeContext = React.createContext('light');

// src/contexts/AuthContext.js
import const AuthContext = React.createContext({ ユーザー: null、ログイン: () => {}、ログアウト: () => {} });

// src/contexts/LocaleContext.js
エクスポート const LocaleContext = React.createContext('en');

// src/contexts/NotificationContext.js
エクスポート const NoticeContext = React.createContext([]);
「」

```jsx
// AppProvider.js - 複数のプロバイダーをラップするようになりました
import { ThemeContext } から './contexts/ThemeContext';
import { AuthContext } から './contexts/AuthContext';
import { LocaleContext } から './contexts/LocaleContext';
import { NoticeContext } から './contexts/NotificationContext';

class AppProvider extends React.Component {
  render() {
    const {テーマ、ユーザー、ロケール、通知} = this.state;
    戻る (
      <ThemeContext.Provider 値={テーマ}>
        <AuthContext.Provider 値={{ ユーザー、ログイン: this.login、ログアウト: this.logout }}>
          <LocaleContext.Provider 値={locale}>
            <NotificationContext.Provider 値={通知}>
              {this.props.children}
            </NotificationContext.Provider>
          </LocaleContext.Provider>
        </AuthContext.Provider>
      </ThemeContext.Provider>
    );
  }
}
「」---

## 複数のコンテキストを持つコンシューマ (クラス コンポーネント)

クラス コンポーネントは `static contextType` を 1 つだけ使用できます。複数の場合は、`Consumer` レンダリング プロパティを使用するか、関数コンポーネントに変換します。

### オプション A - 小道具のレンダリング (クラス コンポーネントとして保持)```jsx
import { ThemeContext } から '../contexts/ThemeContext';
import { AuthContext } から '../contexts/AuthContext';

class UserPanel extends React.Component {
  render() {
    戻る (
      <ThemeContext.Consumer>
        {(テーマ) => (
          <AuthContext.Consumer>
            {({ ユーザー, ログアウト }) => (
              <div className={`panel panel-${theme}`}>
                <span>{user?.name}</span>
                <button onClick={logout}>サインアウト</button>
              </div>
            )}
          </AuthContext.Consumer>
        )}
      </ThemeContext.Consumer>
    );
  }
}
「」### オプション B - 関数コンポーネントに変換 (推奨)```jsx
import { useContext } から 'react';
import { ThemeContext } から '../contexts/ThemeContext';
import { AuthContext } から '../contexts/AuthContext';

関数 UserPanel() {
  const テーマ = useContext(ThemeContext);
  const { ユーザー、ログアウト } = useContext(AuthContext);

  戻る (
    <div className={`panel panel-${theme}`}>
      <span>{user?.name}</span>
      <button onClick={logout}>サインアウト</button>
    </div>
  );
}
「」関数コンポーネントへの変換がこの移行スプリントの範囲外である場合は、オプション A を使用します。クラス コンポーネントが単純 (ほとんどがレンダリングのみ) の場合は、オプション B を少し書き直す価値があります。

---

## コンテキスト ファイルの命名規則

コードベース全体で一貫した命名を使用します。「」
ソース/
  コンテキスト/
    ThemeContext.js → エクスポート: ThemeContext、ThemeProvider (オプション)
    AuthContext.js → エクスポート: AuthContext、AuthProvider (オプション)
    LocaleContext.js → エクスポート: LocaleContext
「」各ファイルはコンテキスト オブジェクトをエクスポートします。プロバイダーは元のファイルに留まり、コンテキストをインポートするだけで済みます。

---

## すべてのコンテキストが移行された後の検証「」バッシュ
# 従来のコンテキスト パターンの場合はゼロヒットを返す必要があります
エコー "=== childContextTypes ===
grep -rn "childContextTypes" src/ --include="*.js" --include="*.jsx" | grep -v "\.test\." |トイレ -l

echo "=== contextTypes (レガシー) ===
grep -rn "^\s*static contextTypes\s*=\|contextTypes\.propTypes" src/ --include="*.js" | grep -v "\.test\." |トイレ -l

エコー "=== getChildContext ===
grep -rn "getChildContext" src/ --include="*.js" --include="*.jsx" | grep -v "\.test\." |トイレ -l

echo "3 つすべて 0 でなければなりません"
「」注: `static contextType` (単数形) は MODERN API です - それは正しいです。 `contextTypes` (複数形) のみがレガシーです。