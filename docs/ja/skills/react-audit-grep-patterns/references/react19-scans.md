# React 19 監査 - 完全なスキャン コマンド

この順序で実行します。各セクションは、react19-auditor のフェーズにマップされます。

---

## フェーズ 1 - API の削除 (重大な問題 - 修正が必要)「」バッシュ
#1. ReactDOM.render - 削除されました
grep -rn "ReactDOM\.render\s*(" \
  src/ --include="*.js" --include="*.jsx" 2>/dev/null

# 2. ReactDOM.水和物 - 削除されました
grep -rn "ReactDOM\.水和物\s*(" \
  src/ --include="*.js" --include="*.jsx" 2>/dev/null

# 3. unmountComponentAtNode - 削除されました
grep -rn "unmountComponentAtNode" \
  src/ --include="*.js" --include="*.jsx" 2>/dev/null

#4. findDOMNode - 削除されました
grep -rn "findDOMNode\|ReactDOM\.findDOMNode" \
  src/ --include="*.js" --include="*.jsx" 2>/dev/null

#5.createFactory - 削除されました
grep -rn "createFactory\|React\.createFactory" \
  src/ --include="*.js" --include="*.jsx" 2>/dev/null

# 6.react-dom/test-utils インポート - ほとんどのエクスポートが削除されました
grep -rn "from 'react-dom/test-utils'\|from \"react-dom/test-utils\"\|require.*react-dom/test-utils" \
  src/ --include="*.js" --include="*.jsx" 2>/dev/null

# 7. レガシーコンテキスト API - 削除
grep -rn "contextTypes\|childContextTypes\|getChildContext" \
  src/ --include="*.js" --include="*.jsx" | grep -v "\.test\." 2>/dev/null

#8. 文字列参照 - 削除されました
grep -rn "this\.refs\." \
  src/ --include="*.js" --include="*.jsx" | grep -v "\.test\." 2>/dev/null
「」---

## フェーズ 2 - 非推奨の API (移行する必要があります)「」バッシュ
# 9. forwardRef - 非推奨になりました (ref は直接 prop になりました)
grep -rn "forwardRef\|React\.forwardRef" \
  src/ --include="*.js" --include="*.jsx" | grep -v "\.test\." 2>/dev/null

# 10. 関数コンポーネントのdefaultProps - 関数コンポーネントでは削除されました
grep -rn "\.defaultProps\s*=" \
  src/ --include="*.js" --include="*.jsx" | grep -v "\.test\." 2>/dev/null

# 11. 初期値なしの useRef()
grep -rn "useRef()\|useRef( )" \
  src/ --include="*.js" --include="*.jsx" | grep -v "\.test\." 2>/dev/null

# 12. propTypes (ランタイム検証はサイレントに削除されました)
grep -rn "\.propTypes\s*=" \
  src/ --include="*.js" --include="*.jsx" | grep -v "\.test\." |トイレ -l

#13. 反応テストレンダラー - 非推奨
grep -rn "react-test-renderer\|TestRenderer" \
  src/ --include="*.js" --include="*.jsx" | grep -v "\.test\." 2>/dev/null

# 14. 不要な React のデフォルトインポート (新しい JSX 変換)
grep -rn "^「react」から React をインポート" \
  src/ --include="*.js" --include="*.jsx" | grep -v "\.test\." 2>/dev/null
「」---

## フェーズ 3 - ファイル スキャンのテスト「」バッシュ
# 間違った場所からの act()
grep -rn "from 'react-dom/test-utils'" \
  src/ --include="*.test.js" --include="*.test.jsx" \
       --include="*.spec.js" --include="*.spec.jsx" 2>/dev/null

# 使用状況をシミュレート - 削除されました
grep -rn "シミュレート\。" \
  src/ --include="*.test.*" --include="*.spec.*" 2>/dev/null

# テストのreact-test-renderer
grep -rn "from 'react-test-renderer'" \
  src/ --include="*.test.*" --include="*.spec.*" 2>/dev/null

# Spy コール数アサーション (StrictMode デルタ更新が必要な場合があります)
grep -rn "toHaveBeenCalledTimes" \
  src/ --include="*.test.*" --include="*.spec.*" | head -20 2>/dev/null

# console.error 呼び出し回数アサーション (React 19 エラー報告の変更)
grep -rn "console\.error.*toHaveBeenCalledTimes\|toHaveBeenCalledTimes.*console\.error" \
  src/ --include="*.test.*" --include="*.spec.*" 2>/dev/null
「」---

## フェーズ 4 - StrictMode の動作の変更「」バッシュ
# StrictMode の使用法
grep -rn "StrictMode\|React\.StrictMode" \
  src/ --include="*.js" --include="*.jsx" 2>/dev/null

# StrictMode の二重呼び出しの変更によって影響を受ける可能性のある Spy アサーション
grep -rn "toHaveBeenCalledTimes\|\.mock\.calls\.length" \
  src/ --include="*.test.*" --include="*.spec.*" 2>/dev/null
「」---

## 完全な要約スクリプト「」バッシュ
#!/bin/bash
エコー "===============================
echo "React 19 移行監査の概要"
エコー "===============================
エコー「」
echo "削除された API (重要):"
echo " ReactDOM.render: $(grep -rn "ReactDOM\.render\s*(" src/ --include="*.js" --include="*.jsx" | wc -l | tr -d ' ') ヒット"
echo " ReactDOM.水和物: $(grep -rn "ReactDOM\.水和物\s*(" src/ --include="*.js" --include="*.jsx" | wc -l | tr -d ' ') ヒット"
echo " unmountComponentAtNode: $(grep -rn "unmountComponentAtNode" src/ --include="*.js" --include="*.jsx" | wc -l | tr -d ' ') ヒット"
echo " findDOMNode: $(grep -rn "findDOMNode" src/ --include="*.js" --include="*.jsx" | wc -l | tr -d ' ') ヒット"
echo "react-dom/test-utils: $(grep -rn "from 'react-dom/test-utils'" src/ --include="*.js" --include="*.jsx" | wc -l | tr -d ' ') ヒット"
echo " 従来のコンテキスト: $(grep -rn "contextTypes\|childContextTypes\|getChildContext" src/ --include="*.js" --include="*.jsx" | grep -v "\.test\." | wc -l | tr -d ' ') ヒット"
echo " 文字列参照: $(grep -rn "this\.refs\." src/ --include="*.js" --include="*.jsx" | grep -v "\.test\." | wc -l | tr -d ' ') ヒット"
エコー「」
echo "非推奨の API:"
echo " forwardRef: $(grep -rn "forwardRef" src/ --include="*.js" --include="*.jsx" | grep -v "\.test\." | wc -l | tr -d ' ') ヒット"
echo "defaultProps (fn comps): $(grep -rn "\.defaultProps\s*=" src/ --include="*.js" --include="*.jsx" | grep -v "\.test\." | wc -l | tr -d ' ') ヒット"
echo " useRef() 引数なし: $(grep -rn "useRef()" src/ --include="*.js" --include="*.jsx" | grep -v "\.test\." | wc -l | tr -d ' ') ヒット"
エコー「」
エコー「テストファイルの問題:」
echo "react-dom/test-utils: $(grep -rn "from 'react-dom/test-utils'" src/ --include="*.test.*" --include="*.spec.*" | wc -l | tr -d ' ') ヒット"
echo " 使用法をシミュレートします: $(grep -rn "Simulate\." src/ --include="*.test.*" | wc -l | tr -d ' ') ヒット"
「」
