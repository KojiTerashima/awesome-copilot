---
name: azure-pricing
description: 'Azure Retail Prices API（prices.azure.com）を使用して Azure の小売価格をリアルタイムで取得し、Copilot Studio エージェントのクレジット消費を見積もります。Azure サービスのコストについて質問された場合、SKU 価格を比較したい場合、コスト見積もり用の価格データが必要な場合、Azure pricing / Azure costs / Azure billing への言及がある場合、または Copilot Studio pricing、Copilot Credits、エージェント利用量見積もりについて質問された場合に使用します。コンピュート、ストレージ、ネットワーク、データベース、AI、Copilot Studio、およびその他すべての Azure サービス ファミリーをカバーします。'
compatibility: prices.azure.com および learn.microsoft.com へのインターネットアクセスが必要です。認証は不要です。
metadata:
  author: anthonychu
  version: "1.2"
---

# Azure Pricing Skill

このスキルを使うと、公開されている Azure Retail Prices API から Azure の小売価格データをリアルタイムで取得できます。認証は不要です。

## このスキルを使うタイミング

- ユーザーが Azure サービスのコストを尋ねている（例: 「D4s v5 VM の料金はいくらですか？」）
- ユーザーがリージョン間または SKU 間で価格比較をしたい
- ユーザーがワークロードやアーキテクチャのコスト見積もりを必要としている
- ユーザーが Azure pricing、Azure costs、または Azure billing に言及している
- ユーザーが予約インスタンスと従量課金の価格差を尋ねている
- ユーザーが Savings Plan やスポット価格について知りたい

## API エンドポイント

```
GET https://prices.azure.com/api/retail/prices?api-version=2023-01-01-preview
```

OData フィルター構文で `$filter` をクエリパラメーターとして追加してください。Savings Plan データを含めるため、必ず `api-version=2023-01-01-preview` を使用します。

## 手順

ユーザーの要望に不明点がある場合は、API を呼び出す前に補足質問をして、正しいフィルターフィールドと値を特定してください。

1. ユーザーの要望から**フィルターフィールド**（サービス名、リージョン、SKU、価格タイプ）を特定する。
2. **リージョンを解決**する: API では `armRegionName` に、小文字かつスペースなしの値が必要です（例: "East US" → `eastus`, "West Europe" → `westeurope`, "Southeast Asia" → `southeastasia`）。完全な一覧は [references/REGIONS.md](references/REGIONS.md) を参照。
3. 以下のフィールドで**フィルター文字列を作成**し、URL を取得する。
4. JSON レスポンスの **`Items` 配列を解析**する。各 item には価格とメタデータが含まれる。
5. 最初の 1000 件を超える結果が必要な場合は、`NextPageLink` で**ページネーションを追う**（通常は不要）。
6. [references/COST-ESTIMATOR.md](references/COST-ESTIMATOR.md) の式を使って**コスト見積もりを計算**し、月額/年額見積もりを作成する。
7. サービス、SKU、リージョン、単価、月額/年額見積もりを含む明確なサマリーテーブルで**結果を提示**する。

## フィルター可能なフィールド

| Field | Type | Example |
|---|---|---|
| `serviceName` | string（完全一致、大小文字を区別） | `'Functions'`, `'Virtual Machines'`, `'Storage'` |
| `serviceFamily` | string（完全一致、大小文字を区別） | `'Compute'`, `'Storage'`, `'Databases'`, `'AI + Machine Learning'` |
| `armRegionName` | string（完全一致、小文字） | `'eastus'`, `'westeurope'`, `'southeastasia'` |
| `armSkuName` | string（完全一致） | `'Standard_D4s_v5'`, `'Standard_LRS'` |
| `skuName` | string（contains 対応） | `'D4s v5'` |
| `priceType` | string | `'Consumption'`, `'Reservation'`, `'DevTestConsumption'` |
| `meterName` | string（contains 対応） | `'Spot'` |

等価比較には `eq`、条件結合には `and`、部分一致には `contains(field, 'value')` を使用します。

## フィルター文字列の例

```
# East US の Functions における従量課金価格をすべて取得
serviceName eq 'Functions' and armRegionName eq 'eastus' and priceType eq 'Consumption'

# West Europe の D4s v5 VM（従量課金のみ）
armSkuName eq 'Standard_D4s_v5' and armRegionName eq 'westeurope' and priceType eq 'Consumption'

# あるリージョンのストレージ価格をすべて取得
serviceName eq 'Storage' and armRegionName eq 'eastus'

# 特定 SKU のスポット価格
armSkuName eq 'Standard_D4s_v5' and contains(meterName, 'Spot') and armRegionName eq 'eastus'

# 1 年予約価格
serviceName eq 'Virtual Machines' and priceType eq 'Reservation' and armRegionName eq 'eastus'

# Azure AI / OpenAI の価格（現在は Foundry Models 配下）
serviceName eq 'Foundry Models' and armRegionName eq 'eastus' and priceType eq 'Consumption'

# Azure Cosmos DB の価格
serviceName eq 'Azure Cosmos DB' and armRegionName eq 'eastus' and priceType eq 'Consumption'
```

## 完全な取得 URL の例

```
https://prices.azure.com/api/retail/prices?api-version=2023-01-01-preview&$filter=serviceName eq 'Functions' and armRegionName eq 'eastus' and priceType eq 'Consumption'
```

URL を構築する際は、スペースを `%20`、引用符を `%27` に URL エンコードしてください。

## レスポンスの主要フィールド

```json
{
  "Items": [
    {
      "retailPrice": 0.000016,
      "unitPrice": 0.000016,
      "currencyCode": "USD",
      "unitOfMeasure": "1 Execution",
      "serviceName": "Functions",
      "skuName": "Premium",
      "armRegionName": "eastus",
      "meterName": "vCPU Duration",
      "productName": "Functions",
      "priceType": "Consumption",
      "isPrimaryMeterRegion": true,
      "savingsPlan": [
        { "unitPrice": 0.000012, "term": "1 Year" },
        { "unitPrice": 0.000010, "term": "3 Years" }
      ]
    }
  ],
  "NextPageLink": null,
  "Count": 1
}
```

ユーザーが非プライマリメーターを明示的に求めない限り、`isPrimaryMeterRegion` が `true` の item のみを使用してください。

## サポートされる serviceFamily の値

`Analytics`, `Compute`, `Containers`, `Data`, `Databases`, `Developer Tools`, `Integration`, `Internet of Things`, `Management and Governance`, `Networking`, `Security`, `Storage`, `Web`, `AI + Machine Learning`

## ヒント

- `serviceName` は大小文字を区別します。不確かな場合は、まず `serviceFamily` で絞り込み、結果から有効な `serviceName` を確認してください。
- 結果が空の場合は、フィルターを広げてください（例: まず `priceType` やリージョン制約を外す）。
- `currencyCode` をリクエストで指定しない限り、価格は常に USD です。
- Savings Plan 価格は、各 item の `savingsPlan` 配列を確認してください（`2023-01-01-preview` のみ）。
- よく使うサービス名と正しい大文字小文字は [references/SERVICE-NAMES.md](references/SERVICE-NAMES.md) を参照。
- コスト見積もりの式とパターンは [references/COST-ESTIMATOR.md](references/COST-ESTIMATOR.md) を参照。
- Copilot Studio の課金レートと見積もり式は [references/COPILOT-STUDIO-RATES.md](references/COPILOT-STUDIO-RATES.md) を参照。

## トラブルシューティング

| Issue | Solution |
|-------|----------|
| 結果が空 | フィルターを広げる — まず `priceType` または `armRegionName` を外す |
| サービス名が誤っている | `serviceFamily` フィルターで有効な `serviceName` を確認する |
| Savings Plan データがない | URL に `api-version=2023-01-01-preview` があることを確認する |
| URL エラー | URL エンコードを確認する — スペースは `%20`、引用符は `%27` |
| 結果が多すぎる | フィルターフィールド（リージョン、SKU、priceType）を追加して絞り込む |

---

# Copilot Studio エージェント利用量見積もり

ユーザーが Copilot Studio の価格、Copilot Credits、またはエージェント利用コストについて尋ねた場合は、このセクションを使用してください。

## このセクションを使うタイミング

- ユーザーが Copilot Studio の価格またはコストについて尋ねている
- ユーザーが Copilot Credits またはエージェントのクレジット消費について尋ねている
- ユーザーが Copilot Studio エージェントの月額コストを見積もりたい
- ユーザーがエージェント利用量見積もりや Copilot Studio estimator に言及している
- ユーザーがエージェント運用にかかる費用を尋ねている

## 重要な事実

- **1 Copilot Credit = 0.01 USD**
- クレジットはテナント全体でプールされる
- M365 Copilot ライセンスユーザー向けの従業員向けエージェントは、クラシック回答、生成回答、テナントグラフ グラウンディングを無料で利用可能
- 超過利用の強制は、前払い容量の 125% で発動する

## 見積もり手順

1. ユーザーから**入力情報を収集**する: エージェント種別（従業員/顧客）、ユーザー数、月間インタラクション数、ナレッジ割合、テナントグラフ割合、セッションごとのツール利用量。
2. **最新の課金レートを取得**する — 組み込みの web fetch ツールを使い、下記 URL から最新レートを取得する。これにより、常に最新の Microsoft 価格で見積もりできる。
3. **取得したコンテンツを解析**し、現在の課金レート表（機能タイプごとのクレジット）を抽出する。
4. 取得したコンテンツ内のレートと式を使って**見積もりを計算**する:
   - `total_sessions = users × interactions_per_month`
   - ナレッジクレジット: テナントグラフ グラウンディング率、生成回答率、クラシック回答率を適用
   - エージェントツールクレジット: ツール呼び出しごとのエージェントアクション率を適用
   - エージェントフロークレジット: 100 アクションあたりのフロー率を適用
   - プロンプト修飾子クレジット: 10 応答あたりの basic/standard/premium レートを適用
5. カテゴリ別内訳、合計クレジット、推定 USD コストを含む明確な表で**結果を提示**する。

## 取得すべきソース URL

Copilot Studio の価格に関する質問に回答する際は、次の URL から最新コンテンツを取得してコンテキストとして使用してください。

| URL | Content |
|---|---|
| https://learn.microsoft.com/en-us/microsoft-copilot-studio/requirements-messages-management | 課金レート表、課金例、超過利用の強制ルール |
| https://learn.microsoft.com/en-us/microsoft-copilot-studio/billing-licensing | ライセンスオプション、M365 Copilot の含有内容、前払いと従量課金 |

計算前に、少なくとも最初の URL（課金レート）は取得してください。2 つ目の URL はライセンス関連の質問に対する補足コンテキストです。

レート、式、課金例のキャッシュスナップショットは [references/COPILOT-STUDIO-RATES.md](references/COPILOT-STUDIO-RATES.md) を参照してください（web fetch が使えない場合のフォールバックとして使用）。

<system_reminder>
<sql_tables>No tables currently exist. Default tables (todos, todo_deps) will be created automatically when using the SQL tool for the first time.</sql_tables>
</system_reminder>

