---
name: flowstudio-power-automate-debug
description: >-
  Debug failing Power Automate cloud flows using the FlowStudio MCP server.
  The Graph API only shows top-level status codes. This skill gives your agent
  action-level inputs and outputs to find the actual root cause.
  Load this skill when asked to: debug a flow, investigate a failed run, why is
  this flow failing, inspect action outputs, find the root cause of a flow error,
  fix a broken Power Automate flow, diagnose a timeout, trace a DynamicOperationRequestFailure,
  check connector auth errors, read error details from a run, or troubleshoot
  expression failures. Requires a FlowStudio MCP subscription — see https://mcp.flowstudio.app
metadata:
  openclaw:
    requires:
      env:
        - FLOWSTUDIO_MCP_TOKEN
    primaryEnv: FLOWSTUDIO_MCP_TOKEN
    homepage: https://mcp.flowstudio.app
---
# FlowStudio MCP を使用した Power Automate デバッグ

失敗した Power Automate を調査するための段階的な診断プロセス
クラウドは FlowStudio MCP サーバーを介して流れます。

> **実際のデバッグ例**: [子フローの式エラー](https://github.com/ninihen1/power-automate-mcp-skills/blob/main/examples/fix-expression-error.md) |
> [フローのバグではなくデータ入力](https://github.com/ninihen1/power-automate-mcp-skills/blob/main/examples/data-not-flow.md) |
> [Null 値により子フローがクラッシュする](https://github.com/ninihen1/power-automate-mcp-skills/blob/main/examples/null-child-flow.md)

**前提条件**: FlowStudio MCP サーバーは有効な JWT でアクセス可能である必要があります。
接続セットアップについては、`flowstudio-power-automate-mcp` スキルを参照してください。  
https://mcp.flowstudio.app で購読してください

---

## 真実の情報源

> **必ず最初に `tools/list` に電話して**、利用可能なツール名とそのツールを確認してください
> パラメータスキーマ。ツールの名前とパラメータはサーバーのバージョン間で異なる場合があります。
> このスキルは、応答形状、行動メモ、診断パターンをカバーします —
> `tools/list` ではお伝えできないこと。この文書が `tools/list` と異なる場合
> または実際の API 応答の場合、API が勝ちます。

---

## Python ヘルパー```python
import json, urllib.request

MCP_URL   = "https://mcp.flowstudio.app/mcp"
MCP_TOKEN = "<YOUR_JWT_TOKEN>"

def mcp(tool, **kwargs):
    payload = json.dumps({"jsonrpc": "2.0", "id": 1, "method": "tools/call",
                          "params": {"name": tool, "arguments": kwargs}}).encode()
    req = urllib.request.Request(MCP_URL, data=payload,
        headers={"x-api-key": MCP_TOKEN, "Content-Type": "application/json",
                 "User-Agent": "FlowStudio-MCP/1.0"})
    try:
        resp = urllib.request.urlopen(req, timeout=120)
    except urllib.error.HTTPError as e:
        body = e.read().decode("utf-8", errors="replace")
        raise RuntimeError(f"MCP HTTP {e.code}: {body[:200]}") from e
    raw = json.loads(resp.read())
    if "error" in raw:
        raise RuntimeError(f"MCP error: {json.dumps(raw['error'])}")
    return json.loads(raw["result"]["content"][0]["text"])

ENV = "<environment-id>"   # e.g. Default-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```---

## ステップ 1 — フローを見つける```python
result = mcp("list_live_flows", environmentName=ENV)
# Returns a wrapper object: {mode, flows, totalCount, error}
target = next(f for f in result["flows"] if "My Flow Name" in f["displayName"])
FLOW_ID = target["id"]   # plain UUID — use directly as flowName
print(FLOW_ID)
```---

## ステップ 2 — 失敗した実行を見つける```python
runs = mcp("get_live_flow_runs", environmentName=ENV, flowName=FLOW_ID, top=5)
# Returns direct array (newest first):
# [{"name": "08584296068667933411438594643CU15",
#   "status": "Failed",
#   "startTime": "2026-02-25T06:13:38.6910688Z",
#   "endTime": "2026-02-25T06:15:24.1995008Z",
#   "triggerName": "manual",
#   "error": {"code": "ActionFailed", "message": "An action failed..."}},
#  {"name": "...", "status": "Succeeded", "error": null, ...}]

for r in runs:
    print(r["name"], r["status"], r["startTime"])

RUN_ID = next(r["name"] for r in runs if r["status"] == "Failed")
```---

## ステップ 3 — 最上位のエラーを取得する

> **重大**: `get_live_flow_run_error` は、**どの**アクションが失敗したかを示します。
> `get_live_flow_run_action_outputs` は **理由** を教えてくれます。両方に電話する必要があります。
> 決してエラーだけで終わらせないでください — `ActionFailed` のようなエラー コード
> `NotSpecified` および `InternalServerError` は汎用ラッパーです。実際の
> 根本原因 (間違ったフィールド、null 値、HTTP 500 ボディ、スタック トレース) は、
> アクションの入力と出力に表示されます。```python
err = mcp("get_live_flow_run_error",
    environmentName=ENV, flowName=FLOW_ID, runName=RUN_ID)
# Returns:
# {
#   "runName": "08584296068667933411438594643CU15",
#   "failedActions": [
#     {"actionName": "Apply_to_each_prepare_workers", "status": "Failed",
#      "error": {"code": "ActionFailed", "message": "An action failed..."},
#      "startTime": "...", "endTime": "..."},
#     {"actionName": "HTTP_find_AD_User_by_Name", "status": "Failed",
#      "code": "NotSpecified", "startTime": "...", "endTime": "..."}
#   ],
#   "allActions": [
#     {"actionName": "Apply_to_each", "status": "Skipped"},
#     {"actionName": "Compose_WeekEnd", "status": "Succeeded"},
#     ...
#   ]
# }

# failedActions is ordered outer-to-inner. The ROOT cause is the LAST entry:
root = err["failedActions"][-1]
print(f"Root action: {root['actionName']} → code: {root.get('code')}")

# allActions shows every action's status — useful for spotting what was Skipped
# See common-errors.md to decode the error code.
```---

## ステップ 4 — 失敗したアクションの入力と出力を検査する

> **これは最も重要なステップです。** `get_live_flow_run_error` は
> 一般的なエラー コードです。実際のエラーの詳細 — HTTP ステータス コード、
> レスポンスボディ、スタックトレース、null 値 - アクションのランタイムに存在します
> 入力と出力。 **失敗したアクションの直後に必ず検査してください。
> それを特定しています。**```python
# Get the root failing action's full inputs and outputs
root_action = err["failedActions"][-1]["actionName"]
detail = mcp("get_live_flow_run_action_outputs",
    environmentName=ENV,
    flowName=FLOW_ID,
    runName=RUN_ID,
    actionName=root_action)

out = detail[0] if detail else {}
print(f"Action: {out.get('actionName')}")
print(f"Status: {out.get('status')}")

# For HTTP actions, the real error is in outputs.body
if isinstance(out.get("outputs"), dict):
    status_code = out["outputs"].get("statusCode")
    body = out["outputs"].get("body", {})
    print(f"HTTP {status_code}")
    print(json.dumps(body, indent=2)[:500])

    # Error bodies are often nested JSON strings — parse them
    if isinstance(body, dict) and "error" in body:
        err_detail = body["error"]
        if isinstance(err_detail, str):
            err_detail = json.loads(err_detail)
        print(f"Error: {err_detail.get('message', err_detail)}")

# For expression errors, the error is in the error field
if out.get("error"):
    print(f"Error: {out['error']}")

# Also check inputs — they show what expression/URL/body was used
if out.get("inputs"):
    print(f"Inputs: {json.dumps(out['inputs'], indent=2)[:500]}")
```### アクションの出力で明らかになること (エラー コードでは明らかにされないこと)

| `get_live_flow_run_error` からのエラー コード | `get_live_flow_run_action_outputs` が明らかにすること |
|---|---|
| `ActionFailed` |実際に失敗したネストされたアクションとその HTTP 応答 |
| `NotSpecified` |実際のエラーを含む HTTP ステータス コード + 応答本文 |
| `InternalServerError` |サーバーのエラー メッセージ、スタック トレース、または API エラー JSON |
| `InvalidTemplate` |失敗した正確な式と null/間違った型の値 |
| `BadRequest` |送信されたリクエスト本文とサーバーがリクエストを拒否した理由 |

### 例: 500 を返す HTTP アクション```
Error code: "InternalServerError" ← this tells you nothing

Action outputs reveal:
  HTTP 500
  body: {"error": "Cannot read properties of undefined (reading 'toLowerCase')
    at getClientParamsFromConnectionString (storage.js:20)"}
  ← THIS tells you the Azure Function crashed because a connection string is undefined
```### 例: null の式エラー```
Error code: "BadRequest" ← generic

Action outputs reveal:
  inputs: "body('HTTP_GetTokenFromStore')?['token']?['access_token']"
  outputs: ""   ← empty string, the path resolved to null
  ← THIS tells you the response shape changed — token is at body.access_token, not body.token.access_token
```---

## ステップ 5 — フロー定義を読む```python
defn = mcp("get_live_flow", environmentName=ENV, flowName=FLOW_ID)
actions = defn["properties"]["definition"]["actions"]
print(list(actions.keys()))
```定義内で失敗したアクションを見つけます。 `inputs` 式を検査する
どのようなデータが期待されているかを理解するためです。

---

## ステップ 6 — 失敗から立ち直る

失敗したアクションの入力が上流のアクションを参照している場合は、それらのアクションを検査します
も。発生源が見つかるまで鎖を後ろ向きに進みます。
悪いデータ:```python
# Inspect multiple actions leading up to the failure
for action_name in [root_action, "Compose_WeekEnd", "HTTP_Get_Data"]:
    result = mcp("get_live_flow_run_action_outputs",
        environmentName=ENV,
        flowName=FLOW_ID,
        runName=RUN_ID,
        actionName=action_name)
    out = result[0] if result else {}
    print(f"\n--- {action_name} ({out.get('status')}) ---")
    print(f"Inputs:  {json.dumps(out.get('inputs', ''), indent=2)[:300]}")
    print(f"Outputs: {json.dumps(out.get('outputs', ''), indent=2)[:300]}")
```> ⚠️ 配列処理アクションからの出力ペイロードは非常に大きくなる可能性があります。
> 印刷する前に必ずスライス (例: `[:500]`) してください。

> **ヒント**: 1 回の呼び出しですべてのアクションを取得するには、`actionName` を省略します。
> これはすべてのアクションの入力/出力を返します - 確信が持てない場合に役立ちます
> どの上流のアクションが不正なデータを生成したか。ただし、120 秒以上のタイムアウトを使用します
> 応答は非常に大きくなる可能性があります。

---

## ステップ 7 — 根本原因を特定する

### 式エラー (例: null の `split`)
エラーに `InvalidTemplate` または関数名が記載されている場合:
1. 定義内のアクションを検索します。
2. 読み込まれた上流のアクション/式を確認します。
3. **上流アクションの出力を検査**して、null または欠落しているフィールドがないかどうかを確認します```python
# Example: action uses split(item()?['Name'], ' ')
# → null Name in the source data
result = mcp("get_live_flow_run_action_outputs", ..., actionName="Compose_Names")
if not result:
    print("No outputs returned for Compose_Names")
    names = []
else:
    names = result[0].get("outputs", {}).get("body") or []
nulls = [x for x in names if x.get("Name") is None]
print(f"{len(nulls)} records with null Name")
```### 間違ったフィールド パス
式 `triggerBody()?['fieldName']` が null を返す → `fieldName` が間違っています。
**トリガー出力を検査**して、実際のフィールド名を確認します。```python
result = mcp("get_live_flow_run_action_outputs", ..., actionName="<trigger-action-name>")
print(json.dumps(result[0].get("outputs"), indent=2)[:500])
```### HTTP アクションがエラーを返す
エラー コードには `InternalServerError` または `NotSpecified` が表示されます — **常に検査してください
アクションは** を出力して、実際の HTTP ステータスと応答本文を取得します。```python
result = mcp("get_live_flow_run_action_outputs", ..., actionName="HTTP_Get_Data")
out = result[0]
print(f"HTTP {out['outputs']['statusCode']}")
print(json.dumps(out['outputs']['body'], indent=2)[:500])
```### 接続/認証の失敗
`ConnectionAuthorizationFailed` を探します。接続所有者は、
フローを実行しているサービス アカウント。 API 経由では修正できません。 PAデザイナーで修正。

---

## ステップ 8 — 修正を適用する

**式/データの問題について**:```python
defn = mcp("get_live_flow", environmentName=ENV, flowName=FLOW_ID)
acts = defn["properties"]["definition"]["actions"]

# Example: fix split on potentially-null Name
acts["Compose_Names"]["inputs"] = \
    "@coalesce(item()?['Name'], 'Unknown')"

conn_refs = defn["properties"]["connectionReferences"]
result = mcp("update_live_flow",
    environmentName=ENV,
    flowName=FLOW_ID,
    definition=defn["properties"]["definition"],
    connectionReferences=conn_refs)

print(result.get("error"))  # None = success
```> ⚠️ `update_live_flow` は常に `error` キーを返します。
> `null` (Python `None`) の値は成功を意味します。

---

## ステップ 9 — 修正を確認する

> **HTTP トリガーだけでなく、あらゆるフローをテストするには `resubmit_live_flow_run` を使用します。**
> `resubmit_live_flow_run` は元のトリガーを使用して以前の実行を再生します
>ペイロード。これは **すべてのトリガー タイプ** で機能します: 繰り返し、SharePoint
> 「アイテムの作成時」、コネクタ Webhook、ボタン トリガー、および HTTP
> トリガー。ユーザーにフローを手動でトリガーするよう依頼する必要はありません。
> 次にスケジュールされた実行を待ちます。
>
> `resubmit` が使用できない唯一のケースは、**まったく新しいフローです。
> 実行されたことがありません** — 再実行する以前の実行がありません。```python
# Resubmit the failed run — works for ANY trigger type
resubmit = mcp("resubmit_live_flow_run",
    environmentName=ENV, flowName=FLOW_ID, runName=RUN_ID)
print(resubmit)   # {"resubmitted": true, "triggerName": "..."}

# Wait ~30 s then check
import time; time.sleep(30)
new_runs = mcp("get_live_flow_runs", environmentName=ENV, flowName=FLOW_ID, top=3)
print(new_runs[0]["status"])   # Succeeded = done
```### 再送信とトリガーをいつ使用するか

|シナリオ |使用 |なぜ |
|---|---|---|
|任意のフローで **修正をテスト** | `resubmit_live_flow_run` |失敗の原因となった正確なトリガー ペイロードを再生します。検証する最良の方法です。
|繰り返し/スケジュールされたフロー | `resubmit_live_flow_run` |他の方法ではオンデマンドでトリガーできません。
| SharePoint / コネクタトリガー | `resubmit_live_flow_run` |実際の SP アイテムを作成しないとトリガーできません |
| **カスタム** テスト ペイロードを使用した HTTP トリガー | `trigger_live_flow` |元の実行とは異なるデータを送信する必要がある場合 |
|真新しいフロー、決して実行しない | `trigger_live_flow` (HTTP のみ) |再送信できる以前の実行は存在しません。

### カスタム ペイロードを使用した HTTP トリガー フローのテスト

`Request` (HTTP) トリガーを含むフローの場合は、`trigger_live_flow` を使用します。
元の実行とは**異なる**ペイロードを送信する必要があります:```python
# First inspect what the trigger expects
schema = mcp("get_live_flow_http_schema",
    environmentName=ENV, flowName=FLOW_ID)
print("Expected body schema:", schema.get("requestSchema"))
print("Response schemas:", schema.get("responseSchemas"))

# Trigger with a test payload
result = mcp("trigger_live_flow",
    environmentName=ENV,
    flowName=FLOW_ID,
    body={"name": "Test User", "value": 42})
print(f"Status: {result['responseStatus']}, Body: {result.get('responseBody')}")
```> `trigger_live_flow` は、AAD 認証されたトリガーを自動的に処理します。
> `Request` (HTTP) トリガー タイプのフローでのみ機能します。

---

## クイックリファレンス診断決定ツリー

|症状 |最初のツール |その後、常に | に電話してください。何を探すか |
|---|---|---|---|
|フローは失敗と表示されます | `get_live_flow_run_error` |失敗したアクションに関する `get_live_flow_run_action_outputs` | `outputs` の HTTP ステータス + 応答本文 |
|エラー コードは一般的なものです (`ActionFailed`、`NotSpecified`) | — | `get_live_flow_run_action_outputs` | `outputs.body` には、実際のエラー メッセージ、スタック トレース、または API エラーが含まれています。
| HTTP アクションは 500 を返します | — | `get_live_flow_run_action_outputs` | `outputs.statusCode` + `outputs.body` サーバー エラーの詳細 |
|式のクラッシュ | — | `get_live_flow_run_action_outputs` 前のアクションについて |出力本文内の null / 間違った型のフィールド |
|流れが始まらない | `get_live_flow` | — | check `properties.state` = "開始" |
|アクションが間違ったデータを返す | `get_live_flow_run_action_outputs` | — |実際の出力本体と期待値 |
|修正は適用されましたが、依然として失敗します | `get_live_flow_runs` 再送信後 | — |新しい実行 `status` フィールド |

> **ルール: エラー コードだけから診断しないでください。** `get_live_flow_run_error`
> 失敗したアクションを特定します。 `get_live_flow_run_action_outputs` が明らかにします
>本当の原因。常に両方に電話してください。

---

## 参照ファイル

- [common-errors.md](references/common-errors.md) — エラー コード、考えられる原因、および修正
- [debug-workflow.md](references/debug-workflow.md) — 複雑な障害に対する完全なデシジョン ツリー

## 関連スキル

- `flowstudio-power-automate-mcp` — コア接続の設定と操作のリファレンス
- `flowstudio-power-automate-build` — 新しいフローの構築とデプロイ