# フェーズ 0: 既存リソーススキャナー

このファイルには、フェーズ 0 の詳細な手順が記載されています。ユーザーが既存の Azure リソース分析（Path B）を要求した場合は、このファイルを読み、その手順に従ってください。

スキャン結果はアーキテクチャ図として可視化され、続くユーザーの自然言語による変更要求はフェーズ 1 にルーティングされます。

> **🚨 出力保存パスのルール**: すべての出力（スキャン JSON、図の HTML、Bicep コード）は、**現在の作業ディレクトリ（cwd）配下のプロジェクトフォルダー**に保存する必要があります。`~/.copilot/session-state/` の中には絶対に保存しないでください。session-state ディレクトリは一時領域であり、セッション終了時に削除される可能性があります。

---

## ステップ 1: Azure ログイン + スキャン範囲の選択

### 1-A: Azure ログインの確認

```powershell
az account show 2>&1
```

- ログイン済みの場合 → ステップ 1-B へ進む
- 未ログインの場合 → ユーザーに `az login` の実行を依頼する

### 1-B: サブスクリプション選択（複数選択対応）

```powershell
az account list --output json
```

サブスクリプション一覧を `ask_user` の選択肢として提示します。**複数のサブスクリプションを選択できます:**
```
ask_user({
  question: "分析する Azure サブスクリプションを選択してください。（複数選択する場合は、1 つずつ追加できます）",
  choices: [
    "sub-002 (現在の既定サブスクリプション) (推奨)",
    "sub-001",
    "上記すべてのサブスクリプションを分析"
  ]
})
```

- 単一サブスクリプションを選択 → そのサブスクリプションのみスキャン
- "Analyze all" を選択 → すべてのサブスクリプションをスキャン
- ユーザーが追加のサブスクリプションを希望する場合 → `ask_user` を再度使って追加する

### 1-C: スキャン範囲の選択（複数 RG 選択対応）

```
ask_user({
  question: "Azure リソースをどの範囲で分析しますか？",
  choices: [
    "特定のリソース グループを指定（推奨）",
    "複数のリソース グループを選択",
    "現在のサブスクリプション内のすべてのリソース グループ"
  ]
})
```

- **特定 RG** → RG 一覧から選択、または手動入力
- **複数 RG** → `ask_user` を繰り返し、RG を 1 つずつ追加。ユーザーが「それで十分」と言ったら終了。
  あるいは、ユーザーはカンマ区切りで複数 RG を入力できます（例: `rg-prod, rg-dev, rg-network`）
- **サブスクリプション全体** → `az group list` → すべての RG をスキャン（リソースが多い場合は時間がかかる可能性を警告）

**複数サブスクリプション + 複数 RG の組み合わせに対応:**
- サブスクリプション A の rg-prod + サブスクリプション B の rg-network → 両方をスキャンし、1 つの図に表示

---

## 図の階層 — 複数サブスクリプション/RG の表示

**単一サブスクリプション + 単一 RG**: これまでと同じ（VNet 境界のみ）
**複数 RG（同一サブスクリプション）**: RG ごとに破線境界
**複数サブスクリプション**: Subscription > RG の 2 階層境界

図用 JSON に階層情報を渡します。

**services JSON に `subscription` と `resourceGroup` フィールドを追加:**
```json
{
  "id": "foundry",
  "name": "foundry-xxx",
  "type": "ai_foundry",
  "subscription": "sub-002",
  "resourceGroup": "rg-prod",
  "details": [...]
}
```

**`--hierarchy` パラメーターで階層情報を渡す:**
```
--hierarchy '[{"subscription":"sub-002","resourceGroups":["rg-prod","rg-dev"]},{"subscription":"sub-001","resourceGroups":["rg-network"]}]'
```

この情報に基づき、図スクリプトは次を行います:
- 複数 RG → 各 RG を破線境界のクラスターとして表現（ラベル: RG 名）
- 複数サブスクリプション → RG 境界を、より大きなサブスクリプション境界の内側にネスト
- VNet 境界は、その VNet が属する RG の内側に表示

---

## ステップ 2: リソーススキャン

**🚨 az CLI 出力の原則:**
- az CLI の出力は **必ずファイルに保存** してから `view` で読み取ること。ターミナルへの直接出力は途中で切れる可能性があります。
- 1 回の PowerShell 呼び出しでまとめる az コマンドは **最大 3 つまで**。多すぎるとタイムアウトの原因になります。
- `--query` JMESPath を使って必要なフィールドだけ抽出し、出力サイズを減らしてください。

```powershell
# ✅ 正しい方法 — ファイルへ保存してから読む
az resource list -g "<RG>" --query "[].{name:name,type:type,kind:kind,location:location}" -o json | Set-Content -Path "$outDir/resources.json"

# ❌ 間違った方法 — ターミナルへ直接出力（途中で切れる可能性）
az resource list -g "<RG>" -o json
```

### 2-A: 全リソース一覧 + ユーザーへの表示

```powershell
$outDir = "<project-name>/azure-scan"
New-Item -ItemType Directory -Path $outDir -Force | Out-Null

# Step 1: 基本リソース一覧（name, type, kind, location）
az resource list -g "<RG>" --query "[].{name:name,type:type,kind:kind,location:location,id:id}" -o json | Set-Content "$outDir/resources.json"
```

**🚨 resources.json を読んだ直後に、必ず完全なリソース一覧表をユーザーへ表示すること:**

```
📋 rg-<RG> Resource List (N resources)

┌─────────────────────────┬──────────────────────────────────────────────┬─────────────────┐
│ Name                    │ Type                                         │ Location        │
├─────────────────────────┼──────────────────────────────────────────────┼─────────────────┤
│ my-storage              │ Microsoft.Storage/storageAccounts             │ koreacentral    │
│ my-keyvault             │ Microsoft.KeyVault/vaults                    │ koreacentral    │
│ ...                     │ ...                                          │ ...             │
└─────────────────────────┴──────────────────────────────────────────────┴─────────────────┘

⏳ Retrieving detailed information...
```

詳細クエリへ進む前に、この表を **最初に** 表示してください。どんなリソースが存在するか分からないままユーザーを待たせてはいけません。

### 2-B: 動的な詳細クエリ — resources.json に基づく

**resources.json で見つかったリソースタイプに基づいて、詳細クエリコマンドを動的に決定します。**

固定のコマンドリストは使わないでください。以下のマッピング表から、resources.json に存在するタイプのコマンドだけを実行します。

**Type → 詳細クエリコマンドのマッピング:**

| Type in resources.json | Detailed Query Command | Output File |
|---|---|---|
| `Microsoft.Network/virtualNetworks` | `az network vnet list -g "<RG>" --query "[].{name:name,addressSpace:addressSpace.addressPrefixes,subnets:subnets[].{name:name,prefix:addressPrefix,pePolicy:privateEndpointNetworkPolicies}}" -o json` | `vnets.json` |
| `Microsoft.Network/privateEndpoints` | `az network private-endpoint list -g "<RG>" --query "[].{name:name,subnetId:subnet.id,targetId:privateLinkServiceConnections[0].privateLinkServiceId,groupIds:privateLinkServiceConnections[0].groupIds,state:provisioningState}" -o json` | `pe.json` |
| `Microsoft.Network/networkSecurityGroups` | `az network nsg list -g "<RG>" --query "[].{name:name,location:location,subnets:subnets[].id,nics:networkInterfaces[].id}" -o json` | `nsg.json` |
| `Microsoft.CognitiveServices/accounts` | `az cognitiveservices account list -g "<RG>" --query "[].{name:name,kind:kind,sku:sku.name,endpoint:properties.endpoint,publicAccess:properties.publicNetworkAccess,location:location}" -o json` | `cognitive.json` |
| `Microsoft.Search/searchServices` | `az search service list -g "<RG>" --query "[].{name:name,sku:sku.name,publicAccess:properties.publicNetworkAccess,semanticSearch:properties.semanticSearch,location:location}" -o json 2>$null` | `search.json` |
| `Microsoft.Compute/virtualMachines` | `az vm list -g "<RG>" --query "[].{name:name,size:hardwareProfile.vmSize,os:storageProfile.osDisk.osType,location:location,nicIds:networkProfile.networkInterfaces[].id}" -o json` | `vms.json` |
| `Microsoft.Storage/storageAccounts` | `az storage account list -g "<RG>" --query "[].{name:name,sku:sku.name,kind:kind,hns:properties.isHnsEnabled,publicAccess:properties.publicNetworkAccess,location:location}" -o json` | `storage.json` |
| `Microsoft.KeyVault/vaults` | `az keyvault list -g "<RG>" --query "[].{name:name,location:location}" -o json 2>$null` | `keyvault.json` |
| `Microsoft.ContainerService/managedClusters` | `az aks list -g "<RG>" --query "[].{name:name,kubernetesVersion:kubernetesVersion,sku:sku,agentPoolProfiles:agentPoolProfiles[].{name:name,count:count,vmSize:vmSize},networkProfile:networkProfile.networkPlugin,location:location}" -o json` | `aks.json` |
| `Microsoft.Web/sites` | `az webapp list -g "<RG>" --query "[].{name:name,kind:kind,sku:appServicePlan,state:state,defaultHostName:defaultHostName,httpsOnly:httpsOnly,location:location}" -o json` | `webapps.json` |
| `Microsoft.Web/serverFarms` | `az appservice plan list -g "<RG>" --query "[].{name:name,sku:sku.name,tier:sku.tier,kind:kind,location:location}" -o json` | `appservice-plans.json` |
| `Microsoft.DocumentDB/databaseAccounts` | `az cosmosdb list -g "<RG>" --query "[].{name:name,kind:kind,databaseAccountOfferType:databaseAccountOfferType,locations:locations[].locationName,publicAccess:publicNetworkAccess}" -o json` | `cosmosdb.json` |
| `Microsoft.Sql/servers` | `az sql server list -g "<RG>" --query "[].{name:name,fullyQualifiedDomainName:fullyQualifiedDomainName,publicAccess:publicNetworkAccess,location:location}" -o json` | `sql-servers.json` |
| `Microsoft.Databricks/workspaces` | `az databricks workspace list -g "<RG>" --query "[].{name:name,sku:sku.name,url:workspaceUrl,publicAccess:parameters.enableNoPublicIp.value,location:location}" -o json 2>$null` | `databricks.json` |
| `Microsoft.Synapse/workspaces` | `az synapse workspace list -g "<RG>" --query "[].{name:name,sqlAdminLogin:sqlAdministratorLogin,publicAccess:publicNetworkAccess,location:location}" -o json 2>$null` | `synapse.json` |
| `Microsoft.DataFactory/factories` | `az datafactory list -g "<RG>" --query "[].{name:name,publicAccess:publicNetworkAccess,location:location}" -o json 2>$null` | `adf.json` |
| `Microsoft.EventHub/namespaces` | `az eventhubs namespace list -g "<RG>" --query "[].{name:name,sku:sku.name,location:location}" -o json` | `eventhub.json` |
| `Microsoft.Cache/redis` | `az redis list -g "<RG>" --query "[].{name:name,sku:sku.name,port:port,sslPort:sslPort,publicAccess:publicNetworkAccess,location:location}" -o json` | `redis.json` |
| `Microsoft.ContainerRegistry/registries` | `az acr list -g "<RG>" --query "[].{name:name,sku:sku.name,adminUserEnabled:adminUserEnabled,publicAccess:publicNetworkAccess,location:location}" -o json` | `acr.json` |
| `Microsoft.MachineLearningServices/workspaces` | `az resource show --ids "<ID>" --query "{name:name,sku:sku,kind:kind,location:location,publicAccess:properties.publicNetworkAccess,hbiWorkspace:properties.hbiWorkspace,managedNetwork:properties.managedNetwork.isolationMode}" -o json` | `mlworkspace.json` |
| `Microsoft.Insights/components` | `az monitor app-insights component show -g "<RG>" --app "<NAME>" --query "{name:name,kind:kind,instrumentationKey:instrumentationKey,workspaceResourceId:workspaceResourceId,location:location}" -o json 2>$null` | `appinsights-<NAME>.json` |
| `Microsoft.OperationalInsights/workspaces` | `az monitor log-analytics workspace show -g "<RG>" -n "<NAME>" --query "{name:name,sku:sku.name,retentionInDays:retentionInDays,location:location}" -o json` | `log-analytics-<NAME>.json` |
| `Microsoft.Network/applicationGateways` | `az network application-gateway list -g "<RG>" --query "[].{name:name,sku:sku,location:location}" -o json` | `appgateway.json` |
| `Microsoft.Cdn/profiles` / `Microsoft.Network/frontDoors` | `az afd profile list -g "<RG>" --query "[].{name:name,sku:sku.name,location:location}" -o json 2>$null` | `frontdoor.json` |
| `Microsoft.Network/azureFirewalls` | `az network firewall list -g "<RG>" --query "[].{name:name,sku:sku,threatIntelMode:threatIntelMode,location:location}" -o json` | `firewall.json` |
| `Microsoft.Network/bastionHosts` | `az network bastion list -g "<RG>" --query "[].{name:name,sku:sku.name,location:location}" -o json` | `bastion.json` |

**動的クエリの手順:**

1. `resources.json` を読む
2. `type` フィールドの重複しない値を抽出
3. 上記マッピング表のうち、**一致する type のコマンドだけ**を実行（存在しない type はスキップ）
4. マッピング表にない type が見つかった場合 → 汎用クエリを使用: `az resource show --ids "<ID>" --query "{name:name,sku:sku,kind:kind,location:location,properties:properties}" -o json`
5. コマンドは 2～3 個ずつのバッチで実行（全件同時実行しない）

### 2-C: モデルデプロイメントクエリ（Cognitive Services がある場合）

```powershell
# 各 Cognitive Services リソースのモデルデプロイメントを取得
az cognitiveservices account deployment list --name "<NAME>" -g "<RG>" --query "[].{name:name,model:properties.model.name,version:properties.model.version,sku:sku.name}" -o json | Set-Content "$outDir/<NAME>-deployments.json"
```

### 2-D: NIC + Public IP クエリ（VM がある場合）

```powershell
az network nic list -g "<RG>" --query "[].{name:name,subnetId:ipConfigurations[0].subnet.id,privateIp:ipConfigurations[0].privateIPAddress,publicIpId:ipConfigurations[0].publicIPAddress.id}" -o json | Set-Content "$outDir/nics.json"
az network public-ip list -g "<RG>" --query "[].{name:name,ip:ipAddress,sku:sku.name}" -o json | Set-Content "$outDir/public-ips.json"
```

VNet から取得する情報:
- `addressSpace.addressPrefixes` → CIDR
- `subnets[].name`, `subnets[].addressPrefix` → サブネット情報
- `subnets[].privateEndpointNetworkPolicies` → PE ポリシー

---

## ステップ 3: リソース間の関係推論

スキャンしたリソース間の**関係（接続）**を自動推論し、図のための connections JSON を構築します。

### 関係推論ルール

**🚨 接続線が不足すると図が意味を持たなくなります。可能な限り多くの関係を推論してください。**

#### 確定的推論（リソース ID / プロパティから直接検証可能）

| Relationship Type | Inference Method | connection type |
|---|---|---|
| PE → Service | PE の `privateLinkServiceId` から service ID を抽出 | `private` |
| PE → VNet | PE の `subnet.id` から VNet を抽出 | （VNet 境界として表現） |
| Foundry → Project | `accounts/projects` の親リソース | `api` |
| VM → NIC → Subnet | NIC の `subnet.id` から VNet/Subnet を推論 | （VNet 境界） |
| NSG → Subnet | NSG の `subnets[].id` から接続サブネットを確認 | `network` |
| NSG → NIC | NSG の `networkInterfaces[].id` から接続 VM を確認 | `network` |
| NIC → Public IP | NIC の `publicIPAddress.id` から PIP を確認 | （details に含める） |
| Databricks → VNet | Workspace の VNet Injection 構成 | （VNet 境界） |

#### 妥当な推論（同一 RG 内サービス間の一般的パターン）

| Relationship Type | Inference Condition | connection type |
|---|---|---|
| Foundry → AI Search | 同一 RG 内に両方存在 → RAG 接続と推論 | `api`（label: "RAG Search"） |
| Foundry → Storage | 同一 RG 内に両方存在 → データ接続と推論 | `data`（label: "Data"） |
| AI Search → Storage | 同一 RG 内に両方存在 → インデックス接続と推論 | `data`（label: "Indexing"） |
| Service → Key Vault | 同一 RG 内に Key Vault が存在 → シークレット管理と推論 | `security`（label: "Secrets"） |
| VM → Foundry/Search | 同一 RG 内に VM + AI サービスが存在 → API 呼び出しと推論 | `api`（label: "API"） |
| DI → Foundry | 同一 RG 内に Document Intelligence + Foundry が存在 → OCR/抽出接続と推論 | `api`（label: "OCR/Extract"） |
| ADF → Storage | 同一 RG 内に ADF + Storage が存在 → データパイプラインと推論 | `data`（label: "Pipeline"） |
| ADF → SQL | 同一 RG 内に ADF + SQL が存在 → データソースと推論 | `data`（label: "Source"） |
| Databricks → Storage | 同一 RG 内に両方存在 → データレイク接続と推論 | `data`（label: "Data Lake"） |

#### 推論後のユーザー確認

推論した接続一覧をユーザーに提示し、確認を求めてください:
```
> **⏳ リソース間の関係を推論しました** — 以下が正しいか確認してください。

推論された接続:
- Foundry → AI Search (RAG Search)
- Foundry → Storage (Data)
- VM → Foundry (API Call)
- Document Intelligence → Foundry (OCR/Extract)

この内容で問題ないですか？追加または削除したい接続があれば教えてください。
```

#### 推論できない関係

上記ルールでは推論できない接続が存在する場合があります。ユーザーは追加の接続を自由に追加できます。

### モデルデプロイメントクエリ（Foundry リソースがある場合）

```powershell
az cognitiveservices account deployment list --name "<FOUNDRY_NAME>" -g "<RG>" --query "[].{name:name,model:properties.model.name,version:properties.model.version,sku:sku.name}" -o json
```

各デプロイメントのモデル名、バージョン、SKU を Foundry ノードの details に追加してください。

---

## ステップ 4: services/connections JSON への変換

スキャン結果を、組み込み図エンジンの入力形式に変換します。

### リソースタイプ → 図 type マッピング

| Azure Resource Type | Diagram type |
|---|---|
| `Microsoft.CognitiveServices/accounts` (kind: AIServices) | `ai_foundry` |
| `Microsoft.CognitiveServices/accounts` (kind: OpenAI) | `openai` |
| `Microsoft.CognitiveServices/accounts` (kind: FormRecognizer) | `document_intelligence` |
| `Microsoft.CognitiveServices/accounts` (kind: TextAnalytics, etc.) | `ai_foundry` (default) |
| `Microsoft.CognitiveServices/accounts/projects` | `ai_foundry` |
| `Microsoft.Search/searchServices` | `search` |
| `Microsoft.Storage/storageAccounts` | `storage` |
| `Microsoft.KeyVault/vaults` | `keyvault` |
| `Microsoft.Databricks/workspaces` | `databricks` |
| `Microsoft.Sql/servers` | `sql_server` |
| `Microsoft.Sql/servers/databases` | `sql_database` |
| `Microsoft.DocumentDB/databaseAccounts` | `cosmos_db` |
| `Microsoft.Web/sites` | `app_service` |
| `Microsoft.ContainerService/managedClusters` | `aks` |
| `Microsoft.Web/sites` (kind: functionapp) | `function_app` |
| `Microsoft.Synapse/workspaces` | `synapse` |
| `Microsoft.Fabric/capacities` | `fabric` |
| `Microsoft.DataFactory/factories` | `adf` |
| `Microsoft.Compute/virtualMachines` | `vm` |
| `Microsoft.Network/privateEndpoints` | `pe` |
| `Microsoft.Network/virtualNetworks` | （VNet 境界として表現 — services には含めない） |
| `Microsoft.Network/networkSecurityGroups` | `nsg` |
| `Microsoft.Network/bastionHosts` | `bastion` |
| `Microsoft.OperationalInsights/workspaces` | `log_analytics` |
| `Microsoft.Insights/components` | `app_insights` |
| その他 | `default` |

### services JSON 構築ルール

```json
{
  "id": "resource name (lowercase, special characters removed)",
  "name": "actual resource name",
  "type": "determined from the mapping table above",
  "sku": "actual SKU (if available)",
  "private": true/false,  // true if a PE is connected
  "details": ["property1", "property2", ...]
}
```

**details に含める情報:**
- Endpoint URL
- SKU/tier の詳細
- kind（AIServices、OpenAI など）
- モデルデプロイメント一覧（Foundry）
- 主要プロパティ（isHnsEnabled、semanticSearch など）
- リージョン

### VNet 情報 → `--vnet-info` パラメーター

VNet が見つかった場合、`--vnet-info` で境界ラベルに表示します:
```
--vnet-info "10.0.0.0/16 | pe-subnet: 10.0.1.0/24 | <region>"
```

### PE ノード生成

PE が見つかった場合、各 PE を個別ノードとして追加し、対応するサービスと `private` type で接続します:
```json
{"id": "pe_<serviceId>", "name": "PE: <serviceName>", "type": "pe", "details": ["groupId: <groupId>", "<status>"]}
```

---

## ステップ 5: 図の生成 + ユーザーへの提示

図のファイル名: `<project-name>/00_arch_current.html`

スキャンした RG 名を既定のプロジェクト名として使用します:
```
ask_user({
  question: "プロジェクト名を選択してください。（スキャン結果のフォルダー名になります）",
  choices: ["<RG-name>", "azure-analysis"]
})
```

図を生成した後、次を報告します:
```
## Current Azure Architecture

[Interactive Diagram — 00_arch_current.html]

Scanned Resources (N total):
[Summary table by resource type]

What would you like to change here?
- 🔧 Performance improvement ("it's slow", "increase throughput")
- 💰 Cost optimization ("reduce costs", "make it cheaper")
- 🔒 Security hardening ("add PE", "block public access")
- 🌐 Network changes ("separate VNet", "add Bastion")
- ➕ Add/remove resources ("add a VM", "delete this")
- 📊 Monitoring ("set up logs", "add alerts")
- 🤔 Diagnostics ("is this architecture OK?", "what's wrong?")
- Or just take the diagram and stop here
```

---

## ステップ 6: 変更会話 → フェーズ 1 へ移行

ユーザーが変更を要求したら、フェーズ 1（phase1-advisor.md）へ移行します。
これは既存のスキャン結果をベースラインとして使う **Path B のエントリーポイント** です。

### 自然言語での変更要求処理 — 明確化質問パターン

ユーザーの曖昧な要求を具体化するため、明確化質問を行います:

**🔧 パフォーマンス**

| User Request | Clarifying Question Example |
|---|---|
| "It's slow" / "Response takes too long" | "どのサービスが遅いですか？SKU を上げますか、それともリージョンを変更しますか？" |
| "I want to increase throughput" | "どのサービスのスループットを上げますか？スケールアウトしますか？DTU/RU を増やしますか？" |
| "AI Search indexing is slow" | "パーティションを追加しますか？SKU を S2 に上げますか？" |

**💰 コスト**

| User Request | Clarifying Question Example |
|---|---|
| "I want to reduce costs" | "どのサービスのコストを下げますか？SKU ダウングレードですか？未使用リソースを整理しますか？" |
| "How much does this cost?" | MS Docs の価格情報を調べ、現在の SKU を基に概算コストを提示 |
| "It's a dev environment, so make it cheap" | "Free/Basic ティアに切り替えますか？対象サービスはどれですか？" |

**🔒 セキュリティ**

| User Request | Clarifying Question Example |
|---|---|
| "Harden the security" | "PE がないサービスに PE を追加しますか？RBAC を確認しますか？publicNetworkAccess を無効化しますか？" |
| "Block public access" | "すべてのサービスに PE + publicNetworkAccess: Disabled を適用しますか？" |
| "Manage the keys" | "Key Vault を追加して Managed Identity と接続しますか？" |

**🌐 ネットワーク**

| User Request | Clarifying Question Example |
|---|---|
| "Add PE" | "どのサービスに追加しますか？すべてのサービスへ一括追加しますか？" |
| "Separate the VNet" | "どのサブネットを分離しますか？NSG も追加しますか？" |
| "Add Bastion" | "VM アクセス用に Azure Bastion を追加します。サブネット CIDR を指定してください。" |

**➕ リソースの追加/削除**

| User Request | Clarifying Question Example |
|---|---|
| "Add a VM" | "何台追加しますか？SKU は？同じ VNet にしますか？OS は？" |
| "Add Fabric" | "SKU は何にしますか？管理者メールは何ですか？" |
| "Delete this" | "[resource name] を削除してよろしいですか？接続された PE もあわせて削除されます。" |

**📊 監視/運用**

| User Request | Clarifying Question Example |
|---|---|
| "I want to see logs" | "Log Analytics Workspace を追加して Diagnostic Settings を接続しますか？" |
| "Set up alerts" | "どのメトリクスに対してですか？CPU？エラー率？応答時間？" |
| "Attach Application Insights" | "どのサービスに接続しますか？App Service？Function App？" |

**🔄 移行/変更**

| User Request | Clarifying Question Example |
|---|---|
| "Change the region" | "どのリージョンに変更しますか？そのリージョンで全サービスが利用可能か確認します。" |
| "Switch SQL to Cosmos" | "Cosmos DB の API タイプは何にしますか？（SQL/MongoDB/Cassandra）データ移行ガイドも提供できます。" |
| "Switch Foundry to Hub" | "Hub は ML 学習/オープンソースモデルが必要な場合にのみ適しています。ユースケースを確認させてください。" |

**🤔 診断/質問**

| User Request | Clarifying Question Example |
|---|---|
| "What's wrong?" | 現在の構成（publicNetworkAccess が開放、PE 未接続、不適切な SKU など）を分析し、改善案を提案 |
| "Is this architecture OK?" | Well-Architected Framework（セキュリティ、信頼性、性能、コスト、運用）に照らしてレビュー |
| "Is the PE connected properly?" | `az network private-endpoint show` で接続状態を確認して報告 |
| "Just give me the diagram" | フェーズ 1 には移行せず、00_arch_current.html のパスを提示して終了 |

変更内容が確定したら:
1. フェーズ 1 の Delta Confirmation Rule を適用
2. ファクトチェック（MS Docs と相互検証）
3. 更新後の図を生成（01_arch_diagram_draft.html）
4. ユーザー確認 → フェーズ 2–4 へ進む

---

## スキャン性能の最適化

- リソース数が 50 以上の場合は、ユーザーに警告: 「リソース数が多いため、スキャンに時間がかかる可能性があります。」
- まず `az resource list` を実行してリソース数を把握し、その後に詳細クエリへ進む
- 主要サービス（Foundry、Search、Storage、KeyVault、VNet、PE）を先に問い合わせ、残りは `az resource show` で基本情報のみ収集
- 進捗をユーザーに共有する:
  > **⏳ リソースをスキャン中** — N 件中 M 件完了

---

## 未対応リソースの扱い

図の type マッピングにないリソースタイプについては:
- `default` type（疑問符アイコン）で表示
- details にリソース名とタイプを含める
- ユーザーには表示するが、関係推論は試みない

