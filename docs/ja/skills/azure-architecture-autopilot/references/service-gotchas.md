# サービスの落とし穴（Stable）

サービスごとの **直感的でない必須プロパティ**、**よくあるミス**、および **PE マッピング** の要約です。  
ここには、ほぼ不変のパターンのみを含めています。API バージョン、SKU リスト、リージョンのような動的な値は含めていません。

---

## 1. 必須プロパティ（省略するとデプロイ失敗または機能問題）

| Service | Required Property | Result If Omitted | Notes |
|---------|------------------|-------------------|-------|
| ADLS Gen2 | `isHnsEnabled: true` | 通常の Blob Storage になる。元に戻せない | `kind: 'StorageV2'` が必要 |
| Storage Account | 名前に特殊文字/ハイフンを含めない | デプロイ失敗 | 英小文字+数字のみ、3〜24 文字 |
| Foundry (AIServices) | `customSubDomainName: foundryName` | Project を作成できず、作成後に変更不可 → リソースの削除・再作成が必要 | グローバル一意の値 |
| Foundry (AIServices) | `allowProjectManagement: true` | Foundry Project を作成できない | `kind: 'AIServices'` |
| Foundry (AIServices) | `identity: { type: 'SystemAssigned' }` | Project 作成に失敗 | |
| Foundry Project | Foundry リソースとセットで作成必須 | ポータルから利用できない | `accounts/projects` |
| Key Vault | `enableRbacAuthorization: true` | Access Policy との混在利用リスク | |
| Key Vault | `enablePurgeProtection: true` | 本番では必須 | |
| Fabric Capacity | `administration.members` 必須 | デプロイ失敗 | 管理者メール |
| PE Subnet | `privateEndpointNetworkPolicies: 'Disabled'` | PE デプロイ失敗 | |
| PE DNS Zone | `registrationEnabled: false` (VNet Link) | DNS 競合の可能性 | |
| PE Configuration | 3 要素セット（PE + DNS Zone + VNet Link + Zone Group） | PE があっても DNS 解決に失敗 | |

---

## 2. PE groupId と DNS Zone の対応（主要サービス）

以下の対応は安定していますが、新しいサービスを追加する際は `azure-dynamic-sources.md` の PE DNS 統合ドキュメントで再確認してください。

| Service | groupId | Private DNS Zone |
|---------|---------|-----------------|
| Azure OpenAI / CognitiveServices | `account` | `privatelink.cognitiveservices.azure.com` |
| ⚠️ (Foundry/AIServices additional) | `account` | `privatelink.openai.azure.com` ← **両方のゾーンを DNS Zone Group に含める必要があります。省略すると OpenAI API の DNS 解決に失敗します** |
| Azure AI Search | `searchService` | `privatelink.search.windows.net` |
| Storage (Blob/ADLS) | `blob` | `privatelink.blob.core.windows.net` |
| Storage (DFS/ADLS Gen2) | `dfs` | `privatelink.dfs.core.windows.net` |
| Key Vault | `vault` | `privatelink.vaultcore.azure.net` |
| Azure ML / AI Hub | `amlworkspace` | `privatelink.api.azureml.ms` |
| Container Registry | `registry` | `privatelink.azurecr.io` |
| Cosmos DB (SQL) | `Sql` | `privatelink.documents.azure.com` |
| Azure Cache for Redis | `redisCache` | `privatelink.redis.cache.windows.net` |
| Data Factory | `dataFactory` | `privatelink.datafactory.azure.net` |
| API Management | `Gateway` | `privatelink.azure-api.net` |
| Event Hub | `namespace` | `privatelink.servicebus.windows.net` |
| Service Bus | `namespace` | `privatelink.servicebus.windows.net` |
| Monitor (AMPLS) | ⚠️ 複雑な構成 — 以下参照 | ⚠️ 複数の DNS Zone が必要 — 以下参照 |

> **ADLS Gen2 Note**: `isHnsEnabled: true` の場合、**`blob` と `dfs` の両方の PE が必要**です。  
> - `blob` PE のみだと Blob API は動作しますが、Data Lake 操作（ファイルシステム作成、ディレクトリ操作、`abfss://` プロトコル）は失敗します。  
> - DFS PE: groupId `dfs`、DNS Zone `privatelink.dfs.core.windows.net`  
>
> **⚠️ Azure Monitor Private Link (AMPLS) Note**: Azure Monitor は単一 PE + 単一 DNS Zone では構成できません。Azure Monitor Private Link Scope (AMPLS) 経由で接続し、次の **5 つの DNS Zone** がすべて必要です。  
> - `privatelink.monitor.azure.com`
> - `privatelink.oms.opinsights.azure.com`
> - `privatelink.ods.opinsights.azure.com`
> - `privatelink.agentsvc.azure-automation.net`
> - `privatelink.blob.core.windows.net` (Log Analytics データ取り込み用)
>
> このマッピングは複雑で変更される可能性があるため、Monitor PE を構成する際は必ず MS Docs を取得して確認してください。  
> https://learn.microsoft.com/en-us/azure/azure-monitor/logs/private-link-configure

---

## 3. よくあるミスのチェックリスト

| Item | ❌ Incorrect Example | ✅ Correct Example |
|------|---------------------|-------------------|
| ADLS Gen2 HNS | `isHnsEnabled` を省略または `false` | `isHnsEnabled: true` |
| PE Subnet | Policy 未設定 | `privateEndpointNetworkPolicies: 'Disabled'` |
| DNS Zone Group | PE のみ作成 | PE + DNS Zone + VNet Link + DNS Zone Group |
| Foundry resource | `kind: 'OpenAI'` | `kind: 'AIServices'` + `allowProjectManagement: true` |
| Foundry resource | `customSubDomainName` を省略 | `customSubDomainName: foundryName` — 作成後に変更不可 |
| Foundry Project | Project なしで Foundry のみ存在 | セットで作成必須 |
| Key Vault auth | Access Policy | `enableRbacAuthorization: true` |
| Public network | 未設定 | `publicNetworkAccess: 'Disabled'` |
| Storage name | `st-my-storage` | `stmystorage` または `st${uniqueString(...)}` |
| API version | 前回の会話/エラーからコピー | MS Docs で最新 stable を確認 |
| Region | ハードコード（`'eastus'`） | パラメーターで渡す（`param location`） |
| Sensitive values | `.bicepparam` に平文 | `@secure()` + Key Vault 参照 |

---

## 4. サービス関係の判断ルール

絶対的な決定ではなく、**デフォルトの選択ルール**として記述します。

### Foundry vs Azure OpenAI vs AI Hub

```
デフォルトルール:
├─ AI/RAG ワークロード → Microsoft Foundry を使用 (kind: 'AIServices')
│   ├─ Foundry リソース + Foundry Project をセットで作成
│   └─ モデルデプロイは Foundry リソースレベルで実施 (accounts/deployments)
│
├─ ML/オープンソースモデルの学習が必要 → AI Hub (MachineLearningServices) を検討
│   └─ ユーザーが明示的に要求する場合、または Foundry で未対応の機能が必要な場合のみ
│
└─ 単独の Azure OpenAI リソース →
    ユーザーが明示的に要求する場合、または
    公式ドキュメントで別リソースが必要な場合のみ検討
```

> これらのルールは、現在の MS 推奨を反映した **デフォルト選択ガイド** です。  
> Azure 製品間の関係は変わる可能性があるため、不確かな場合は MS Docs を確認してください。

### Monitoring

```
デフォルトルール:
├─ Foundry (AIServices) → Application Insights は不要
└─ AI Hub (MachineLearningServices) → Application Insights + Log Analytics が必要
```

