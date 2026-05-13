# Azure 動的ソースレジストリ

このファイルは、**頻繁に変化する情報のソース（URL）のみ**を管理します。  
実際の値（API バージョン、SKU、リージョンなど）はここには記録しません。  
Bicep を生成する前に、必ず以下の URL を取得して最新情報を確認してください。

---

## 1. Bicep API バージョン（常に取得必須）

サービスごとの MS Docs Bicep リファレンスです。使用前に、これらの URL から最新の安定版 apiVersion を確認してください。

| Service | MS Docs URL |
|---------|-------------|
| CognitiveServices (Foundry/OpenAI) | https://learn.microsoft.com/en-us/azure/templates/microsoft.cognitiveservices/accounts |
| AI Search | https://learn.microsoft.com/en-us/azure/templates/microsoft.search/searchservices |
| Storage Account | https://learn.microsoft.com/en-us/azure/templates/microsoft.storage/storageaccounts |
| Key Vault | https://learn.microsoft.com/en-us/azure/templates/microsoft.keyvault/vaults |
| Virtual Network | https://learn.microsoft.com/en-us/azure/templates/microsoft.network/virtualnetworks |
| Private Endpoints | https://learn.microsoft.com/en-us/azure/templates/microsoft.network/privateendpoints |
| Private DNS Zones | https://learn.microsoft.com/en-us/azure/templates/microsoft.network/privatednszones |
| Fabric | https://learn.microsoft.com/en-us/azure/templates/microsoft.fabric/capacities |
| Data Factory | https://learn.microsoft.com/en-us/azure/templates/microsoft.datafactory/factories |
| Application Insights | https://learn.microsoft.com/en-us/azure/templates/microsoft.insights/components |
| ML Workspace (Hub) | https://learn.microsoft.com/en-us/azure/templates/microsoft.machinelearningservices/workspaces |

> **子リソースも必ず確認してください**: `accounts/projects`、`accounts/deployments`、`privateDnsZones/virtualNetworkLinks` などの子リソースは、親と異なる API バージョンを持つ場合があります。確認するには、親ページから子リソースのリンクをたどってください。

### 上記テーブルにないサービス

上記テーブルには v1 スコープのサービスのみを掲載しています。その他のサービスは、次の形式で URL を組み立てて取得してください:
```
https://learn.microsoft.com/en-us/azure/templates/microsoft.{provider}/{resourceType}
```

---

## 2. モデル可用性（Foundry/OpenAI モデル利用時は必須）

対象リージョンでそのモデル名がデプロイ可能かを確認してください。静的な知識には依存しないでください。

| Verification Method | URL / Command |
|--------------------|---------------|
| MS Docs model availability | https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/models |
| Azure CLI (existing resources) | `az cognitiveservices account list-models --name "<NAME>" --resource-group "<RG>" -o table` |

> 対象リージョンでモデルが利用不可の場合 → ユーザーに通知し、利用可能なリージョン/代替モデルを提案してください。ユーザー承認なしに置き換えてはいけません。

---

## 3. Private Endpoint マッピング（新しいサービス追加時）

PE の groupId と DNS Zone のマッピングは Azure 側で変更される可能性があります。新しいサービスを追加する場合、または確認が必要な場合:

| Verification Method | URL |
|--------------------|-----|
| PE DNS integration official docs | https://learn.microsoft.com/en-us/azure/private-link/private-endpoint-dns |

> `service-gotchas.md` にある主要サービスのマッピングは安定していますが、新しいサービスを追加する際は必ず上記 URL で再確認してください。

---

## 4. サービスのリージョン可用性

特定のサービスが特定のリージョンで利用可能かを確認します:

| Verification Method | URL |
|--------------------|-----|
| Azure service-by-region availability | https://azure.microsoft.com/en-us/explore/global-infrastructure/products-by-region/ |

---

## 5. Azure Updates（補助的な情報把握）

以下のソースは**参照専用**です。一次情報源は常に MS Docs の公式ドキュメントです。

| Source | URL | Purpose |
|--------|-----|---------|
| Azure Updates | https://azure.microsoft.com/en-us/updates/ | サービス変更の把握 |
| What's New in Azure | Docs 内のサービス別 What's New ページ | 機能変更の確認 |

---

## 判断ルール: いつ取得するか？

| Information Type | Must Fetch? | Rationale |
|-----------------|-------------|-----------|
| API version | **常に取得** | 変更頻度が高く、誤った値はデプロイ失敗を引き起こす |
| Model availability (name, region) | **常に取得** | リージョンごとに異なり、変更頻度が高い |
| SKU list | **常に取得** | サービスごとに変更される可能性がある |
| Region availability | **常に取得** | サービスごとのリージョンサポートは頻繁に変わる。ユーザー指定リージョンがそのサービスで利用可能かを必ず確認すること |
| PE groupId & DNS Zone | v1 の主要サービスは `service-gotchas.md` を参照可能。**ただし新規サービスや複雑な構成（Monitor など）では取得必須** | 主要サービスのマッピングは安定しているが、新規/複雑サービスはリスクが高い |
| Required property patterns | まず参照ファイルを確認 | ほぼ不変（isHnsEnabled など） |

