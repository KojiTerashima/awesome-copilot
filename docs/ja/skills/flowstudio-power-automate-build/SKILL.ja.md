---
name: flowstudio-power-automate-build
description: >-
  Build, scaffold, and deploy Power Automate cloud flows using the FlowStudio
  MCP server. Your agent constructs flow definitions, wires connections, deploys,
  and tests — all via MCP without opening the portal.
  Load this skill when asked to: create a flow, build a new flow,
  deploy a flow definition, scaffold a Power Automate workflow, construct a flow
  JSON, update an existing flow's actions, patch a flow definition, add actions
  to a flow, wire up connections, or generate a workflow definition from scratch.
  Requires a FlowStudio MCP subscription — see https://mcp.flowstudio.app
metadata:
  openclaw:
    requires:
      env:
        - FLOWSTUDIO_MCP_TOKEN
    primaryEnv: FLOWSTUDIO_MCP_TOKEN
    homepage: https://mcp.flowstudio.app
---
# FlowStudio MCP を使用した Power Automate フローの構築とデプロイ

Power Automate クラウド フローを構築およびデプロイするためのステップバイステップ ガイド
FlowStudio MCP サーバー経由でプログラム的に。

**前提条件**: FlowStudio MCP サーバーは有効な JWT でアクセス可能である必要があります。
接続セットアップについては、`flowstudio-power-automate-mcp` スキルを参照してください。  
https://mcp.flowstudio.app で購読してください

---

## 真実の情報源

> **必ず最初に `tools/list` に電話して**、利用可能なツール名とそのツールを確認してください
> パラメータスキーマ。ツールの名前とパラメータはサーバーのバージョン間で異なる場合があります。
> このスキルは、応答形状、行動メモ、構築パターンをカバーします —
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

ENV = "<environment-id>"  # e.g. Default-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```---

## ステップ 1 — 安全性チェック: フローはすでに存在しますか?

重複を避けるために、ビルドする前に必ず確認してください。```python
results = mcp("list_live_flows", environmentName=ENV)

# list_live_flows returns { "flows": [...] }
matches = [f for f in results["flows"]
           if "My New Flow".lower() in f["displayName"].lower()]

if len(matches) > 0:
    # Flow exists — modify rather than create
    FLOW_ID = matches[0]["id"]   # plain UUID from list_live_flows
    print(f"Existing flow: {FLOW_ID}")
    defn = mcp("get_live_flow", environmentName=ENV, flowName=FLOW_ID)
else:
    print("Flow not found — building from scratch")
    FLOW_ID = None
```---

## ステップ 2 — 接続参照の取得

すべてのコネクタ アクションには、キーを指す `connectionName` が必要です。
フローの `connectionReferences` マップ。そのキーは認証された接続にリンクします
環境の中で。

> **必須**: 最初に `list_live_connections` に電話する必要があります。質問しないでください。
> 接続名または GUID のユーザー。 API は必要な正確な値を返します。
> 必要な接続が欠落していることを API が確認した場合にのみ、ユーザーにプロンプ​​トを表示します。

### 2a — 常に最初に `list_live_connections` を呼び出します```python
conns = mcp("list_live_connections", environmentName=ENV)

# Filter to connected (authenticated) connections only
active = [c for c in conns["connections"]
          if c["statuses"][0]["status"] == "Connected"]

# Build a lookup: connectorName → connectionName (id)
conn_map = {}
for c in active:
    conn_map[c["connectorName"]] = c["id"]

print(f"Found {len(active)} active connections")
print("Available connectors:", list(conn_map.keys()))
```### 2b — フローに必要なコネクタを決定する

構築しているフローに基づいて、どのコネクタが必要かを特定します。
一般的なコネクタ API 名:

|コネクタ | API名 |
|---|---|
|シェアポイント | `shared_sharepointonline` |
| Outlook / Office 365 | `shared_office365` |
|チーム | `shared_teams` |
|承認 | `shared_approvals` |
|ビジネス向け OneDrive | `shared_onedriveforbusiness` |
| Excel Online (ビジネス) | `shared_excelonlinebusiness` |
|データバース | `shared_commondataserviceforapps` |
| Microsoft フォーム | `shared_microsoftforms` |

> **接続を必要としないフロー** (例: Recurrence + Compose + HTTP のみ)
> ステップ 2 の残りをスキップできます。デプロイ呼び出しから `connectionReferences` を省略します。

### 2c — 接続が見つからない場合は、ユーザーをガイドします```python
connectors_needed = ["shared_sharepointonline", "shared_office365"]  # adjust per flow

missing = [c for c in connectors_needed if c not in conn_map]

if not missing:
    print("✅ All required connections are available — proceeding to build")
else:
    # ── STOP: connections must be created interactively ──
    # Connections require OAuth consent in a browser — no API can create them.
    print("⚠️  The following connectors have no active connection in this environment:")
    for c in missing:
        friendly = c.replace("shared_", "").replace("onlinebusiness", " Online (Business)")
        print(f"   • {friendly}  (API name: {c})")
    print()
    print("Please create the missing connections:")
    print("  1. Open https://make.powerautomate.com/connections")
    print("  2. Select the correct environment from the top-right picker")
    print("  3. Click '+ New connection' for each missing connector listed above")
    print("  4. Sign in and authorize when prompted")
    print("  5. Tell me when done — I will re-check and continue building")
    # DO NOT proceed to Step 3 until the user confirms.
    # After user confirms, re-run Step 2a to refresh conn_map.
```### 2d — connectionReferences ブロックを構築する

2c でコネクタが欠落していないことを確認した後でのみ、これを実行します。```python
connection_references = {}
for connector in connectors_needed:
    connection_references[connector] = {
        "connectionName": conn_map[connector],   # the GUID from list_live_connections
        "source": "Invoker",
        "id": f"/providers/Microsoft.PowerApps/apis/{connector}"
    }
```> **重要 — `host.connectionName` アクション**: でアクションを構築する場合
> ステップ 3、`host.connectionName` をこのマップの **key** に設定します (例:
> `shared_teams`)、接続 GUID ではありません。 GUID は内部にのみ入ります。
> `connectionReferences` エントリ。エンジンはアクションと一致します
> `host.connectionName` をキーに入力して、正しい接続を見つけます。

> **代替** — 同じコネクタを使用するフローがすでにある場合は、
> `connectionReferences` をその定義から抽出できます。
>```python
> ref_flow = mcp("get_live_flow", environmentName=ENV, flowName="<existing-flow-id>")
> connection_references = ref_flow["properties"]["connectionReferences"]
> ````flowstudio-power-automate-mcp` スキルの **connection-references.md** リファレンスを参照してください。
完全な接続参照構造については。

---

## ステップ 3 — フロー定義を構築する

定義オブジェクトを構築します。 [flow-schema.md](references/flow-schema.md) を参照してください。
完全なスキーマと、コピー＆ペースト テンプレートのアクション パターンのリファレンスについては、次のとおりです。
- [action-patterns-core.md](references/action-patterns-core.md) — 変数、制御フロー、式
- [action-patterns-data.md](references/action-patterns-data.md) — 配列変換、HTTP、解析
- [action-patterns-connectors.md](references/action-patterns-connectors.md) — SharePoint、Outlook、Teams、承認```python
definition = {
    "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
    "contentVersion": "1.0.0.0",
    "triggers": { ... },   # see trigger-types.md / build-patterns.md
    "actions": { ... }     # see ACTION-PATTERNS-*.md / build-patterns.md
}
```> すぐに使用できる完全なものについては、[build-patterns.md](references/build-patterns.md) を参照してください。
> Recurrence+SharePoint+Teams、HTTP トリガーなどをカバーするフロー定義。

---

## ステップ 4 — デプロイ (作成または更新)

`update_live_flow` は、作成と更新の両方を 1 つのツールで処理します。

### 新しいフローを作成します (既存のフローはありません)

`flowName` を省略します — サーバーは新しい GUID を生成し、PUT 経由で作成します。```python
result = mcp("update_live_flow",
    environmentName=ENV,
    # flowName omitted → creates a new flow
    definition=definition,
    connectionReferences=connection_references,
    displayName="Overdue Invoice Notifications",
    description="Weekly SharePoint → Teams notification flow, built by agent"
)

if result.get("error") is not None:
    print("Create failed:", result["error"])
else:
    # Capture the new flow ID for subsequent steps
    FLOW_ID = result["created"]
    print(f"✅ Flow created: {FLOW_ID}")
```### 既存のフローを更新する

`flowName` を PATCH に指定します。```python
result = mcp("update_live_flow",
    environmentName=ENV,
    flowName=FLOW_ID,
    definition=definition,
    connectionReferences=connection_references,
    displayName="My Updated Flow",
    description="Updated by agent on " + __import__('datetime').datetime.utcnow().isoformat()
)

if result.get("error") is not None:
    print("Update failed:", result["error"])
else:
    print("Update succeeded:", result)
```> ⚠️ `update_live_flow` は常に `error` キーを返します。
> `null` (Python `None`) は成功を意味します。キーの存在を失敗として扱わないでください。
>
> ⚠️ `description` は作成と更新の両方に必要です。

### 一般的な展開エラー

|エラー メッセージ (含む) |原因 |修正 |
|---|---|---|
| `missing from connectionReferences` |アクションの `host.connectionName` が `connectionReferences` マップに存在しないキーを参照しています。 `host.connectionName` が生の GUID ではなく、`connectionReferences` の **キー** (例: `shared_teams`) を使用していることを確認してください。
| `ConnectionAuthorizationFailed` / 403 |接続 GUID が別のユーザーに属しているか、許可されていません。ステップ 2a を再実行し、現在の `x-api-key` ユーザーが所有する接続を使用します。
| `InvalidTemplate` / `InvalidDefinition` | JSON 定義の構文エラー | `runAfter` チェーン、式の構文、およびアクション タイプのスペルを確認してください。
| `ConnectionNotConfigured` |コネクタ アクションは存在しますが、接続 GUID が無効か期限切れです。 `list_live_connections` を再確認して新しい GUID を確認します。

---

## ステップ 5 — デプロイメントを確認する```python
check = mcp("get_live_flow", environmentName=ENV, flowName=FLOW_ID)

# Confirm state
print("State:", check["properties"]["state"])  # Should be "Started"
# If state is "Stopped", use set_live_flow_state — NOT update_live_flow
# mcp("set_live_flow_state", environmentName=ENV, flowName=FLOW_ID, state="Started")

# Confirm the action we added is there
acts = check["properties"]["definition"]["actions"]
print("Actions:", list(acts.keys()))
```---

## ステップ 6 — フローをテストする

> **必須**: テスト実行を開始する前に、**ユーザーに確認を求めてください**。
> フローの実行には実際の副作用が伴います。メールの送信、Teams メッセージの投稿、
> SharePoint への書き込み、承認の開始、または外部 API の呼び出し。どういうことなのか説明してください
> フローは実行し、`trigger_live_flow` を呼び出す前に明示的な承認を待ちます。
> または `resubmit_live_flow_run`。

### 更新されたフロー (以前の実行がある) - 任意のトリガー タイプ

> **最初に `resubmit_live_flow_run` を使用してください。** すべてのトリガー タイプで機能します —
> 繰り返し、SharePoint、コネクタ Webhook、ボタン、および HTTP。再生します
> 元のトリガー ペイロード。ユーザーに手動でトリガーするよう依頼しないでください。
> 実行するか、次にスケジュールされた実行を待ちます。```python
runs = mcp("get_live_flow_runs", environmentName=ENV, flowName=FLOW_ID, top=1)
if runs:
    # Works for Recurrence, SharePoint, connector triggers — not just HTTP
    result = mcp("resubmit_live_flow_run",
        environmentName=ENV, flowName=FLOW_ID, runName=runs[0]["name"])
    print(result)   # {"resubmitted": true, "triggerName": "..."}
```### HTTP によってトリガーされるフロー — カスタム テスト ペイロード

**異なる**ペイロードを送信する必要がある場合にのみ `trigger_live_flow` を使用してください
本来の走りよりも。修正を確認するには、`resubmit_live_flow_run` を使用します。
失敗の原因となった正確なデータを使用するため、より優れています。```python
schema = mcp("get_live_flow_http_schema",
    environmentName=ENV, flowName=FLOW_ID)
print("Expected body:", schema.get("requestSchema"))

result = mcp("trigger_live_flow",
    environmentName=ENV, flowName=FLOW_ID,
    body={"name": "Test", "value": 1})
print(f"Status: {result['responseStatus']}")
```### 新しい非 HTTP フロー (反復、コネクタ トリガーなど)

新しい繰り返しフローまたはコネクタによってトリガーされるフローには **以前の実行はありません**。
再送信しても呼び出す HTTP エンドポイントがありません。これが唯一のシナリオです。
以下の一時的な HTTP トリガーのアプローチが必要です。 **一時的なものを使用して展開する
最初に HTTP トリガー、アクションをテストしてから、本番トリガーに切り替えます。**

#### 7a — 実際のトリガーを保存し、一時的な HTTP トリガーを使用してデプロイします```python
# Save the production trigger you built in Step 3
production_trigger = definition["triggers"]

# Replace with a temporary HTTP trigger
definition["triggers"] = {
    "manual": {
        "type": "Request",
        "kind": "Http",
        "inputs": {
            "schema": {}
        }
    }
}

# Deploy (create or update) with the temp trigger
result = mcp("update_live_flow",
    environmentName=ENV,
    flowName=FLOW_ID,       # omit if creating new
    definition=definition,
    connectionReferences=connection_references,
    displayName="Overdue Invoice Notifications",
    description="Deployed with temp HTTP trigger for testing")

if result.get("error") is not None:
    print("Deploy failed:", result["error"])
else:
    if not FLOW_ID:
        FLOW_ID = result["created"]
    print(f"✅ Deployed with temp HTTP trigger: {FLOW_ID}")
```#### 7b — フローを起動して結果を確認する```python
# Trigger the flow
test = mcp("trigger_live_flow",
    environmentName=ENV, flowName=FLOW_ID)
print(f"Trigger response status: {test['status']}")

# Wait for the run to complete
import time; time.sleep(15)

# Check the run result
runs = mcp("get_live_flow_runs",
    environmentName=ENV, flowName=FLOW_ID, top=1)
run = runs[0]
print(f"Run {run['name']}: {run['status']}")

if run["status"] == "Failed":
    err = mcp("get_live_flow_run_error",
        environmentName=ENV, flowName=FLOW_ID, runName=run["name"])
    root = err["failedActions"][-1]
    print(f"Root cause: {root['actionName']} → {root.get('code')}")
    # Debug and fix the definition before proceeding
    # See flowstudio-power-automate-debug skill for full diagnosis workflow
```#### 7c — 本番トリガーに切り替える

テストの実行が成功したら、一時的な HTTP トリガーを実際のトリガーに置き換えます。```python
# Restore the production trigger
definition["triggers"] = production_trigger

result = mcp("update_live_flow",
    environmentName=ENV,
    flowName=FLOW_ID,
    definition=definition,
    connectionReferences=connection_references,
    description="Swapped to production trigger after successful test")

if result.get("error") is not None:
    print("Trigger swap failed:", result["error"])
else:
    print("✅ Production trigger deployed — flow is live")
```> **これが機能する理由**: トリガーは単なるエントリ ポイントであり、アクションは次のとおりです。
> フローの開始方法に関係なく、同一です。 HTTPトリガーによるテスト
> Compose、SharePoint、Teams などの同じアクションをすべて実行します。
>
> **コネクタ トリガー** (例: 「SharePoint でアイテムが作成されたとき」):
> アクションが `triggerBody()` または `triggerOutputs()` を参照する場合、
> `trigger_live_flow` の `body` パラメータの代表的なテスト ペイロード
> コネクタ トリガーが生成する形状と一致します。

---

## 落とし穴

|間違い |結果 |予防 |
|---|---|---|
|デプロイに `connectionReferences` がありません | 400 "供給接続参照" |常に最初に `list_live_connections` を呼び出します。
| `"operationOptions"` が Foreach にありません |並列実行、書き込み時の競合状態 |常に `"Sequential"` を追加します。
| `union(old_data, new_data)` |古い値が新しい値をオーバーライドします (先者勝ち) | `union(new_data, old_data)` を使用してください。
| NULL の可能性がある文字列に対する `split()` | `InvalidTemplate` クラッシュ | `coalesce(field, '')` で囲む |
| `result["error"]` が存在することを確認しています |常に存在します。本当のエラーは `!= null` | `result.get("error") is not None` を使用します。
|フローはデプロイされましたが、状態は「停止」です。フローがスケジュール通りに実行されない | `set_live_flow_state` を `state: "Started"` とともに呼び出します — 状態の変更には `update_live_flow` を使用しないでください。
| Teams の「フロー ボットとチャット」受信者をオブジェクトとして | 400 `GraphUserDetailNotFound` |末尾にセミコロンを付けたプレーン文字列を使用します (以下を参照)。

### チーム `PostMessageToConversation` — 受信者の形式

`body/recipient` パラメータの形式は、`location` 値によって異なります。

|場所 | `body/recipient` 形式 |例 |
|---|---|---|
| **Flow ボットとチャット** | **末尾にセミコロン**を含むプレーンな電子メール文字列 | `"user@contoso.com;"` |
| **チャンネル** | `groupId` および `channelId` を含むオブジェクト | `{"groupId": "...", "channelId": "..."}` |

> **よくある間違い**: 「フロー ボットとのチャット」に `{"to": "user@contoso.com"}` を渡す
> 400 `GraphUserDetailNotFound` エラーが返されます。 API はプレーンな文字列を想定しています。

---

## 参照ファイル

- [flow-schema.md](references/flow-schema.md) — 完全なフロー定義の JSON スキーマ
- [trigger-types.md](references/trigger-types.md) — トリガー タイプ テンプレート
- [action-patterns-core.md](references/action-patterns-core.md) — 変数、制御フロー、式
- [action-patterns-data.md](references/action-patterns-data.md) — 配列変換、HTTP、解析
- [action-patterns-connectors.md](references/action-patterns-connectors.md) — SharePoint、Outlook、Teams、承認
- [build-patterns.md](references/build-patterns.md) — 完全なフロー定義テンプレート (Recurrence+SP+Teams、HTTP トリガー)

## 関連スキル

- `flowstudio-power-automate-mcp` — コア接続のセットアップとツールのリファレンス
- `flowstudio-power-automate-debug` — デプロイ後に失敗したフローをデバッグする