# テスト ファイル スキャン - 両方の監査人

特にテスト ファイルの問題をスキャンします。 R18 監査と R19 監査の両方で実行します。

---

## セットアップ ファイル「」バッシュ
# テストセットアップファイルを検索する
find src/ -name "setupTests*" -o -name "jest.setup*" 2>/dev/null
見つけてください。 -name "jest.config.js" -o -name "jest.config.ts" 2>/dev/null | grep -v "ノードモジュール"

# セットアップ ファイルでレガシー パターンを確認する
grep -n "ReactDOM\|react-dom/test-utils\|Enzyme\|configure\|Adapter" \
  src/setupTests.js 2>/dev/null
「」---

## スキャンのインポート「」バッシュ
# テスト内のすべてのreact-dom/test-utilsインポート
grep -rn "from 'react-dom/test-utils'\|require.*react-dom/test-utils" \
  src/ --include="*.test.*" --include="*.spec.*" 2>/dev/null

# 酵素のインポート
grep -rn "from '酵素'\|require.*enzyme" \
  src/ --include="*.test.*" --include="*.spec.*" 2>/dev/null

# 反応テストレンダラー
grep -rn "from 'react-test-renderer'" \
  src/ --include="*.test.*" --include="*.spec.*" 2>/dev/null

# 旧幕の場所
grep -rn "act.*from 'react-dom'" \
  src/ --include="*.test.*" --include="*.spec.*" 2>/dev/null
「」---

## レンダリング パターン スキャン「」バッシュ
# テストでの ReactDOM.render (RTL レンダリングを使用する必要があります)
grep -rn "ReactDOM\.render\s*(" \
  src/ --include="*.test.*" --include="*.spec.*" 2>/dev/null

# 酵素シャロー/マウント
grep -rn "浅い(\|マウント(" \
  src/ --include="*.test.*" --include="*.spec.*" 2>/dev/null

# カスタムレンダリングヘルパー
find src/ -name "test-utils.js" -o -name "renderWithProviders*" \
  -o -name "customRender*" -o -name "render-helpers*" 2>/dev/null
「」---

## アサーション スキャン「」バッシュ
# コール数アサーション (StrictMode に依存)
grep -rn "toHaveBeenCalledTimes" \
  src/ --include="*.test.*" --include="*.spec.*" 2>/dev/null

# console.error アサーション (R19 で変更された React エラー ログ)
grep -rn "コンソール\.エラー" \
  src/ --include="*.test.*" --include="*.spec.*" 2>/dev/null

# 中間状態アサーション (バッチ処理に依存)
grep -rn "fireEvent\|userEvent" \
  src/ --include="*.test.*" --include="*.spec.*" -A 1 \
  | grep "expect\|getBy\|queryBy" | head -20 2>/dev/null
「」---

## 非同期スキャン「」バッシュ
# act() の使用法
grep -rn "\bact(" \
  src/ --include="*.test.*" --include="*.spec.*" 2>/dev/null

# waitFor の使用法 (良い - これらが適切に非同期であることを確認してください)
grep -rn "waitFor\|findBy" \
  src/ --include="*.test.*" --include="*.spec.*" |トイレ -l

# テストでの setTimeout (バッチ処理に依存する可能性があります)
grep -rn "setTimeout\|setInterval" \
  src/ --include="*.test.*" --include="*.spec.*" 2>/dev/null
「」
