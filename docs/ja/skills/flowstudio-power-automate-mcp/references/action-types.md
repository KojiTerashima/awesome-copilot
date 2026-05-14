# FlowStudio MCP — アクション タイプのリファレンス

`get_live_flow` によって返されたアクション タイプを認識するためのコンパクトなルックアップ。
これを使用して、既存のフロー定義を**読んで理解**します。

> 完全なコピー＆ペースト構築パターンについては、`flowstudio-power-automate-build` スキルを参照してください。

---

## フロー定義の読み取り方法

すべてのアクションには `"type"`、`"runAfter"`、および `"inputs"` があります。 `runAfter` オブジェクト
依存関係を宣言します: `{"Previous": ["Succeeded"]}`。有効なステータス:
`Succeeded`、`Failed`、`Skipped`、`TimedOut`。

---

## アクション タイプのクイック リファレンス

|タイプ |目的 |検査する主要なフィールド |出力リファレンス |
|---|---|---|---|
| `Compose` |値の保存/変換 | `inputs` (任意の式) | `outputs('Name')` |
| `InitializeVariable` |変数を宣言する | `inputs.variables[].{name, type, value}` | `variables('name')` |
| `SetVariable` |変数を更新する | `inputs.{name, value}` | `variables('name')` |
| `IncrementVariable` |数値変数をインクリメントする | `inputs.{name, value}` | `variables('name')` |
| `AppendToArrayVariable` |配列変数にプッシュする | `inputs.{name, value}` | `variables('name')` |
| `If` |条件分岐 | `expression.and/or`、`actions`、`else.actions` | — |
| `Switch` |多方向分岐 | `expression`、`cases.{case, actions}`、`default` | — |
| `Foreach` |配列をループする | `foreach`、`actions`、`operationOptions` | `item()` / `items('Name')` |
| `Until` |条件 | までループします。 `expression`、`limit.{count, timeout}`、`actions` | — |
| `Wait` |遅延 | `inputs.interval.{count, unit}` | — |
| `Scope` |グループ / トライキャッチ | `actions` (ネストされたアクション マップ) | `result('Name')` |
| `Terminate` |エンドラン | `inputs.{runStatus, runError}` | — |
| `OpenApiConnection` |コネクタ通話 (SP、Outlook、Teams…) | `inputs.host.{apiId, connectionName, operationId}`、`inputs.parameters` | `outputs('Name')?['body/...']` |
| `OpenApiConnectionWebhook` | Webhook 待機 (承認、アダプティブ カード) |同上 | `body('Name')?['...']` |
| `Http` |外部 HTTP 呼び出し | `inputs.{method, uri, headers, body}` | `outputs('Name')?['body']` |
| `Response` | HTTP 呼び出し元に戻る | `inputs.{statusCode, headers, body}` | — |
| `Query` |フィルター配列 | `inputs.{from, where}` | `body('Name')` (フィルターされた配列) |
| `Select` |配列の再形成/投影 | `inputs.{from, select}` | `body('Name')` (投影された配列) |
| `Table` |配列 → CSV/HTML 文字列 | `inputs.{from, format, columns}` | `body('Name')` (文字列) |
| `ParseJson` |スキーマを使用して JSON を解析する | `inputs.{content, schema}` | `body('Name')?['field']` |
| `Expression` |組み込み関数 (ConvertTimeZone など) | `kind`、`inputs` | `body('Name')` |

---

## コネクタの識別

`type: OpenApiConnection` が表示されたら、`host.apiId` からのコネクタを特定します。

| APIId サフィックス |コネクタ |
|---|---|
| `shared_sharepointonline` |シェアポイント |
| `shared_office365` | Outlook / Office 365 |
| `shared_teams` |マイクロソフトチーム |
| `shared_approvals` |承認 |
| `shared_office365users` | Office 365 ユーザー |
| `shared_flowmanagement` |フロー管理 |

`operationId` は、特定の操作を示します (例: `GetItems`、`SendEmailV2`、
`PostMessageToConversation`)。 `connectionName` は、次の GUID にマップされます。
`properties.connectionReferences`。

---

## 一般的な表現 (チートシートの読み方)|式 |意味 |
|---|---|
| `@outputs('X')?['body/value']` |コネクタアクション X | の結果の配列
| `@body('X')` |アクション X の直接本体 (Query、Select、ParseJson) |
| `@item()?['Field']` |現在のループ項目のフィールド |
| `@triggerBody()?['Field']` |トリガーペイロードフィールド |
| `@variables('name')` |変数値 |
| `@coalesce(a, b)` | a、b の最初の非 null |
| `@first(array)` |最初の要素 (空の場合は null) |
| `@length(array)` |配列数 |
| `@empty(value)` | null/空の文字列/空の配列の場合は True |
| `@union(a, b)` |配列を結合する — 重複した場合は **最初の勝利** |
| `@result('Scope')` |スコープ内のアクション結果の配列 |