# FlowStudio MCP — ツール レスポンス カタログ

FlowStudio Power Automate MCP サーバーの応答形状と動作に関するメモ。

> **ツール名とパラメータについて**: 常にサーバー上で `tools/list` を呼び出します。
> すべてのツールの信頼できる最新のスキーマを返します。
> このドキュメントでは、`tools/list` では分からないことについて説明します: **応答形状**
> および実際の使用を通じて発見された **明白ではない動作**。

---

## 真実の情報源

|優先順位 |出典 |カバー |
|----------|----------|----------|
| 1 | **実際の API レスポンス** |サーバーが実際に返すものを常に信頼してください。
| 2 | **`tools/list`** |ツール名、パラメータ名、タイプ、必要なフラグ |
| 3 | **このドキュメント** |応答形状、行動メモ、注意点 |

> このドキュメントが `tools/list` または実際の API の動作と一致しない場合は、
> API が勝ちます。それに応じてこのドキュメントを更新してください。

---

## 環境とテナントの探索

### `list_live_environments`

応答: 環境の直接配列。```json
[
  {
    "id": "Default-26e65220-5561-46ef-9783-ce5f20489241",
    "displayName": "FlowStudio (default)",
    "sku": "Production",
    "location": "australia",
    "state": "Enabled",
    "isDefault": true,
    "isAdmin": true,
    "isMember": true,
    "createdTime": "2023-08-18T00:41:05Z"
  }
]
```> 他のすべてのツールでは、`id` 値を `environmentName` として使用します。

### `list_store_environments`

`list_live_environments` と同じ形式ですが、キャッシュから読み取られます (高速です)。

---

## 接続の検出

### `list_live_connections`

応答: `connections` 配列を含むラッパー オブジェクト。```json
{
  "connections": [
    {
      "id": "shared-office365-9f9d2c8e-55f1-49c9-9f9c-1c45d1fbbdce",
      "displayName": "user@contoso.com",
      "connectorName": "shared_office365",
      "createdBy": "User Name",
      "statuses": [{"status": "Connected"}],
      "createdTime": "2024-03-12T21:23:55.206815Z"
    }
  ],
  "totalCount": 56,
  "error": null
}
```> **キー フィールド**: `id` は、`connectionReferences` で使用される `connectionName` の値です。
>
> **キー フィールド**: `connectorName` は apiId にマップされます:
> @@コード4@@
>
> ステータスでフィルタリングします: `statuses[0].status == "Connected"`。
>
> **注意**: `tools/list` は `environmentName` をオプションとしてマークしますが、サーバーは
> 省略した場合は `MissingEnvironmentFilter` (HTTP 400) を返します。必ず通過します
> @@コード9@@。

### `list_store_connections`

キャッシュからの同じ接続データ。

---

## フローの検出とリスト

### `list_live_flows`

応答: `flows` 配列を含むラッパー オブジェクト。```json
{
  "mode": "owner",
  "flows": [
    {
      "id": "0757041a-8ef2-cf74-ef06-06881916f371",
      "displayName": "My Flow",
      "state": "Started",
      "triggerType": "Request",
      "triggerKind": "Http",
      "createdTime": "2023-08-18T01:18:17Z",
      "lastModifiedTime": "2023-08-18T12:47:42Z",
      "owners": "<aad-object-id>",
      "definitionAvailable": true
    }
  ],
  "totalCount": 100,
  "error": null
}
```> `result["flows"]` からアクセスします。 `id` はプレーンな UUID --- `flowName` として直接使用します。
>
> `mode` は、使用されるアクセス範囲 (`"owner"` または `"admin"`) を示します。

### `list_store_flows`

応答: **直接配列** (ラッパーなし)。```json
[
  {
    "id": "3991358a-f603-e49d-b1ed-a9e4f72e2dcb.0757041a-8ef2-cf74-ef06-06881916f371",
    "displayName": "Admin | Sync Template v3 (Solutions)",
    "state": "Started",
    "triggerType": "OpenApiConnectionWebhook",
    "environmentName": "3991358a-f603-e49d-b1ed-a9e4f72e2dcb",
    "runPeriodTotal": 100,
    "createdTime": "2023-08-18T01:18:17Z",
    "lastModifiedTime": "2023-08-18T12:47:42Z"
  }
]
```> **`id` 形式**: `<environmentId>.<flowId>` --- 最初の `.` で分割してフロー UUID を抽出します。
> @@コード3@@

### `get_store_flow`

応答: キャッシュからの単一フロー メタデータ (選択されたフィールド)。```json
{
  "id": "<environmentId>.<flowId>",
  "displayName": "My Flow",
  "state": "Started",
  "triggerType": "Recurrence",
  "runPeriodTotal": 100,
  "runPeriodFailRate": 0.1,
  "runPeriodSuccessRate": 0.9,
  "runPeriodFails": 10,
  "runPeriodSuccess": 90,
  "runPeriodDurationAverage": 29410.8,
  "runPeriodDurationMax": 158900.0,
  "runError": "{\"code\": \"EACCES\", ...}",
  "description": "Flow description",
  "tier": "Premium",
  "complexity": "{...}",
  "actions": 42,
  "connections": ["sharepointonline", "office365"],
  "owners": ["user@contoso.com"],
  "createdBy": "user@contoso.com"
}
```> `runPeriodDurationAverage` / `runPeriodDurationMax` 単位は **ミリ秒** (1000 で割ります)。
> `runError` は **JSON 文字列 ** --- `json.loads()` で解析します。

---

## フロー定義 (ライブ API)

### `get_live_flow`

応答: PA API からの完全なフロー定義。```json
{
  "name": "<flow-guid>",
  "properties": {
    "displayName": "My Flow",
    "state": "Started",
    "definition": {
      "triggers": { "..." },
      "actions": { "..." },
      "parameters": { "..." }
    },
    "connectionReferences": { "..." }
  }
}
```### `update_live_flow`

**作成モード**: `flowName` を省略 --- 新しいフローを作成します。 `definition` と `displayName` は必須です。

**更新モード**: `flowName` を指定します --- 既存のフローにパッチを適用します。

応答：```json
{
  "created": false,
  "flowKey": "<environmentId>.<flowId>",
  "updated": ["definition", "connectionReferences"],
  "displayName": "My Flow",
  "state": "Started",
  "definition": { "...full definition..." },
  "error": null
}
```> `error` は **常に存在します** が、`null` になる場合もあります。 `result.get("error") is not None`を確認してください。
>
> 作成時: `created` は新しいフロー GUID (文字列) です。更新時: `created` は `false` です。
>
> `description` は **常に必須** (作成および更新)。

### @@コード7@@

非ソリューション フローをソリューションに移行します。すでにソリューション内にある場合はエラーを返します。

---

## 実行履歴と監視

### `get_live_flow_runs`

応答: 実行の直接配列 (新しいものから順)。```json
[{
  "name": "<run-id>",
  "status": "Succeeded|Failed|Running|Cancelled",
  "startTime": "2026-02-25T06:13:38Z",
  "endTime": "2026-02-25T06:14:02Z",
  "triggerName": "Recurrence",
  "error": null
}]
```> `top` のデフォルトは **30** で、値が大きくなると自動ページ分割されます。 `top: 300`を設定します
> 5 分ごとに実行されるフローを 24 時間カバーします。
>
> 実行 ID フィールドは **`name`** (`runName` ではありません) です。この値を `runName` として使用します
> 他のツールのパラメータ。

### `get_live_flow_run_error`

応答: 失敗した実行の構造化エラーの内訳。```json
{
  "runName": "08584296068667933411438594643CU15",
  "failedActions": [
    {
      "actionName": "Apply_to_each_prepare_workers",
      "status": "Failed",
      "error": {"code": "ActionFailed", "message": "An action failed."},
      "code": "ActionFailed",
      "startTime": "2026-02-25T06:13:52Z",
      "endTime": "2026-02-25T06:15:24Z"
    },
    {
      "actionName": "HTTP_find_AD_User_by_Name",
      "status": "Failed",
      "code": "NotSpecified",
      "startTime": "2026-02-25T06:14:01Z",
      "endTime": "2026-02-25T06:14:05Z"
    }
  ],
  "allActions": [
    {"actionName": "Apply_to_each", "status": "Skipped"},
    {"actionName": "Compose_WeekEnd", "status": "Succeeded"},
    {"actionName": "HTTP_find_AD_User_by_Name", "status": "Failed"}
  ]
}
```> `failedActions` は外側から内側の順序です --- **最後のエントリが根本原因です**。
> `failedActions[-1]["actionName"]` を診断の開始点として使用します。

### `get_live_flow_run_action_outputs`

応答: アクション詳細オブジェクトの配列。```json
[
  {
    "actionName": "Compose_WeekEnd_now",
    "status": "Succeeded",
    "startTime": "2026-02-25T06:13:52Z",
    "endTime": "2026-02-25T06:13:52Z",
    "error": null,
    "inputs": "Mon, 25 Feb 2026 06:13:52 GMT",
    "outputs": "Mon, 25 Feb 2026 06:13:52 GMT"
  }
]
```> **`actionName` はオプションです**: 実行中のすべてのアクションを返すには省略します。
> そのアクションのみの単一要素の配列を返すように指定します。
>
> バルクデータアクションの場合、出力は非常に大きくなる可能性があります (50 MB 以上)。 120 秒以上のタイムアウトを使用します。

---

## 実行制御

### `resubmit_live_flow_run`

応答: `{ flowKey, resubmitted: true, runName, triggerName }`

### `cancel_live_flow_run`

`Running` フローの実行をキャンセルします。

> アダプティブ カードの応答を待っている実行をキャンセルしないでください --- status `Running`
> Teams カードがユーザー入力を待っている間は正常です。

---

## HTTP トリガー ツール

### `get_live_flow_http_schema`

応答キー:```
flowKey            - Flow GUID
displayName        - Flow display name
triggerName        - Trigger action name (e.g. "manual")
triggerType        - Trigger type (e.g. "Request")
triggerKind        - Trigger kind (e.g. "Http")
requestMethod      - HTTP method (e.g. "POST")
relativePath       - Relative path configured on the trigger (if any)
requestSchema      - JSON schema the trigger expects as POST body
requestHeaders     - Headers the trigger expects
responseSchemas    - Array of JSON schemas defined on Response action(s)
responseSchemaCount - Number of Response actions that define output schemas
```> リクエスト本文のスキーマは `requestSchema` (`triggerSchema` ではありません) にあります。

### `get_live_flow_trigger_url`

HTTP によってトリガーされるフローの署名付きコールバック URL を返します。応答には以下が含まれます
`flowKey`、`triggerName`、`triggerType`、`triggerKind`、`triggerMethod`、`triggerUrl`。

### `trigger_live_flow`

応答キー: `flowKey`、`triggerName`、`triggerUrl`、`requiresAadAuth`、`authType`、
`responseStatus`、`responseBody`。

> **`Request` (HTTP) トリガーでのみ機能します。** 繰り返しの場合はエラーが返されます。
> およびその他のトリガー タイプ: `"only HTTP Request triggers can be invoked via this tool"`。
> `Button` 種類のトリガーは `ListCallbackUrlOperationBlocked` を返します。
>
> `responseStatus` + `responseBody` には、フローの応答アクション出力が含まれます。
> AAD 認証されたトリガーは自動的に処理されます。
>
> **コンテンツタイプに関する注意**: 本文は `application/octet-stream` (生) として送信されます。
> `application/json` ではありません。 `required` フィールドを持つトリガー スキーマを含むフロー
> PA が検証するため、`InvalidRequestContent` (400) でリクエストを拒否します。
> `Content-Type` スキーマに対して解析する前に。スキーマのないフロー、または
> 生の入力を受け入れるように設計されたフロー (例: 本体を解析するベイカー パターン フロー)
> 内部的に)、正常に動作します。フローは JSON を Base64 でエンコードされたものとして受信します
> `$content` と `$content-type: application/octet-stream`。

---

## フロー状態管理

### `set_live_flow_state`

ライブ PA API を介して Power Automate フローを開始または停止します。 **必要ありません**
Power Clarity ワークスペース — 偽装アカウントがアクセスできるあらゆるフローで機能します。
最初に現在の状態を読み取り、変更があった場合にのみ開始/停止呼び出しを発行します。
実際に必要です。

パラメータ: `environmentName`、`flowName`、`state` (`"Started"` | `"Stopped"`) — すべて必須。

応答：```json
{
  "flowName": "6321ab25-7eb0-42df-b977-e97d34bcb272",
  "environmentName": "Default-26e65220-...",
  "requestedState": "Started",
  "actualState": "Started"
}
```> **フローを開始または停止するには、`update_live_flow` ではなくこのツールを使用してください**。
> `update_live_flow` は表示名/定義のみを変更します。 PA API は無視します
> 状態はそのエンドポイントを通過しました。

### `set_store_flow_state`

ライブ PA API 経由でフローを開始または停止し、** 更新された状態を保持します。
Power Clarity キャッシュに保存します。 `set_live_flow_state` と同じパラメータですが、必須です
Power Clarity ワークスペース。

応答 (`set_live_flow_state` とは異なる形式):```json
{
  "flowKey": "<environmentId>.<flowId>",
  "requestedState": "Stopped",
  "currentState": "Stopped",
  "flow": { /* full gFlows record, same shape as get_store_flow */ }
}
```> 状態を切り替えるだけの場合は `set_live_flow_state` を優先します。
> よりシンプルで、サブスクリプション要件もありません。
>
> キャッシュをすぐに更新する必要がある場合は、`set_store_flow_state` を使用します
> (次回の毎日のスキャンを待たずに) 完全な更新を希望します
> 同じ通話内でガバナンス レコードを戻す - 次のようなワークフローに役立ちます。
> フローを停止し、すぐにタグ付けするか検査します。

---

## ストア ツール --- FlowStudio for Teams のみ

### `get_store_flow_summary`

応答: 集計された実行統計。```json
{
  "totalRuns": 100,
  "failRuns": 10,
  "failRate": 0.1,
  "averageDurationSeconds": 29.4,
  "maxDurationSeconds": 158.9,
  "firstFailRunRemediation": "<hint or null>"
}
```### `get_store_flow_runs`

過去 N 日間のキャッシュされた実行履歴と、期間と修復のヒント。

### `get_store_flow_errors`

キャッシュされた失敗のみの実行には、失敗したアクション名と修復ヒントが含まれます。

### `get_store_flow_trigger_url`

キャッシュから URL をトリガーします (インスタント、PA API 呼び出しなし)。

### `update_store_flow`

ガバナンス メタデータ (説明、タグ、監視フラグ、通知ルール、ビジネスへの影響) を更新します。

### `list_store_makers` / `get_store_maker`

メーカー（市民開発者）の発見と詳細。

### `list_store_power_apps`

キャッシュからすべての Power Apps キャンバス アプリを一覧表示します。

---

## 行動に関するメモ

実際の API の使用を通じて発見された非明白な動作。これらは物事です
`tools/list` ではわかりません。

### `get_live_flow_run_action_outputs`
- **`actionName` はオプションです**: すべてのアクションを取得する場合は省略し、1 つを取得する場合は指定します。
  これにより、応答が N 要素から 1 要素 (配列のまま) に変更されます。
- バルク データ アクションの出力は 50 MB 以上になる可能性があります --- 常に 120 秒以上のタイムアウトを使用します。

### `update_live_flow`
- `description` は **常に必須** (作成モードと更新モード)。
- `error` キーは応答に **常に存在します** --- `null` は成功を意味します。
  `if "error" in result` はチェックしないでください。 `result.get("error") is not None`を確認してください。
- 作成時、`created` = 新しいフロー GUID (文字列)。更新時は、`created` = `false` となります。
- **フロー状態は変更できません。** 表示名、定義、および表示のみを更新します。
  接続参照。フローを開始/停止するには、`set_live_flow_state` を使用します。

### `trigger_live_flow`
- **HTTP リクエスト トリガーに対してのみ機能します。** 繰り返し、コネクタ、
  およびその他のトリガータイプ。
- AAD 認証されたトリガーは自動的に処理されます (偽装ベアラー トークン)。

### `get_live_flow_runs`
- `top` のデフォルトは **30** で、より大きな値の場合は自動ページネーションが使用されます。
- 実行 ID フィールドは `runName` ではなく `name` です。他のツールでは、この値を `runName` として使用します。
- 実行は新しい順に返されます。

### チーム `PostMessageToConversation` (`update_live_flow` 経由)
- **「フロー ボットとチャット」**: `body/recipient` = `"user@domain.com;"` (末尾にセミコロンが付いている文字列)。
- **「チャネル」**: `body/recipient` = `{"groupId": "...", "channelId": "..."}` (オブジェクト)。
- `poster`: ワークフロー ボット ID の場合は `"Flow bot"`、ユーザー ID の場合は `"User"`。

### `list_live_connections`
- `id` は、`connectionReferences` の `connectionName` に必要な値です。
- `connectorName` は apiId: `"/providers/Microsoft.PowerApps/apis/" + connectorName` にマップされます。