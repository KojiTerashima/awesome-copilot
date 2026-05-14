# FlowStudio MCP — Power Automate の一般的なエラー

デバッグ時のエラー コード、考えられる原因、推奨される修正方法のリファレンス
Power Automate フローは、FlowStudio MCP サーバーを介して行われます。

---

## 式/テンプレートのエラー

### `InvalidTemplate` — Null に適用される関数

**完全なメッセージ パターン**: `"Unable to process template language expressions... function 'split' expects its first argument 'text' to be of type string"`

**根本原因**: `@split(item()?['Name'], ' ')` のような式が null 値を受け取りました。

**診断**:
1. エラー メッセージ内のアクション名をメモします。
2. 配列を生成するアクションで `get_live_flow_run_action_outputs` を呼び出します。
3. `Name` (または参照されるフィールド) が `null` である項目を検索します。

**修正**:```
Before: @split(item()?['Name'], ' ')
After:  @split(coalesce(item()?['Name'], ''), ' ')

Or guard the whole foreach body with a condition:
  expression: "@not(empty(item()?['Name']))"
```---

### `InvalidTemplate` — 間違った式パス

**完全なメッセージ パターン**: `"Unable to process template language expressions... 'triggerBody()?['FieldName']' is of type 'Null'"`

**根本原因**: 式内のフィールド名が実際のペイロード スキーマと一致しません。

**診断**：```python
# Check trigger output shape
mcp("get_live_flow_run_action_outputs",
    environmentName=ENV, flowName=FLOW_ID, runName=RUN_ID,
    actionName="<trigger-name>")
# Compare actual keys vs expression
```**修正**: 正しいキー名を使用するように式を更新します。よくある不一致:
- `triggerBody()?['body']` と `triggerBody()?['Body']` (大文字と小文字を区別)
- `triggerBody()?['Subject']` 対 `triggerOutputs()?['body/Subject']`

---

### `InvalidTemplate` — 型の不一致

**完全なメッセージ パターン**: `"... expected type 'Array' but got type 'Object'"`

**根本原因**: 式で配列が必要な場所にオブジェクトを渡します (例: 単一項目の HTTP 応答とリスト応答)。

**修理**：```
Before: @outputs('HTTP')?['body']
After:  @outputs('HTTP')?['body/value']    ← for OData list responses
        @createArray(outputs('HTTP')?['body'])  ← wrap single object in array
```---

## 接続/認証エラー

### `ConnectionAuthorizationFailed`

**メッセージ全文**: `"The API connection ... is not authorized."`

**根本原因**: フロー内で参照されている接続は、別のユーザーによって所有されています。
JWT が使用されているアカウントよりもユーザー/サービス アカウントが異なります。

**診断**: `properties.connectionReferences` — `connectionName` GUID を確認してください
所有者を特定します。 API経由では修正できません。

**修正オプション**:
1. Power Automate デザイナーでフローを開く → 接続を再認証する
2. トークンを保持しているサービス アカウントが所有する接続を使用します。
3. PA 管理者のサービス アカウントと接続を共有します。

---

### `InvalidConnectionCredentials`

**根本原因**: 接続の基礎となる OAuth トークンの有効期限が切れているか、
ユーザーの資格情報が変更されました。

**修正**: 所有者は Power Automate にサインインし、接続を更新する必要があります。

---

## HTTP アクション エラー

### `ActionFailed` — HTTP 4xx/5xx

**完全なメッセージ パターン**: `"An HTTP request to... failed with status code '400'"`

**診断**:```python
actions_out = mcp("get_live_flow_run_action_outputs", ..., actionName="HTTP_My_Call")
item = actions_out[0]   # first entry in the returned array
print(item["outputs"]["statusCode"])   # 400, 401, 403, 500...
print(item["outputs"]["body"])         # error details from target API
```**一般的な原因**:
- 401 — 認証ヘッダーが見つからないか期限切れです
- 403 — ターゲットリソースに対する権限が拒否されました
- 404 — 間違った URL / リソースが削除されました
- 400 — 不正な形式の JSON 本文 (本文を構築する式を確認してください)

---

### `ActionFailed` — HTTP タイムアウト

**根本原因**: ターゲット エンドポイントがコネクタのタイムアウト内に応答しませんでした
(HTTP アクションのデフォルトは 90 秒)。

**修正**: HTTP アクションに再試行ポリシーを追加するか、ペイロードをより小さいものに分割します。
バッチを使用してリクエストごとの処理時間を短縮します。

---

## 制御フローのエラー

### `ActionSkipped` を実行する代わりに

**根本原因**: `runAfter` 条件が満たされませんでした。例えば。に設定されたアクション
`Prev` が失敗したかスキップされた場合、`runAfter: { "Prev": ["Succeeded"] }` は実行されません。

**診断**: 前のアクションのステータスを確認します。意図的にスキップした
(例: false ブランチ内) は意図的です。予期しないスキップは論理ギャップです。

**修正**:
それらの結果に対してもアクションを実行する必要があります。

---

### Foreach が間違った順序で実行される / 競合状態

**根本原因**: `operationOptions: "Sequential"` なしで `Foreach` が実行される
並列で反復すると、書き込み競合や未定義の順序付けが発生します。

**修正**: `"operationOptions": "Sequential"` を Foreach アクションに追加します。

---

## 更新/デプロイのエラー

### `update_live_flow` は NoOp を返します

**症状**: `result["updated"]` が空のリスト、または `result["created"]` が空です。

**考えられる原因**: 間違ったパラメータ名を渡しています。必要なキーは `definition` です
(オブジェクト)、`flowDefinition` または `body` ではありません。

---

### `update_live_flow` — `"Supply connectionReferences"`

**根本原因**: 定義に `OpenApiConnection` または
`OpenApiConnectionWebhook` アクションが渡されませんでした。

**修正**: `get_live_flow` を使用して既存の接続参照をフェッチし、渡します
それらを `connectionReferences` 引数として使用します。

---

## データロジックエラー

### `union()` 正しいレコードを Null で上書きする

**症状**: 2 つの配列をマージした後、一部のレコードに null フィールドが存在します。
ソース配列の 1 つで。

**根本原因**: `union(old_data, new_data)` — `union()` が先勝のため、old_data
値は、一致するレコードの new_data をオーバーライドします。

**修正**: 引数の順序を入れ替えます: `union(new_data, old_data)````
Before: @sort(union(outputs('Old_Array'), body('New_Array')), 'Date')
After:  @sort(union(body('New_Array'), outputs('Old_Array')), 'Date')
```
