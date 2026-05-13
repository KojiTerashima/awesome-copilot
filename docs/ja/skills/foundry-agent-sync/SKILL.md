---
name: foundry-agent-sync
description: "ローカルの JSON マニフェストから、REST API を介して Azure AI Foundry 内にプロンプトベースの AI エージェントを直接作成・同期します。ローカルコードのみを生成するスキャフォールディング系スキルとは異なり、このスキルは Foundry サービス自体にエージェントを登録するため、すぐに呼び出せる状態になります。ユーザーが Foundry でのエージェント作成、同期、デプロイ、登録、Foundry へのプッシュ、エージェント指示の更新、新規リポジトリ向けマニフェストと同期スクリプトのスキャフォールディングを求める場合に使用します。トリガー: 'create agent in foundry', 'sync foundry agents', 'deploy agents to foundry', 'register agents in foundry', 'push agents', 'create foundry agent manifest', 'scaffold agent sync'."
---

# Foundry Agent Sync

## 概要

Agent Service REST API を使って、Azure AI Foundry 内にプロンプトベースの AI エージェントを直接作成・同期します。このスキルは Foundry サービス自体にエージェントを登録するため、Foundry ポータルまたは API を通じて、すぐに呼び出し・評価・管理できるようになります。各エージェントは、ローカル JSON マニフェストの定義を使い、名前付き POST 呼び出しで冪等に作成または更新されます。

> **重要な違い:** このスキルは AI Foundry 内（サーバー側）にエージェントを作成します。ローカルのエージェントコードやコンテナーイメージはスキャフォールディングしません。そちらには `microsoft-foundry` スキルの `create` サブスキルを使用してください。

## 前提条件

ユーザーには以下が必要です。

1. モデル（例: `gpt-5-4`）がデプロイ済みの Azure AI Foundry プロジェクト
2. Foundry プロジェクトにアクセス可能な状態で認証済みの Azure CLI（`az`）
3. Foundry プロジェクトリソースに対する **Azure AI User** ロール（またはそれ以上）

進める前に、以下の値を収集してください。

| 値 | 取得方法 |
|---|---|
| **Foundry project endpoint** | Azure Portal → AI Foundry project → Overview → Endpoint、または `az resource show` |
| **Subscription ID** | `az account show --query id -o tsv` |
| **Model deployment name** | Foundry プロジェクトにデプロイされたモデル名（例: `gpt-5-4`） |

## マニフェスト形式

マニフェストは JSON 配列で、各エントリが 1 つのエージェントを定義します。一般的なパスとして `infra/foundry-agents.json`、`foundry-agents.json`、`.foundry/agents.json` を確認してください。存在しない場合はスキャフォールディングします。

```json
[
  {
    "useCaseId": "alert-triage",
    "description": "このエージェントが行う内容の短い説明。",
    "baseInstruction": "You are an assistant that... <system prompt for the agent>"
  }
]
```

### フィールドリファレンス

| Field | Required | Description |
|---|---|---|
| `useCaseId` | Yes | Kebab-case 識別子。エージェント名（`{prefix}-{useCaseId}`）の生成に使用 |
| `description` | Yes | エージェントのメタデータとして保存される人間可読な説明 |
| `baseInstruction` | Yes | エージェント用のシステムプロンプト / 基本指示 |

## 同期スクリプト

### PowerShell（対話実行 / CI）

同期スクリプトを作成または特定します。標準パスは `infra/scripts/sync-foundry-agents.ps1` ですが、リポジトリ構成に合わせて調整してください。

```powershell
param(
  [Parameter(Mandatory)]
  [string]$SubscriptionId,

  [Parameter(Mandatory)]
  [string]$ProjectEndpoint,

  [string]$ManifestPath = (Join-Path $PSScriptRoot '..\foundry-agents.json'),
  [string]$ModelName = 'gpt-5-4',
  [string]$AgentNamePrefix = 'myproject',
  [string]$ApiVersion = '2025-11-15-preview'
)

$ErrorActionPreference = 'Stop'

# Optional: append a common instruction suffix to every agent
$commonSuffix = ''

az account set --subscription $SubscriptionId | Out-Null
$accessToken = az account get-access-token --resource https://ai.azure.com/ --query accessToken -o tsv
if (-not $accessToken) { throw 'Failed to acquire Foundry access token.' }

$definitions = Get-Content -Raw -Path $ManifestPath | ConvertFrom-Json
$headers = @{ Authorization = "Bearer $accessToken" }
$results = @()

foreach ($def in $definitions) {
  $agentName = "$AgentNamePrefix-$($def.useCaseId)"
  $instructions = if ($commonSuffix) { "$($def.baseInstruction)`n`n$commonSuffix" } else { $def.baseInstruction }
  $body = @{
    definition  = @{ kind = 'prompt'; model = $ModelName; instructions = $instructions }
    description = $def.description
    metadata    = @{ useCaseId = $def.useCaseId; managedBy = 'foundry-agent-sync' }
  } | ConvertTo-Json -Depth 8

  $uri = "$($ProjectEndpoint.TrimEnd('/'))/agents/$agentName`?api-version=$ApiVersion"
  $resp = Invoke-RestMethod -Method Post -Uri $uri -Headers $headers -ContentType 'application/json' -Body $body
  $version = $resp.version ?? $resp.latest_version ?? $resp.id ?? 'unknown'
  Write-Host "Synced $agentName ($version)"
  $results += [pscustomobject]@{ name = $agentName; version = $version }
}

$results | Format-Table -AutoSize
```

### Bash（Bicep デプロイスクリプト / CI）

`Microsoft.Resources/deploymentScripts` による自動デプロイでは、以下を行う bash スクリプトを使用します。

1. マネージド ID で認証: `az login --identity --username "$CLIENT_ID"`
2. Foundry トークンを取得: `az account get-access-token --resource https://ai.azure.com/`
3. `FOUNDRY_AGENT_DEFINITIONS` 環境変数（JSON 文字列）から定義を反復処理
4. 各エージェントを `{endpoint}/agents/{name}?api-version=2025-11-15-preview` に POST

## Bicep 統合（任意）

インフラデプロイ時に同期を自動実行するには:

1. **マニフェストを** コンパイル時に読み込む:
   ```bicep
   var agentDefinitions = loadJsonContent('foundry-agents.json')
   ```

2. Foundry プロジェクトに対して **Azure AI User** ロールを持つ **User-Assigned Managed Identity** を作成する。

3. **`Microsoft.Resources/deploymentScripts`** リソース（kind: `AzureCLI`）を作成し、以下を設定する:
   - マネージド ID を使用
   - `loadTextContent` で bash 同期スクリプトを読み込む
   - プロジェクト endpoint、definitions、model を環境変数として渡す

チームが有効/無効を選べるよう、`deployFoundryAgents` パラメータで制御してください。

## ワークフロー

### Step 1 — マニフェストを特定またはスキャフォールディング

リポジトリ内で `foundry-agents.json` を検索します。存在しない場合は、必要なエージェントをユーザーに確認してマニフェストを作成します。

### Step 2 — 同期スクリプトを特定またはスキャフォールディング

`sync-foundry-agents.ps1` または `foundry-agent-sync.sh` を検索します。見つからない場合は、上記テンプレートを使って PowerShell スクリプトを作成し、以下を調整します。
- プロジェクト名に合わせて `$AgentNamePrefix`
- ユーザーのデプロイ済みモデルに合わせて `$ModelName`
- 実際のマニフェスト位置に合わせて `$ManifestPath`

### Step 3 — パラメータを収集

ユーザーに次を確認します。
- Foundry project endpoint
- Subscription ID
- Model deployment name（デフォルト: `gpt-5-4`）
- Agent name prefix（デフォルト: リポジトリ名の kebab-case）

### Step 4 — 同期を実行

収集したパラメータで PowerShell スクリプトを実行します。

```powershell
.\infra\scripts\sync-foundry-agents.ps1 `
  -SubscriptionId '<sub-id>' `
  -ProjectEndpoint '<endpoint>' `
  -ModelName '<model>' `
  -AgentNamePrefix '<prefix>'
```

### Step 5 — 検証

一覧取得で同期済みエージェントを確認します。

```powershell
$token = az account get-access-token --resource https://ai.azure.com/ --query accessToken -o tsv
$endpoint = '<project-endpoint>'
Invoke-RestMethod -Uri "$endpoint/agents?api-version=2025-11-15-preview" `
  -Headers @{ Authorization = "Bearer $token" }
```

## REST API リファレンス

| Operation | Method | URL |
|---|---|---|
| Create/update agent | POST | `{projectEndpoint}/agents/{agentName}?api-version=2025-11-15-preview` |
| List agents | GET | `{projectEndpoint}/agents?api-version=2025-11-15-preview` |
| Get agent | GET | `{projectEndpoint}/agents/{agentName}?api-version=2025-11-15-preview` |
| Delete agent | DELETE | `{projectEndpoint}/agents/{agentName}?api-version=2025-11-15-preview` |

### Create/Update Payload

```json
{
  "definition": {
    "kind": "prompt",
    "model": "<deployed-model-name>",
    "instructions": "<system prompt>"
  },
  "description": "<agent description>",
  "metadata": {
    "useCaseId": "<use-case-id>",
    "managedBy": "foundry-agent-sync"
  }
}
```

## トラブルシューティング

| Symptom | Cause | Fix |
|---|---|---|
| `401 Unauthorized` | トークン期限切れ、または audience が誤り | `az account get-access-token --resource https://ai.azure.com/` を再実行 |
| `403 Forbidden` | Azure AI User ロールが不足 | Foundry プロジェクトスコープでロールを割り当て |
| `404 Not Found` | project endpoint が誤り | endpoint に `/api/projects/{projectName}` が含まれることを確認 |
| Model not found | プロジェクトにモデルが未デプロイ | 先に AI Foundry ポータルでモデルをデプロイ |
| Empty definitions | マニフェストパスが誤り | `-ManifestPath` が JSON ファイルを指しているか確認 |
