# 依存関係スキャン - 両方の監査人

依存関係の互換性とピアの競合をスキャンします。 R18 監査と R19 監査の両方で実行します。

---

## 現在のバージョン「」バッシュ
# すべての反応関連パッケージのバージョンをワンショットで
猫パッケージ.json | python3 -c "
SYS、JSONをインポート
d = json.load(sys.stdin)
deps = {**d.get('依存関係',{}), **d.get('devDependency',{})}
キー = ['react', 'react-dom', 'react-router', 'react-router-dom',
        '@testing-library/react'、'@testing-library/jest-dom'、
        '@testing-library/user-event'、'@apollo/client'、'graphql'、
        '@emotion/react'、'@emotion/styled'、'jest'、'enzyme'、
        'react-redux'、'@reduxjs/toolkit'、'prop-types']
キーの k の場合:
    k が深さの場合:
        print(f'{k}: {deps[k]}')
" 2>/dev/null
「」---

## ピア依存関係の競合「」バッシュ
# すべてのピア dep 警告 (移行が完了する前に 0 である必要があります)
npm ls 2>&1 | grep -E "警告|エラー|ピア|無効|未確認"

# ピアエラーの数
npm ls 2>&1 | grep -E "警告|エラー|ピア|無効|未確認" |トイレ -l

# 特定のパッケージのピア配備要件
npm info @testing-library/reactpeerDependency 2>/dev/null
npm info @apollo/clientpeerDependency 2>/dev/null
npm info @emotion/reactpeerDependency 2>/dev/null
npm info 反応ルーターダムピア依存関係 2>/dev/null
「」---

## 酵素検出 (R18 ブロッカー)「」バッシュ
# package.json内
猫パッケージ.json | python3 -c "
SYS、JSONをインポート
d = json.load(sys.stdin)
deps = {**d.get('依存関係',{}), **d.get('devDependency',{})}
酵素 = {k: v for k, v in deps.items() if 'enzyme' in k. lower()}
酵素の場合:
    print('BLOCKER - 酵素が見つかりました:', 酵素)
それ以外の場合:
    print('酵素なし - OK')
" 2>/dev/null

# 酵素アダプターファイル
見つけてください。 -name "酵素アダプター*" -not -path "*/node_modules/*" 2>/dev/null
「」---

## Reactルーターのバージョン確認「」バッシュ
ROUTER=$(node -e "console.log(require('./node_modules/react-router-dom/package.json').version)" 2>/dev/null)
echo "react-router-dom バージョン: $ROUTER"

# v5 の場合 - 評価用のフラグ
if [[ $ROUTER == 5* ]];それから
  echo "警告: 反応ルーター v5 が見つかりました - アップグレードの前にスコープの評価が必要です"
  echo "ルーター移行スコープ スキャンを実行します:"
  echo " ルート: $(grep -rn "<Route\|<Switch\|<Redirect" src/ --include="*.js" --include="*.jsx" | grep -v "\.test\." | wc -l) ヒット"
  echo " useHistory: $(grep -rn "useHistory()" src/ --include="*.js" --include="*.jsx" | grep -v "\.test\." | wc -l) ヒット"
フィ
「」---

## ロックファイルの一貫性「」バッシュ
# ロックファイルが package.json と同期していることを確認します
npm ls --length=0 2>&1 |頭 -20

# 重複した反応インストールをチェックします (フックエラーが発生する可能性があります)
find node_modules -name "package.json" -path "*/react/package.json" 2>/dev/null \
  | grep -v "ノードモジュール/ノードモジュール" \
  | xargs grep '"バージョン"' |並べ替え -u
「」
