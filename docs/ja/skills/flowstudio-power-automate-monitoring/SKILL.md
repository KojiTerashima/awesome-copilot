---
name: flowstudio-power-automate-monitoring
description: >-
  FlowStudio MCPのキャッシュストアを利用して、Power Automateフローの健康状態を監視し、失敗率を追跡し、テナント資産をインベントリします。ライブAPIは最上位の実行ステータスのみ返しますが、ストアツールは集計された統計情報、各実行の失敗詳細と対応ヒント、作成者の活動、Power Appsのインベントリなどを高速キャッシュから提供し、PA APIのレート制限の心配がありません。
  フローの健康チェック、失敗しているフローの検索、失敗率取得、エラー傾向の確認、監視有効な全フロー一覧、フロー作成者の確認、非アクティブな作成者の検索、Power Appsのインベントリ、環境や接続数の確認、フロー概要取得、テナント全体の健康状態確認などを求められた際にこのスキルを読み込んでください。FlowStudio for TeamsまたはMCP Pro+サブスクリプションが必要です — https://mcp.flowstudio.app を参照。
metadata:
  openclaw:
    requires:
      env:
        - FLOWSTUDIO_MCP_TOKEN
    primaryEnv: FLOWSTUDIO_MCP_TOKEN
    homepage: https://mcp.flowstudio.app
---

# FlowStudio MCPによるPower Automate監視

FlowStudio MCPの**キャッシュストア**を通じて、フローの健康状態監視、失敗率の追跡、テナント資産のインベントリが可能です。高速な読み込み、PA APIのレート制限なし、ガバナンスメタデータや対応ヒントも付加されています。

> **必要条件:** [FlowStudio for TeamsまたはMCP Pro+](https://mcp.flowstudio.app)
> サブスクリプションが必要です。
>
> **各セッションの開始時は必ず`tools/list`を実行**し、ツール名とパラメータを確認してください。
> このスキルはレスポンス形式、動作上の注意点、ワークフローのパターンなど、`tools/list`では分からない内容をカバーします。このドキュメントと`tools/list`や実際のAPIレスポンスが異なる場合は、APIの内容を優先してください。

---

## 監視の仕組み

Flow Studioは各サブスクライバーのPower Automate APIを毎日スキャンし、結果をキャッシュします。2つのレベルがあります：

- **全フロー**：メタデータをスキャン（定義、接続、所有者、トリガータイプ、集計実行統計情報（`runPeriodTotal`, `runPeriodFailRate`など））。環境、アプリ、接続、作成者もスキャンされます。
- **監視対象フロー**（`monitor: true`）：さらに各実行の詳細（個々の実行記録、ステータス、所要時間、失敗アクション名、対応ヒント）を取得します。これが`get_store_flow_runs`, `get_store_flow_errors`, `get_store_flow_summary`に反映されます。

**データの新鮮度:** フローが最後にスキャンされた日時は`get_store_flow`の`scanned`フィールドで確認できます。古い場合はスキャンパイプラインが動作していない可能性があります。

**監視の有効化:** `update_store_flow`またはFlow Studio for Teamsアプリから`monitor: true`を設定してください。
（[フロー選択方法](https://learn.flowstudio.app/teams-monitoring)）

**重要フローの指定:** ビジネスクリティカルなフローには`update_store_flow`で`critical=true`を設定してください。これによりガバナンススキルの通知ルール管理が自動で失敗アラートを設定します。

---

## ツール一覧

| ツール | 目的 |
|---|---|
| `list_store_flows` | 失敗率や監視フィルター付きフロー一覧 |
| `get_store_flow` | キャッシュされたフルレコード：実行統計、所有者、階層、接続、定義 |
| `get_store_flow_summary` | 集計実行統計：成功/失敗率、平均/最大所要時間 |
| `get_store_flow_runs` | 各実行履歴：所要時間、ステータス、失敗アクション、対応ヒント |
| `get_store_flow_errors` | 失敗のみの実行履歴：アクション名と対応ヒント |
| `get_store_flow_trigger_url` | キャッシュからトリガーURL取得（即時、PA API呼び出しなし） |
| `set_store_flow_state` | フローの開始/停止とキャッシュへの状態同期 |
| `update_store_flow` | 監視フラグ、通知ルール、タグ、ガバナンスメタデータ設定 |
| `list_store_environments` | 全Power Platform環境一覧 |
| `list_store_connections` | 全接続一覧 |
| `list_store_makers` | 全作成者（市民開発者）一覧 |
| `get_store_maker` | 作成者詳細：フロー/アプリ数、ライセンス、アカウント状態 |
| `list_store_power_apps` | 全Power Appsキャンバスアプリ一覧 |

---

## ストアとライブの使い分け

| 質問 | ストア利用 | ライブ利用 |
|---|---|---|
| 失敗しているフロー数は？ | `list_store_flows` | — |
| 30日間の失敗率は？ | `get_store_flow_summary` | — |
| フローのエラー履歴を表示 | `get_store_flow_errors` | — |
| このフローの作成者は誰？ | `get_store_flow` → `owners`解析 | — |
| フロー定義を全文取得 | `get_store_flow`（JSON文字列） | `get_live_flow`（構造化） |
| 実行のアクション入出力を調査 | — | `get_live_flow_run_action_outputs` |
| 失敗した実行を再送信 | — | `resubmit_live_flow_run` |

> ストアツールは「何が起きたか」「どれくらい健康か」を答えます。
> ライブツールは「何が正確に問題だったか」「今すぐ直す」を答えます。

> `get_store_flow_runs`, `get_store_flow_errors`, `get_store_flow_summary`が空の場合は、(1)フローの`monitor: true`が有効か、(2)`scanned`フィールドが新しいかを確認してください。両方とも`get_store_flow`で確認できます。

---

## レスポンス形式

### `list_store_flows`

配列形式。フィルター：`monitor`（bool）、`rule_notify_onfail`（bool）、`rule_notify_onmissingdays`（bool）。

```json
[
  {
    "id": "Default-<envGuid>.<flowGuid>",
    "displayName": "Stripe subscription updated",
    "state": "Started",
    "triggerType": "Request",
    "triggerUrl": "https://...",
    "tags": ["#operations", "#sensitive"],
    "environmentName": "Default-26e65220-...",
    "monitor": true,
    "runPeriodFailRate": 0.012,
    "runPeriodTotal": 82,
    "createdTime": "2025-06-24T01:20:53Z",
    "lastModifiedTime": "2025-06-24T03:51:03Z"
  }
]
```

> `id`形式：`Default-<envGuid>.<flowGuid>`。最初の`.`で分割すると`environmentName`と`flowName`が得られます。
>
> `triggerUrl`と`tags`はオプションです。一部のエントリは`sparse`（`id`と`monitor`のみ）なので、`displayName`がないものはスキップしてください。
>
> `list_store_flows`のタグはフローの`description`フィールドから自動抽出されます（作成者のハッシュタグ例：`#operations`）。`update_store_flow(tags=...)`で書き込んだタグは別途保存され、`get_store_flow`でのみ表示されます。リストレスポンスには出ません。

### `get_store_flow`

キャッシュされたフルレコード。主なフィールド：

| カテゴリ | フィールド |
|---|---|
| 識別 | `name`, `displayName`, `environmentName`, `state`, `triggerType`, `triggerKind`, `tier`, `sharingType` |
| 実行統計 | `runPeriodTotal`, `runPeriodFails`, `runPeriodSuccess`, `runPeriodFailRate`, `runPeriodSuccessRate`, `runPeriodDurationAverage`/`Max`/`Min`（ミリ秒）, `runTotal`, `runFails`, `runFirst`, `runLast`, `runToday` |
| ガバナンス | `monitor`（bool）, `rule_notify_onfail`（bool）, `rule_notify_onmissingdays`（数値）, `rule_notify_email`（文字列）, `log_notify_onfail`（ISO）, `description`, `tags` |
| 新鮮度 | `scanned`（ISO）, `nextScan`（ISO） |
| ライフサイクル | `deleted`（bool）, `deletedTime`（ISO） |
| JSON文字列 | `actions`, `connections`, `owners`, `complexity`, `definition`, `createdBy`, `security`, `triggers`, `referencedResources`, `runError` — すべて`json.loads()`で解析が必要 |

> 所要時間フィールド（`runPeriodDurationAverage`, `Max`, `Min`）は**ミリ秒**です。秒に変換する場合は1000で割ってください。
>
> `runError`は直近の実行エラーをJSON文字列で保持します。解析例：`json.loads(record["runError"])`。エラーがない場合は`{}`を返します。

### `get_store_flow_summary`

指定期間（デフォルト：直近7日間）の集計統計。

```json
{
  "flowKey": "Default-<envGuid>.<flowGuid>",
  "windowStart": null,
  "windowEnd": null,
  "totalRuns": 82,
  "successRuns": 81,
  "failRuns": 1,
  "successRate": 0.988,
  "failRate": 0.012,
  "averageDurationSeconds": 2.877,
  "maxDurationSeconds": 9.433,
  "firstFailRunRemediation": null,
  "firstFailRunUrl": null
}
```

> この期間に実行データがない場合は全てゼロを返します。
> `startTime`と`endTime`（ISO 8601）パラメータで期間を変更できます。

### `get_store_flow_runs` / `get_store_flow_errors`

配列形式。`get_store_flow_errors`は`status=Failed`のみを返します。
パラメータ：`startTime`, `endTime`, `status`（配列：`["Failed"]`, `["Succeeded"]`など）。

> 実行データがない場合は`[]`を返します。

### `get_store_flow_trigger_url`

```json
{
  "flowKey": "Default-<envGuid>.<flowGuid>",
  "displayName": "Stripe subscription updated",
  "triggerType": "Request",
  "triggerKind": "Http",
  "triggerUrl": "https://..."
}
```

> HTTPトリガー以外の場合は`triggerUrl`はnullです。

### `set_store_flow_state`

ライブPA APIを呼び出し、状態をキャッシュに同期し、更新後のフルレコードを返します。

```json
{
  "flowKey": "Default-<envGuid>.<flowGuid>",
  "requestedState": "Stopped",
  "currentState": "Stopped",
  "flow": { /* get_store_flowと同じ形式のgFlowsレコード */ }
}
```

> 埋め込まれた`flow`オブジェクトは新しい状態を即時反映します。追加の`get_store_flow`呼び出しは不要です。フロー停止後にタグ/監視/所有者メタデータを同時に読み取るガバナンスワークフローに便利です。
>
> 状態変更の機能は`set_live_flow_state`と同等ですが、`set_live_flow_state`は`{flowName, environmentName, requestedState, actualState}`のみ返し、キャッシュ同期はしません。キャッシュの新鮮度が不要なら`set_live_flow_state`を推奨します。

### `update_store_flow`

ガバナンスメタデータを更新します。指定したフィールドのみ更新（マージ）。
更新後のフルレコード（`get_store_flow`と同形式）を返します。

設定可能フィールド：`monitor`（bool）, `rule_notify_onfail`（bool）, `rule_notify_onmissingdays`（数値, 0=無効）, `rule_notify_email`（カンマ区切り）, `description`, `tags`, `businessImpact`, `businessJustification`, `businessValue`, `ownerTeam`, `ownerBusinessUnit`, `supportGroup`, `supportEmail`, `critical`（bool）, `tier`, `security`

### `list_store_environments`

配列形式。

```json
[
  {
    "id": "Default-26e65220-...",
    "displayName": "Flow Studio (default)",
    "sku": "Default",
    "type": "NotSpecified",
    "location": "australia",
    "isDefault": true,
    "isAdmin": true,
    "isManagedEnvironment": false,
    "createdTime": "2017-01-18T01:06:46Z"
  }
]
```

> `sku`値：`Default`, `Production`, `Developer`, `Sandbox`, `Teams`

### `list_store_connections`

配列形式。非常に多くなる場合あり（1500件以上）。

```json
[
  {
    "id": "<environmentId>.<connectionId>",
    "displayName": "user@contoso.com",
    "createdBy": "{\"id\":\"...\",\"displayName\":\"...\",\"email\":\"...\"}",
    "environmentName": "...",
    "statuses": "[{\"status\":\"Connected\"}]"
  }
]
```

> `createdBy`と`statuses`は**JSON文字列**なので、`json.loads()`で解析してください。

### `list_store_makers`

配列形式。

```json
[
  {
    "id": "09dbe02f-...",
    "displayName": "Catherine Han",
    "mail": "catherine.han@flowstudio.app",
    "deleted": false,
    "ownerFlowCount": 199,
    "ownerAppCount": 209,
    "userIsServicePrinciple": false
  }
]
```

> 削除済み作成者は`deleted: true`で、`displayName`や`mail`フィールドがありません。

### `get_store_maker`

作成者のフルレコード。主なフィールド：`displayName`, `mail`, `userPrincipalName`, `ownerFlowCount`, `ownerAppCount`, `accountEnabled`, `deleted`, `country`, `firstFlow`, `firstFlowCreatedTime`, `lastFlowCreatedTime`, `firstPowerApp`, `lastPowerAppCreatedTime`, `licenses`（M365 SKUのJSON文字列）

### `list_store_power_apps`

配列形式。

```json
[
  {
    "id": "<environmentId>.<appId>",
    "displayName": "My App",
    "environmentName": "...",
    "ownerId": "09dbe02f-...",
    "ownerName": "Catherine Han",
    "appType": "Canvas",
    "sharedUsersCount": 0,
    "createdTime": "2023-08-18T01:06:22Z",
    "lastModifiedTime": "2023-08-18T01:06:22Z",
    "lastPublishTime": "2023-08-18T01:06:22Z"
  }
]
```

---

## よくあるワークフロー

### 不健康なフローの検索

```
1. list_store_flows
2. runPeriodFailRate > 0.1かつrunPeriodTotal >= 5でフィルタ
3. runPeriodFailRate降順でソート
4. 各フローについてget_store_flowで詳細取得
```

### 特定フローの健康状態確認

```
1. get_store_flow → scanned（新鮮度）、runPeriodFailRate、runPeriodTotalを確認
2. get_store_flow_summary → 集計統計（期間指定可）
3. get_store_flow_errors → 各実行の失敗詳細と対応ヒント
4. 詳細診断が必要ならライブツールへ：
   get_live_flow_runs → get_live_flow_run_action_outputs
```

### フローの監視有効化

```
1. update_store_flowでmonitor=trueを設定
2. 必要に応じてrule_notify_onfail=true, rule_notify_email="user@domain.com"を設定
3. 次回の毎日スキャン後に実行データが表示されます
```

### 毎日の健康チェック

```
1. list_store_flows
2. runPeriodFailRate > 0.2かつrunPeriodTotal >= 3のフローをフラグ
3. state="Stopped"の監視フローもフラグ（自動停止の可能性あり）
4. 重大な失敗はget_store_flow_errorsで対応ヒント確認
```

### 作成者監査

```
1. list_store_makers
2. deleted=trueかつownerFlowCount > 0の削除済みアカウントを特定
3. get_store_makerで特定ユーザーの詳細取得
```

### インベントリ

```
1. list_store_environments → 環境数、SKU、場所
2. list_store_flows → 状態別フロー数、トリガータイプ、失敗率
3. list_store_power_apps → アプリ数、所有者、共有状況
4. list_store_connections → 環境ごとの接続数
```

---

## 関連スキル

- `power-automate-mcp` — コア接続設定、ライブツール参照
- `power-automate-debug` — アクションレベルの入出力による詳細診断（ライブAPI）
- `power-automate-build` — フロー定義の構築とデプロイ
- `power-automate-governance` — ガバナンスメタデータ、タグ付け、通知ルール、CoEパターン
