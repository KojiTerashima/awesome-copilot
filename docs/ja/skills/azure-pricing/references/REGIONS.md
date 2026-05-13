# Azure リージョン名リファレンス

Azure Retail Prices API では、`armRegionName` の値はスペースなしの小文字である必要があります。この表を使って、一般的なリージョン名を API の値に対応付けてください。

## リージョン対応表

| 表示名 | armRegionName |
|-------------|---------------|
| East US | `eastus` |
| East US 2 | `eastus2` |
| Central US | `centralus` |
| North Central US | `northcentralus` |
| South Central US | `southcentralus` |
| West Central US | `westcentralus` |
| West US | `westus` |
| West US 2 | `westus2` |
| West US 3 | `westus3` |
| Canada Central | `canadacentral` |
| Canada East | `canadaeast` |
| Brazil South | `brazilsouth` |
| North Europe | `northeurope` |
| West Europe | `westeurope` |
| UK South | `uksouth` |
| UK West | `ukwest` |
| France Central | `francecentral` |
| France South | `francesouth` |
| Germany West Central | `germanywestcentral` |
| Germany North | `germanynorth` |
| Switzerland North | `switzerlandnorth` |
| Switzerland West | `switzerlandwest` |
| Norway East | `norwayeast` |
| Norway West | `norwaywest` |
| Sweden Central | `swedencentral` |
| Italy North | `italynorth` |
| Poland Central | `polandcentral` |
| Spain Central | `spaincentral` |
| East Asia | `eastasia` |
| Southeast Asia | `southeastasia` |
| Japan East | `japaneast` |
| Japan West | `japanwest` |
| Australia East | `australiaeast` |
| Australia Southeast | `australiasoutheast` |
| Australia Central | `australiacentral` |
| Korea Central | `koreacentral` |
| Korea South | `koreasouth` |
| Central India | `centralindia` |
| South India | `southindia` |
| West India | `westindia` |
| UAE North | `uaenorth` |
| UAE Central | `uaecentral` |
| South Africa North | `southafricanorth` |
| South Africa West | `southafricawest` |
| Qatar Central | `qatarcentral` |

## 変換ルール

1. すべてのスペースを削除する
2. 小文字に変換する
3. 例:
   - "East US" → `eastus`
   - "West Europe" → `westeurope`
   - "Southeast Asia" → `southeastasia`
   - "South Central US" → `southcentralus`

## よくある別名

ユーザーがリージョンを非公式な呼び方で指定する場合があります。これらを正しい `armRegionName` に対応付けてください:

| ユーザーの呼び方 | 対応先 |
|-----------|---------|
| "US East", "Virginia" | `eastus` |
| "US West", "California" | `westus` |
| "Europe", "EU" | `westeurope` (既定) |
| "UK", "London" | `uksouth` |
| "Asia", "Singapore" | `southeastasia` |
| "Japan", "Tokyo" | `japaneast` |
| "Australia", "Sydney" | `australiaeast` |
| "India", "Mumbai" | `centralindia` |
| "Korea", "Seoul" | `koreacentral` |
| "Brazil", "São Paulo" | `brazilsouth` |
| "Canada", "Toronto" | `canadacentral` |
| "Germany", "Frankfurt" | `germanywestcentral` |
| "France", "Paris" | `francecentral` |
| "Sweden", "Stockholm" | `swedencentral` |

