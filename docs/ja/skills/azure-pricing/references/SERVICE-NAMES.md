# Azure サービス名リファレンス

Azure Retail Prices API の `serviceName` フィールドは**大文字・小文字を区別**します。フィルターで使用する正確なサービス名を見つけるために、このリファレンスを利用してください。

## コンピューティング

| サービス | `serviceName` の値 |
|---------|-------------------|
| Virtual Machines | `Virtual Machines` |
| Azure Functions | `Functions` |
| Azure App Service | `Azure App Service` |
| Azure Container Apps | `Azure Container Apps` |
| Azure Container Instances | `Container Instances` |
| Azure Kubernetes Service | `Azure Kubernetes Service` |
| Azure Batch | `Azure Batch` |
| Azure Spring Apps | `Azure Spring Apps` |
| Azure VMware Solution | `Azure VMware Solution` |

## ストレージ

| サービス | `serviceName` の値 |
|---------|-------------------|
| Azure Storage (Blob, Files, Queues, Tables) | `Storage` |
| Azure NetApp Files | `Azure NetApp Files` |
| Azure Backup | `Backup` |
| Azure Data Box | `Data Box` |

> **注**: Blob Storage、Files、Disk Storage、Data Lake Storage はすべて単一の `Storage` サービス名に含まれます。これらを区別するには `meterName` または `productName` を使用してください（例: `contains(meterName, 'Blob')`）。

## データベース

| サービス | `serviceName` の値 |
|---------|-------------------|
| Azure Cosmos DB | `Azure Cosmos DB` |
| Azure SQL Database | `SQL Database` |
| Azure SQL Managed Instance | `SQL Managed Instance` |
| Azure Database for PostgreSQL | `Azure Database for PostgreSQL` |
| Azure Database for MySQL | `Azure Database for MySQL` |
| Azure Cache for Redis | `Redis Cache` |

## AI + 機械学習

| サービス | `serviceName` の値 |
|---------|-------------------|
| Azure AI Foundry Models (incl. OpenAI) | `Foundry Models` |
| Azure AI Foundry Tools | `Foundry Tools` |
| Azure Machine Learning | `Azure Machine Learning` |
| Azure Cognitive Search (AI Search) | `Azure Cognitive Search` |
| Azure Bot Service | `Azure Bot Service` |

> **注**: Azure OpenAI の価格は現在 `Foundry Models` に含まれています。OpenAI 固有のモデルで絞り込むには、`contains(productName, 'OpenAI')` または `contains(meterName, 'GPT')` を使用してください。

## ネットワーキング

| サービス | `serviceName` の値 |
|---------|-------------------|
| Azure Load Balancer | `Load Balancer` |
| Azure Application Gateway | `Application Gateway` |
| Azure Front Door | `Azure Front Door Service` |
| Azure CDN | `Azure CDN` |
| Azure DNS | `Azure DNS` |
| Azure Virtual Network | `Virtual Network` |
| Azure VPN Gateway | `VPN Gateway` |
| Azure ExpressRoute | `ExpressRoute` |
| Azure Firewall | `Azure Firewall` |

## 分析

| サービス | `serviceName` の値 |
|---------|-------------------|
| Azure Synapse Analytics | `Azure Synapse Analytics` |
| Azure Data Factory | `Azure Data Factory v2` |
| Azure Stream Analytics | `Azure Stream Analytics` |
| Azure Databricks | `Azure Databricks` |
| Azure Event Hubs | `Event Hubs` |

## 統合

| サービス | `serviceName` の値 |
|---------|-------------------|
| Azure Service Bus | `Service Bus` |
| Azure Logic Apps | `Logic Apps` |
| Azure API Management | `API Management` |
| Azure Event Grid | `Event Grid` |

## 管理と監視

| サービス | `serviceName` の値 |
|---------|-------------------|
| Azure Monitor | `Azure Monitor` |
| Azure Log Analytics | `Log Analytics` |
| Azure Key Vault | `Key Vault` |
| Azure Backup | `Backup` |

## Web

| サービス | `serviceName` の値 |
|---------|-------------------|
| Azure Static Web Apps | `Azure Static Web Apps` |
| Azure SignalR | `Azure SignalR Service` |

## ヒント

- サービス名が不明な場合は、まず**`serviceFamily` でフィルター**して、レスポンス内で有効な `serviceName` の値を確認してください。
- 例: `serviceFamily eq 'Databases' and armRegionName eq 'eastus'` は、すべてのデータベースサービス名を返します。
- 一部のサービスには、異なる階層や世代ごとに複数の `serviceName` エントリがあります。

