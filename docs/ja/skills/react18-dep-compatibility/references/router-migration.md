# React Router v5 → v6 - スコープ評価

## これが別のスプリントである理由

React Router v5 → v6 は API を完全に書き直しました。個々のパターンに影響するほとんどの React 18 アップグレード手順とは異なり、ルーターの移行は以下に影響します。

- すべての `<Route>` コンポーネント
- すべての `<Switch>` (`<Routes>` に置き換えられます)
- すべての `useHistory()` (`useNavigate()` に置き換えられます)
- すべての `useRouteMatch()` (`useMatch()` に置き換えられます)
- すべての `<Redirect>` (`<Navigate>` に置き換えられます)
- ネストされたルート定義 (まったく新しいモデル)
- ルートパラメータへのアクセス
- クエリ文字列の処理

React 18 アップグレード スプリントの一部としてこれを試みると、移行の範囲が大幅に拡大します。

## 推奨されるアプローチ

### オプション A - ルーターの移行を延期する (推奨)

React 18 のアップグレード中は、`react-router-dom@5.3.4` を `--legacy-peer-deps` とともに使用します。これは、レガシー ルートでの React 18 との互換性のために、react-router チームによってサポートされる回避策として明示的に文書化されています。「」バッシュ
# React 18 dep 外科医では:
npm install reverse-router-dom@5.3.4 --legacy-peer-deps
「」package.json 内のドキュメント:```json
"_legacyPeerDepsReason": {
  "react-router-dom@5.3.4": "ルーター v5→v6 の移行は別のスプリントに延期されました。React 18 ピア デップの不一致のみ - レガシー ルートで API の非互換性はありません。"
}
「」次に、React 18 アップグレードが安定した後、独自のスプリントとして v5 → v6 移行をスケジュールします。

### オプション B - React 18 Sprint の一部としてルーターを移行する

次の場合にのみこれを選択してください。

- アプリのルーティングは最小限です (ルート数 10 未満、ネストされたルートなし、複雑なナビゲーション ロジックなし)
- チームには帯域幅があり、スプリント タイムラインがそれを可能にします

### 範囲評価スキャン

決定する前に、これを実行してルーターの移行範囲を理解します。「」バッシュ
echo "=== ルート定義 ===
grep -rn "<ルート\|<スイッチ\|<リダイレクト" src/ --include="*.js" --include="*.jsx" | grep -v "\.test\." |トイレ -l

echo "=== useHistory 呼び出し ===
grep -rn "useHistory()" src/ --include="*.js" --include="*.jsx" | grep -v "\.test\." |トイレ -l

echo "=== useRouteMatch 呼び出し ===
grep -rn "useRouteMatch()" src/ --include="*.js" --include="*.jsx" | grep -v "\.test\." |トイレ -l

echo "=== withRouter HOC ===
grep -rn "withRouter" src/ --include="*.js" --include="*.jsx" | grep -v "\.test\." |トイレ -l

エコー "=== 履歴.push / 履歴.replace ===
grep -rn "history\.push\|history\.replace\|history\.go" src/ --include="*.js" --include="*.jsx" | grep -v "\.test\." |トイレ -l
「」**意思決定ガイド:**

- 合計ヒット数 < 30 → このスプリントではルーターの移行が可能です
- 合計ヒット数 30 ～ 100 → 延期することを強くお勧めします
- 合計ヒット数 > 100 → 延期する必要があります - 別のスプリントが必要

## v5 → v6 API 変更の概要

| v5 | v6 |メモ |
|---|---|---|
| `<Switch>` | `<Routes>` |直接交換 |
| `<Route path="/" component={C}>` | `<Route path="/" element={<C />}>` |コンポーネントではなく要素プロパティ |
| `<Route exact path="/">` | `<Route path="/">` |正確な値は v6 のデフォルトです。
| `<Redirect to="/new">` | `<Navigate to="/new" />` |コンポーネントの名前変更 |
| `useHistory()` | `useNavigate()` |オブジェクトではなく関数を返します |
| `history.push('/path')` | `navigate('/path')` |直接電話 |
| `history.replace('/path')` | `navigate('/path', { replace: true })` |オプション オブジェクト |
| `useRouteMatch()` | `useMatch()` |異なるリターン形状 |
| `match.params` | `useParams()` |支柱の代わりにフック |
|ネストされたルートがインラインにある |構成内のネストされたルート |レイアウト ルートのコンセプト |
| `withRouter` ホック | `useNavigate` / `useParams` フック | HOC が削除されました |