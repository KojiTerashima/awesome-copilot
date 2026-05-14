# React 18.3.1 監査 - 完全なスキャン コマンド

この順序で実行します。各セクションは、react18-auditor のフェーズにマップされます。

---

## フェーズ 0 - コードベース プロファイル「」バッシュ
ソース ファイルの合計数 (テストを除く)
find src/ \( -name "*.js" -o -name "*.jsx" \) \
  | grep -v "\.test\.\|\.spec\.\|__tests__\|node_modules" \
  |トイレ -l

# クラスコンポーネント数
grep -rl "React\.Component を拡張\|コンポーネントを拡張\|PureComponent を拡張" \
  src/ --include="*.js" --include="*.jsx" \
  | grep -v "\.test\." |トイレ -l

# 関数コンポーネントのおおよその数
grep -rl "const [A-Z][a-zA-Z]* = \|function [A-Z][a-zA-Z]*(" \
  src/ --include="*.js" --include="*.jsx" \
  | grep -v "\.test\." |トイレ -l

# 現在の React バージョン
node -e "console.log(require('./node_modules/react/package.json').version)" 2>/dev/null

# StrictMode を使用していますか? (すでに表示されたライフサイクル警告の数に影響します)
grep -rn "StrictMode\|React\.StrictMode" \
  src/ --include="*.js" --include="*.jsx" | grep -v "\.test\."
「」---

## フェーズ 1 - 安全でないライフサイクル手法「」バッシュ
#componentWillMount (UNSAFE_ プレフィックスなし)
grep -rn "コンポーネントウィルマウント\b" \
  src/ --include="*.js" --include="*.jsx" \
  | grep -v "UNSAFE_componentWillMount\|\.test\."

#componentWillReceiveProps (UNSAFE_ プレフィックスなし)
grep -rn "componentWillReceiveProps\b" \
  src/ --include="*.js" --include="*.jsx" \
  | grep -v "UNSAFE_componentWillReceiveProps\|\.test\."

#componentWillUpdate (UNSAFE_ プレフィックスなし)
grep -rn "componentWillUpdate\b" \
  src/ --include="*.js" --include="*.jsx" \
  | grep -v "UNSAFE_componentWillUpdate\|\.test\."

# UNSAFE_ プレフィックスを付けて部分的に移行済みですか? (チームがすでに部分的な作業を行っているかどうかを確認してください)
grep -rn "UNSAFE_コンポーネント" \
  src/ --include="*.js" --include="*.jsx" | grep -v "\.test\."

# 簡単なカウントの概要:
echo "=== ライフサイクルの問題の概要 ===
echo "componentWillMount: $(grep -rn "componentWillMount\b" src/ --include="*.js" --include="*.jsx" | grep -v "UNSAFE_\|\.test\." | wc -l)"
echo "componentWillReceiveProps: $(grep -rn "componentWillReceiveProps\b" src/ --include="*.js" --include="*.jsx" | grep -v "UNSAFE_\|\.test\." | wc -l)"
echo "componentWillUpdate: $(grep -rn "componentWillUpdate\b" src/ --include="*.js" --include="*.jsx" | grep -v "UNSAFE_\|\.test\." | wc -l)"
「」---

## フェーズ 2 - 自動バッチ処理の脆弱性「」バッシュ
# 非同期クラス メソッド (主要なリスク ゾーン)
grep -rn "^\s*async [a-zA-Z]" \
  src/ --include="*.js" --include="*.jsx" | grep -v "\.test\."

# アロー関数の非同期メソッド
grep -rn "=\s*async\s*(" \
  src/ --include="*.js" --include="*.jsx" | grep -v "\.test\."

# .then() コールバック内の setState
grep -rn "\.then\s*(" \
  src/ --include="*.js" --include="*.jsx" -A 3 \
  | grep "setState" | grep -v "\.test\."

# .catch() コールバック内の setState
grep -rn "\.catch\s*(" \
  src/ --include="*.js" --include="*.jsx" -A 3 \
  | grep "setState" | grep -v "\.test\."

# setTimeout 内の setState
grep -rn "setTimeout" \
  src/ --include="*.js" --include="*.jsx" -A 5 \
  | grep "setState" | grep -v "\.test\."

# await に従う this.state 読み取り (最も危険なパターン)
grep -rn "この\.state\." \
  src/ --include="*.js" --include="*.jsx" -B 3 \
  | grep "待つ" | grep -v "\.test\."

# setState を使用したドキュメント/ウィンドウ イベント ハンドラー
grep -rn "addEventListener" \
  src/ --include="*.js" --include="*.jsx" -A 5 \
  | grep "setState" | grep -v "\.test\."
「」---

## フェーズ 3 - レガシー コンテキスト API「」バッシュ
# プロバイダー側
grep -rn "childContextTypes\s*=" \
  src/ --include="*.js" --include="*.jsx" | grep -v "\.test\."

grep -rn "getChildContext\s*(" \
  src/ --include="*.js" --include="*.jsx" | grep -v "\.test\."

# 消費者側
grep -rn "contextTypes\s*=" \
  src/ --include="*.js" --include="*.jsx" | grep -v "\.test\."

# this.context の使用法 (レガシーまたはモダンを示す場合があります - ヒットごとに確認します)
grep -rn "この\.context\." \
  src/ --include="*.js" --include="*.jsx" | grep -v "\.test\."

# 個別のレガシー コンテキストの数 (childContextTypes ブロックの数による)
grep -rn "childContextTypes" \
  src/ --include="*.js" --include="*.jsx" | grep -v "\.test\." |トイレ -l
「」---

## フェーズ 4 - 文字列参照「」バッシュ
# JSX での文字列参照の代入
grep -rn 'ref="[^"]*"' \
  src/ --include="*.js" --include="*.jsx" | grep -v "\.test\."

# 代替引用スタイル
grep -rn "ref='[^']*'" \
  src/ --include="*.js" --include="*.jsx" | grep -v "\.test\."

# this.refs アクセサーの使用法
grep -rn "this\.refs\." \
  src/ --include="*.js" --include="*.jsx" | grep -v "\.test\."
「」---

## フェーズ 5 - findDOMNode「」バッシュ
grep -rn "findDOMNode\|ReactDOM\.findDOMNode" \
  src/ --include="*.js" --include="*.jsx" | grep -v "\.test\."
「」---

## フェーズ 6 - ルート API (ReactDOM.render)「」バッシュ
grep -rn "ReactDOM\.render\s*(" \
  src/ --include="*.js" --include="*.jsx"

grep -rn "ReactDOM\.水和物\s*(" \
  src/ --include="*.js" --include="*.jsx"

grep -rn "unmountComponentAtNode" \
  src/ --include="*.js" --include="*.jsx"
「」---

## フェーズ 7 - イベント委任 (React 16 キャリーオーバー)「」バッシュ
# ドキュメントレベルのイベントリスナー (React 17 委任の変更後、React イベントが失われる可能性があります)
grep -rn "document\.addEventListener\|document\.removeEventListener" \
  src/ --include="*.js" --include="*.jsx" | grep -v "\.test\."

# ウィンドウイベントリスナー
grep -rn "window\.addEventListener" \
  src/ --include="*.js" --include="*.jsx" | grep -v "\.test\."
「」---

## フェーズ 8 - 酵素検出 (ハードブロッカー)「」バッシュ
# package.json 内の酵素
猫パッケージ.json | python3 -c "
SYS、JSONをインポート
d = json.load(sys.stdin)
deps = {**d.get('依存関係',{}), **d.get('devDependency',{})}
enzyme_pkgs = [k. lower() の '酵素' の場合、deps の k の k]
print('酵素パッケージが見つかりました:',enzyme_pkgs if elastic_pkgs else 'NONE')
」

# テストファイル内の酵素インポート
grep -rn "from '酵素'\|require.*enzyme" \
  src/ --include="*.test.*" --include="*.spec.*" 2>/dev/null |トイレ -l
「」---

## 完全な要約スクリプト

詳細なスキャンの前に、簡単な概要を確認するためにこれを実行します。「」バッシュ
#!/bin/bash
エコー "===============================
echo "React 18 移行監査の概要"
エコー "===============================
エコー「」
echo "ライフサイクルメソッド:"
echo "componentWillMount: $(grep -rn "componentWillMount\b" src/ --include="*.js" --include="*.jsx" | grep -v "UNSAFE_\|\.test\." | wc -l | tr -d ' ') ヒット"
echo "componentWillReceiveProps: $(grep -rn "componentWillReceiveProps\b" src/ --include="*.js" --include="*.jsx" | grep -v "UNSAFE_\|\.test\." | wc -l | tr -d ' ') ヒット"
echo "componentWillUpdate: $(grep -rn "componentWillUpdate\b" src/ --include="*.js" --include="*.jsx" | grep -v "UNSAFE_\|\.test\." | wc -l | tr -d ' ') ヒット"
エコー「」
エコー「レガシー API:」
echo " 従来のコンテキスト (プロバイダー): $(grep -rn "childContextTypes" src/ --include="*.js" --include="*.jsx" | grep -v "\.test\." | wc -l | tr -d ' ') ヒット"
echo " 文字列参照 (this.refs): $(grep -rn "this\.refs\." src/ --include="*.js" --include="*.jsx" | grep -v "\.test\." | wc -l | tr -d ' ') ヒット"
echo " findDOMNode: $(grep -rn "findDOMNode" src/ --include="*.js" --include="*.jsx" | grep -v "\.test\." | wc -l | tr -d ' ') ヒット"
echo " ReactDOM.render: $(grep -rn "ReactDOM\.render\s*(" src/ --include="*.js" --include="*.jsx" | wc -l | tr -d ' ') ヒット"
エコー「」
エコー「酵素（ブロッカー）：」
echo "酵素テスト ファイル: $(grep -rl "from 'enzyme'" src/ --include="*.test.*" 2>/dev/null | wc -l | tr -d ' ') files"
エコー「」
echo "非同期バッチ処理のリスク:"
echo " 非同期クラス メソッド: $(grep -rn "^\s*async [a-zA-Z]" src/ --include="*.js" --include="*.jsx" | grep -v "\.test\." | wc -l | tr -d ' ') ヒット"
「」
