---
name: flowstudio-power-automate-mcp
description: >-
  Give your AI agent the same visibility you have in the Power Automate portal — plus
  a bit more. The Graph API only returns top-level run status. Flow Studio MCP exposes
  action-level inputs, outputs, loop iterations, and nested child flow failures.
  Use when asked to: list flows, read a flow definition, check run history, inspect
  action outputs, resubmit a run, cancel a running flow, view connections, get a
  trigger URL, validate a definition, monitor flow health, or any task that requires
  talking to the Power Automate API through an MCP tool. Also use for Power Platform
  environment discovery and connection management. Requires a FlowStudio MCP
  subscription or compatible server — see https://mcp.flowstudio.app
metadata:
  openclaw:
    requires:
      env:
        - FLOWSTUDIO_MCP_TOKEN
    primaryEnv: FLOWSTUDIO_MCP_TOKEN
    homepage: https://mcp.flowstudio.app
---
# FlowStudio MCP を介した Power Automate

このスキルにより、AI エージェントは Microsoft Power Automate を読み取り、監視し、操作できるようになります。
クラウド フローは **FlowStudio MCP サーバー** を介してプログラム的にフローします。ブラウザーは必要ありません。
UI も手動の手順もありません。

> **実際のデバッグ例**: [子フローの式エラー](https://github.com/ninihen1/power-automate-mcp-skills/blob/main/examples/fix-expression-error.md) |
> [フローのバグではなくデータ入力](https://github.com/ninihen1/power-automate-mcp-skills/blob/main/examples/data-not-flow.md) |
> [Null 値により子フローがクラッシュする](https://github.com/ninihen1/power-automate-mcp-skills/blob/main/examples/null-child-flow.md)

> **必要なもの:** [FlowStudio](https://mcp.flowstudio.app) MCP サブスクリプション (または
> 互換性のある Power Automate MCP サーバー)。必要なものは次のとおりです。
> - MCP エンドポイント: `https://mcp.flowstudio.app/mcp` (すべてのサブスクライバで同じ)
> - API キー / JWT トークン (`x-api-key` ヘッダー — ベアラーではありません)
> - Power Platform 環境名 (例: `Default-<tenant-guid>`)

---

## 真実の情報源

|優先順位 |出典 |カバー |
|----------|----------|----------|
| 1 | **実際の API レスポンス** |サーバーが実際に返すものを常に信頼してください。
| 2 | **@@コード3@@** |ツール名、パラメータ名、タイプ、必要なフラグ |
| 3 | **SKILL ドキュメントとリファレンス ファイル** |応答形状、動作メモ、ワークフロー レシピ |

> **新しいセッションはすべて `tools/list`.** で開始します
> すべてのツールの信頼できる最新のスキーマ (パラメーター名、
> タイプと必要なフラグ。 SKILL ドキュメントでは、`tools/list` が教えてくれないことについて説明しています。
> 応答形状、非自明な動作、エンドツーエンドのワークフロー パターン。
>
> `tools/list` または実際の API 応答と一致しないドキュメントがある場合は、
> API が勝ちます。

---

## 推奨言語: Python または Node.js

このスキルと関連するビルド/デバッグ スキルのすべての例では **Python を使用します
`urllib.request`** を使用します (stdlib — `pip install` は必要ありません)。 **Node.js** は
同様に有効な選択肢: `fetch` は Node 18 以降から組み込まれており、JSON 処理は
ネイティブであり、非同期/待機モデルは要求と応答のパターンにきれいにマップされます。
の MCP ツール呼び出し - すでに作業しているチームに自然に適合します。
JavaScript/TypeScript スタック。

|言語 |評決 |メモ |
|---|---|---|
| **Python** | ✅ おすすめ |クリーンな JSON 処理、エスケープの問題なし、すべてのスキル例でそれを使用 |
| **Node.js (≥ 18)** | ✅ おすすめ |ネイティブ `fetch` + `JSON.stringify`/`JSON.parse`; async/await は MCP 呼び出しパターンによく適合します。追加のパッケージは必要ありません |
|パワーシェル | ⚠️ フロー操作では避ける | `ConvertTo-Json -Depth` は、ネストされた定義を暗黙的に切り捨てます。引用符を付けてエスケープすると、複雑なペイロードが壊れます。 `tools/list` の簡単な検出呼び出しには使用できますが、フローの構築または更新には使用できません。 |
| cURL / バッシュ | ⚠️ 可能だが壊れやすい |シェルエスケープのネストされた JSON はエラーが発生しやすくなります。ネイティブ JSON パーサーはありません |

> **TL;DR — 以下のコア MCP ヘルパー (Python または Node.js) を使用します。** 両方のハンドル
> 単一の再利用可能な関数での JSON-RPC フレーミング、認証、および応答の解析。

---

## あなたにできることFlowStudio MCP には 2 つのアクセス層があります。 **FlowStudio for Teams** サブスクライバーは次の特典を獲得できます
高速な Azure テーブル ストア (キャッシュされたスナップショット データ + ガバナンス メタデータ) と
完全なライブ Power Automate API アクセス。 **MCP のみのサブスクライバー** はライブ ツールを入手できます —
フローの構築、デバッグ、操作には十分です。

### ライブ ツール — すべての MCP 加入者が利用可能

|ツール |何をするのか |
|---|---|
| `list_live_flows` | PA API から直接環境内のフローを一覧表示します (常に最新) |
| `list_live_environments` |サービス アカウントに表示されるすべての Power Platform 環境を一覧表示します。
| `list_live_connections` | PA API から環境内のすべての接続をリストする |
| `get_live_flow` |完全なフロー定義 (トリガー、アクション、パラメーター) を取得します。
| `get_live_flow_http_schema` | HTTP によってトリガーされるフローの JSON 本文スキーマと応答スキーマを検査する |
| `get_live_flow_trigger_url` | HTTP によってトリガーされるフローの現在の署名付きコールバック URL を取得します。
| `trigger_live_flow` | HTTP によってトリガーされるフローのコールバック URL への POST (AAD 認証は自動的に処理されます) |
| `update_live_flow` | 1 回の呼び出しで新しいフローを作成するか、既存の定義にパッチを適用します。
| `add_live_flow_to_solution` |非ソリューション フローをソリューションに移行する |
| `get_live_flow_runs` |最近の実行履歴をステータス、開始/終了時刻、エラーとともに一覧表示します。
| `get_live_flow_run_error` |失敗した実行の構造化されたエラーの詳細 (アクションごと) を取得する |
| `get_live_flow_run_action_outputs` |実行中の任意のアクション (またはすべての foreach 反復) の入力/出力を検査します。
| `resubmit_live_flow_run` |元のトリガー ペイロードを使用して、失敗した実行またはキャンセルされた実行を再実行します。
| `cancel_live_flow_run` |現在実行中のフロー実行をキャンセルする |

### ストア ツール — Teams サブスクライバー専用の FlowStudio

これらのツールは、FlowStudio Azure テーブル (監視対象テーブル) から読み取り (および書き込み) します。
ガバナンス メタデータと実行統計で強化されたテナントのフローのスナップショット。

|ツール |何をするのか |
|---|---|
| `list_store_flows` |ガバナンス フラグ、実行失敗率、所有者のメタデータを使用してキャッシュからフローを検索します。
| `get_store_flow` |実行統計やガバナンス フィールドを含む、単一フローのキャッシュされた完全な詳細を取得します。
| `get_store_flow_trigger_url` |キャッシュからトリガー URL を取得します (即時、PA API 呼び出しなし)。
| `get_store_flow_runs` |過去 N 日間のキャッシュされた実行履歴 (期間と修復のヒント付き) |
| `get_store_flow_errors` |キャッシュされた失敗時のみの実行。失敗したアクション名と修復ヒントが含まれます。
| `get_store_flow_summary` |集計された統計: 成功率、失敗回数、平均/最大継続時間 |
| `set_store_flow_state` | PA API 経由でフローを開始または停止し、結果をストアに同期します。
| `update_store_flow` |ガバナンス メタデータの更新 (説明、タグ、監視フラグ、通知ルール、ビジネスへの影響) |
| `list_store_environments` |キャッシュからすべての環境をリストする |
| `list_store_makers` |キャッシュからすべてのメーカー (シチズン開発者) をリストします。
| `get_store_maker` |メーカーのフロー/アプリ数とアカウントのステータスを取得する |
| `list_store_power_apps` |キャッシュからすべての Power Apps キャンバス アプリを一覧表示する |
| `list_store_connections` |キャッシュからすべての Power Platform 接続を一覧表示する |

---

## 最初に呼び出すツール層|タスク |ツール |メモ |
|---|---|---|
|フローのリスト | `list_live_flows` |常に最新 — PA API を直接呼び出します。
|定義を読む | `get_live_flow` |常にライブでフェッチされます - キャッシュされません |
|障害をデバッグする | `get_live_flow_runs` → `get_live_flow_run_error` |ライブ実行データを使用する |

> ⚠️ **`list_live_flows` は、`flows` 配列を含むラッパー オブジェクト**を返します — `result["flows"]` 経由でアクセスします。

> **FlowStudio for Teams** サブスクライバーはストア ツール (`list_store_flows`、`get_store_flow` など) を利用でき、キャッシュされたガバナンス メタデータを提供します。疑わしい場合はライブ ツールを使用してください。ライブ ツールはすべてのサブスクリプション層で機能します。

---

## ステップ 0 — 利用可能なツールを見つける

必ず `tools/list` を呼び出してサーバーにアクセスできることを確認してから始めてください。
使用可能なツール名を正確に示します (名前はサーバーのバージョンによって異なる場合があります)。```python
import json, urllib.request

TOKEN = "<YOUR_JWT_TOKEN>"
MCP   = "https://mcp.flowstudio.app/mcp"

def mcp_raw(method, params=None, cid=1):
    payload = {"jsonrpc": "2.0", "method": method, "id": cid}
    if params:
        payload["params"] = params
    req = urllib.request.Request(MCP, data=json.dumps(payload).encode(),
        headers={"x-api-key": TOKEN, "Content-Type": "application/json",
                 "User-Agent": "FlowStudio-MCP/1.0"})
    try:
        resp = urllib.request.urlopen(req, timeout=30)
    except urllib.error.HTTPError as e:
        raise RuntimeError(f"MCP HTTP {e.code} — check token and endpoint") from e
    return json.loads(resp.read())

raw = mcp_raw("tools/list")
if "error" in raw:
    print("ERROR:", raw["error"]); raise SystemExit(1)
for t in raw["result"]["tools"]:
    print(t["name"], "—", t["description"][:60])
```---

## コア MCP ヘルパー (Python)

後続のすべての操作を通じてこのヘルパーを使用します。```python
import json, urllib.request

TOKEN = "<YOUR_JWT_TOKEN>"
MCP   = "https://mcp.flowstudio.app/mcp"

def mcp(tool, args, cid=1):
    payload = {"jsonrpc": "2.0", "method": "tools/call", "id": cid,
               "params": {"name": tool, "arguments": args}}
    req = urllib.request.Request(MCP, data=json.dumps(payload).encode(),
        headers={"x-api-key": TOKEN, "Content-Type": "application/json",
                 "User-Agent": "FlowStudio-MCP/1.0"})
    try:
        resp = urllib.request.urlopen(req, timeout=120)
    except urllib.error.HTTPError as e:
        body = e.read().decode("utf-8", errors="replace")
        raise RuntimeError(f"MCP HTTP {e.code}: {body[:200]}") from e
    raw = json.loads(resp.read())
    if "error" in raw:
        raise RuntimeError(f"MCP error: {json.dumps(raw['error'])}")
    text = raw["result"]["content"][0]["text"]
    return json.loads(text)
```> **一般的な認証エラー:**
> - HTTP 401/403 → トークンが見つからないか、有効期限が切れているか、形式が不正です。 [mcp.flowstudio.app](https://mcp.flowstudio.app) から新しい JWT を取得します。
> - HTTP 400 → 不正な形式の JSON-RPC ペイロード。 `Content-Type: application/json` と本文の構造を確認してください。
> - `MCP error: {"code": -32602, ...}` → ツール引数が間違っているか欠落しています。

---

## コア MCP ヘルパー (Node.js)

Node.js 18 以降の同等のヘルパー (組み込み `fetch` — パッケージは必要ありません):```js
const TOKEN = "<YOUR_JWT_TOKEN>";
const MCP   = "https://mcp.flowstudio.app/mcp";

async function mcp(tool, args, cid = 1) {
  const payload = {
    jsonrpc: "2.0",
    method: "tools/call",
    id: cid,
    params: { name: tool, arguments: args },
  };
  const res = await fetch(MCP, {
    method: "POST",
    headers: {
      "x-api-key": TOKEN,
      "Content-Type": "application/json",
      "User-Agent": "FlowStudio-MCP/1.0",
    },
    body: JSON.stringify(payload),
  });
  if (!res.ok) {
    const body = await res.text();
    throw new Error(`MCP HTTP ${res.status}: ${body.slice(0, 200)}`);
  }
  const raw = await res.json();
  if (raw.error) throw new Error(`MCP error: ${JSON.stringify(raw.error)}`);
  return JSON.parse(raw.result.content[0].text);
}
```> Node.js 18 以降が必要です。古いノードの場合は、`fetch` を `https.request` に置き換えます。
> stdlib からダウンロードするか、`node-fetch` をインストールします。

---

## フローのリストを表示する```python
ENV = "Default-<tenant-guid>"

result = mcp("list_live_flows", {"environmentName": ENV})
# Returns wrapper object:
# {"mode": "owner", "flows": [{"id": "0757041a-...", "displayName": "My Flow",
#   "state": "Started", "triggerType": "Request", ...}], "totalCount": 42, "error": null}
for f in result["flows"]:
    FLOW_ID = f["id"]   # plain UUID — use directly as flowName
    print(FLOW_ID, "|", f["displayName"], "|", f["state"])
```---

## フロー定義を読み取る```python
FLOW = "<flow-uuid>"

flow = mcp("get_live_flow", {"environmentName": ENV, "flowName": FLOW})

# Display name and state
print(flow["properties"]["displayName"])
print(flow["properties"]["state"])

# List all action names
actions = flow["properties"]["definition"]["actions"]
print("Actions:", list(actions.keys()))

# Inspect one action's expression
print(actions["Compose_Filter"]["inputs"])
```---

## 実行履歴を確認する```python
# Most recent runs (newest first)
runs = mcp("get_live_flow_runs", {"environmentName": ENV, "flowName": FLOW, "top": 5})
# Returns direct array:
# [{"name": "08584296068667933411438594643CU15",
#   "status": "Failed",
#   "startTime": "2026-02-25T06:13:38.6910688Z",
#   "endTime": "2026-02-25T06:15:24.1995008Z",
#   "triggerName": "manual",
#   "error": {"code": "ActionFailed", "message": "An action failed..."}},
#  {"name": "08584296028664130474944675379CU26",
#   "status": "Succeeded", "error": null, ...}]

for r in runs:
    print(r["name"], r["status"])

# Get the name of the first failed run
run_id = next((r["name"] for r in runs if r["status"] == "Failed"), None)
```---

## アクションの出力を検査する```python
run_id = runs[0]["name"]

out = mcp("get_live_flow_run_action_outputs", {
    "environmentName": ENV,
    "flowName": FLOW,
    "runName": run_id,
    "actionName": "Get_Customer_Record"   # exact action name from the definition
})
print(json.dumps(out, indent=2))
```---

## 実行エラーを取得する```python
err = mcp("get_live_flow_run_error", {
    "environmentName": ENV,
    "flowName": FLOW,
    "runName": run_id
})
# Returns:
# {"runName": "08584296068...",
#  "failedActions": [
#    {"actionName": "HTTP_find_AD_User_by_Name", "status": "Failed",
#     "code": "NotSpecified", "startTime": "...", "endTime": "..."},
#    {"actionName": "Scope_prepare_workers", "status": "Failed",
#     "error": {"code": "ActionFailed", "message": "An action failed..."}}
#  ],
#  "allActions": [
#    {"actionName": "Apply_to_each", "status": "Skipped"},
#    {"actionName": "Compose_WeekEnd", "status": "Succeeded"},
#    ...
#  ]}

# The ROOT cause is usually the deepest entry in failedActions:
root = err["failedActions"][-1]
print(f"Root failure: {root['actionName']} → {root['code']}")
```---

## 実行を再送信する```python
result = mcp("resubmit_live_flow_run", {
    "environmentName": ENV,
    "flowName": FLOW,
    "runName": run_id
})
print(result)   # {"resubmitted": true, "triggerName": "..."}
```---

## 実行中の実行をキャンセルする```python
mcp("cancel_live_flow_run", {
    "environmentName": ENV,
    "flowName": FLOW,
    "runName": run_id
})
```> ⚠️ **`Running` が表示される実行はキャンセルしないでください。
> アダプティブ カード レスポンス。** そのステータスは正常です。フローは待機中です。
> 人間が Teams で応答するため。キャンセルすると保留中のカードは破棄されます。

---

## 完全なラウンドトリップの例 — 失敗したフローのデバッグと修正```python
# ── 1. Find the flow ─────────────────────────────────────────────────────
result = mcp("list_live_flows", {"environmentName": ENV})
target = next(f for f in result["flows"] if "My Flow Name" in f["displayName"])
FLOW_ID = target["id"]

# ── 2. Get the most recent failed run ────────────────────────────────────
runs = mcp("get_live_flow_runs", {"environmentName": ENV, "flowName": FLOW_ID, "top": 5})
# [{"name": "08584296068...", "status": "Failed", ...}, ...]
RUN_ID = next(r["name"] for r in runs if r["status"] == "Failed")

# ── 3. Get per-action failure breakdown ──────────────────────────────────
err = mcp("get_live_flow_run_error", {"environmentName": ENV, "flowName": FLOW_ID, "runName": RUN_ID})
# {"failedActions": [{"actionName": "HTTP_find_AD_User_by_Name", "code": "NotSpecified",...}], ...}
root_action = err["failedActions"][-1]["actionName"]
print(f"Root failure: {root_action}")

# ── 4. Read the definition and inspect the failing action's expression ───
defn = mcp("get_live_flow", {"environmentName": ENV, "flowName": FLOW_ID})
acts = defn["properties"]["definition"]["actions"]
print("Failing action inputs:", acts[root_action]["inputs"])

# ── 5. Inspect the prior action's output to find the null ────────────────
out = mcp("get_live_flow_run_action_outputs", {
    "environmentName": ENV, "flowName": FLOW_ID,
    "runName": RUN_ID, "actionName": "Compose_Names"
})
nulls = [x for x in out.get("body", []) if x.get("Name") is None]
print(f"{len(nulls)} records with null Name")

# ── 6. Apply the fix ─────────────────────────────────────────────────────
acts[root_action]["inputs"]["parameters"]["searchName"] = \
    "@coalesce(item()?['Name'], '')"

conn_refs = defn["properties"]["connectionReferences"]
result = mcp("update_live_flow", {
    "environmentName": ENV, "flowName": FLOW_ID,
    "definition": defn["properties"]["definition"],
    "connectionReferences": conn_refs
})
assert result.get("error") is None, f"Deploy failed: {result['error']}"
# ⚠️ error key is always present — only fail if it is NOT None

# ── 7. Resubmit and verify ───────────────────────────────────────────────
mcp("resubmit_live_flow_run", {"environmentName": ENV, "flowName": FLOW_ID, "runName": RUN_ID})

import time; time.sleep(30)
new_runs = mcp("get_live_flow_runs", {"environmentName": ENV, "flowName": FLOW_ID, "top": 1})
print(new_runs[0]["status"])   # Succeeded = done
```---

## 認証と接続に関する注意事項

|フィールド |値 |
|---|---|
|認証ヘッダー | `x-api-key: <JWT>` — **違います** `Authorization: Bearer` |
|トークンの形式 |プレーン JWT — 削除、変更、プレフィックスを付けないでください。
|タイムアウト | `get_live_flow_run_action_outputs` (大きな出力) には 120 秒以上を使用します。
|環境名 | `Default-<tenant-guid>` (`list_live_environments` または `list_live_flows` 応答で検索) |

---

## 参照ファイル

- [MCP-BOOTSTRAP.md](references/MCP-BOOTSTRAP.md) — エンドポイント、認証、リクエスト/レスポンスの形式 (最初にお読みください)
- [tool-reference.md](references/tool-reference.md) — 応答形状と動作メモ (パラメータは `tools/list` にあります)
- [action-types.md](references/action-types.md) — Power Automate アクション タイプ パターン
- [connection-references.md](references/connection-references.md) — コネクタ リファレンス ガイド

---

## さらなる機能

**失敗したフローを診断**する場合、エンドツーエンド → `flowstudio-power-automate-debug` スキルをロードします。

**新しいフローの構築とデプロイ**の場合 → `flowstudio-power-automate-build` スキルをロードします。