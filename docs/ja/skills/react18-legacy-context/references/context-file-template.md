# コンテキスト ファイル テンプレート

新しいコンテキスト モジュールの標準テンプレート。コピーして名前を入力します。

＃＃ テンプレート```jsx
// src/contexts/[名前]Context.js
「react」から React をインポートします。

// ──── 1. デフォルト値 ──────────────────
// 形状はプロバイダーが `value` として渡すものと一致する必要があります
// コンシューマーがプロバイダーの外部でレンダリングするときに使用されます (エッジ ケース保護)
const デフォルト値 = {
  // 形状を塗りつぶします
};

// ──── 2. コンテキストの作成 ─────────────────
import const [名前]Context = React.createContext(defaultValue);

// ──── 3. 表示名 (React DevTools の場合) ──────────
[名前]Context.displayName = '[名前]コンテキスト';

// ─── 4. オプション: カスタム フック (強く推奨) ------------------------------------------
// クリーンなインポート パスと、プロバイダー外で使用された場合に役立つエラーを提供します
エクスポート関数 use[名前]() {
  const context = React.useContext([名前]コンテキスト);
  if (context === デフォルト値) {
    //defaultValue がセンチネルの場合のみスローします - 実際のデフォルトが意味をなす場合はスキップします
    // throw new Error('use[Name] は [Name]Provider 内で使用する必要があります');
  }
  コンテキストを返します。
}
「」## 入力例 - AuthContext```jsx
// src/contexts/AuthContext.js
「react」から React をインポートします。

const デフォルト値 = {
  ユーザー: null、
  isAuthenticated: false、
  ログイン: () => Promise.resolve(),
  ログアウト: () => {},
};

エクスポート const AuthContext = React.createContext(defaultValue);
AuthContext.displayName = 'AuthContext';

エクスポート関数 useAuth() {
  React.useContext(AuthContext) を返します。
}
「」## 入力例 - ThemeContext```jsx
// src/contexts/ThemeContext.js
「react」から React をインポートします。

const デフォルト値 = {
  テーマ：「光」、
  トグルテーマ: () => {},
};

エクスポート const ThemeContext = React.createContext(defaultValue);
ThemeContext.displayName = 'テーマコンテキスト';

エクスポート関数 useTheme() {
  React.useContext(ThemeContext) を返します。
}
「」## コンテキスト ファイルを置く場所「」
ソース/
  contexts/ ← 推奨: 専用フォルダー
    AuthContext.js
    テーマコンテキスト.js
「」代替可能な場所:「」
src/context/ ← 単数形でも大丈夫
src/store/contexts/ ← 状態管理と同じ場所にある場合
「」コンテキスト ファイルをコンポーネント フォルダー内に置かないでください。コンテキストは横断的なものであり、1 つのコンポーネントによって所有されるべきではありません。

## アプリ内のプロバイダーの配置

コンテキスト プロバイダーは、アクセスが必要なコンポーネントをラップします。ツリーのできるだけ低い位置に配置します。必ずしも根元に配置する必要はありません。```jsx
// App.js
import { ThemeProvider } from './ThemeProvider';
import { AuthProvider } から './AuthProvider';

関数 App() {
  戻る (
    // 認証はすべてをラップします - どこでもログイン状態が必要です
    <認証プロバイダ>
      {/* テーマは UI シェルのみをラップします - 純粋なデータ プロバイダでは必要ありません */}
      <テーマプロバイダー>
        <ルーター>
          <AppShell />
        </ルーター>
      </テーマプロバイダ>
    </認証プロバイダ>
  );
}
「」
