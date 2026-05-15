---
applyTo: '**/*.bicep,**/*.tf,**/*.tfvars,**/*.bicepparam,**/infra/**,**/infrastructure/**'
description: 'Microsoft CAF (Cloud Adoption Framework) に基づく Azure リソースの命名規則。 Azure リソースの名前を作成、確認、提案するときに使用します。'
---

# Azure リソース命名規則 (CAF)

出典: [命名規則を定義する](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/resource-naming) | [略語](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/resource-abbreviations) | [名前のルール](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/resource-name-rules)

Azure リソース名を作成、提案、または確認するときは、常に次のルールに従ってください。

---

## 一般的なパターン
```
<resource-type-abbr>-<workload>-<environment>-<region>-<instance>
```

**コンポーネントのルール:**
- **リソース タイプ** — 以下の表の公式略語を最初に使用します。
- **ワークロード / アプリ / プロジェクト** — 短いわかりやすい名前 (例: `navigator`、`payments`)
- **環境** — `prod`、`dev`、`qa`、`stage`、`test`
- **リージョン** — Azure リージョンの短縮名を使用します: `westus`、`eastus2`、`westeurope`、`northeurope`、`uksouth`、`southeastasia`、`australiaeast` など。
- **インスタンス** — ゼロ埋め数値: `001`、`002`

> 一部のリソース タイプはこのパターンから逸脱します (ハイフンは許可されないなど)。リソースごとのパターンと制約については、[公式の略語と命名規則](#official-abbreviations-and-naming-rules) を参照してください。

**一般的な文字ルール:**
- 小文字とハイフン (`-`) を使用してください。リソースタイプで必要な場合を除き、スペースやアンダースコアは使用できません。
- 一部のリソースでは **ハイフンを使用できません**。代わりに連結された小文字の英数字を使用します (表を参照)。
- `#`、`<`、`>`、`%`、`&`、`\`、`?`、`/` または制御文字は使用しないでください。
- 機密データ (サブスクリプション ID、テナント ID) を名前にエンコードしないでください。
- Azure ではほとんどの名前は **大文字と小文字を区別しません**。常に大文字と小文字を区別せずに比較されます。
- パブリック エンドポイントを持つリソースには、予約語や商標を含めることはできません。

---

## 命名範囲

|範囲 |意味 |
|------|-----------|
| **グローバル** | Azure 全体で一意 (パブリック エンドポイントを備えた PaaS) |
| **リソース グループ** |リソース グループ内で一意 |
| **リソース** |親リソース内で一意 |

---

## 公式の略語と命名規則

### 経営とガバナンス

|リソース |略語 |範囲 |長さ |有効な文字 |例 |
|----------|------|----------|----------|------|---------|
|管理グループ | `mg` |テナント | 1-90 |英数字、ハイフン、アンダースコア、ピリオド、括弧 | `mg-platform-prod` |
|リソースグループ | `rg` |購読 | 1-90 |アンダースコア、ハイフン、ピリオド、括弧、文字、数字 | `rg-navigator-prod` |
| Log Analytics ワークスペース | `log` |リソースグループ | 4-63 |英数字とハイフン | `log-navigator-prod-001` |
|アプリケーションインサイト | `appi` |リソースグループ | 1-260 |使用できません: `%&\?/` | `appi-navigator-prod-001` |
| Automation アカウント | `aa` |リソースグループ + リージョン | 6-50 |英数字とハイフン。文字 | で始まります。 `aa-navigator-prod-001` |

### ネットワーキング

|リソース |略語 |範囲 |長さ |有効な文字 |例 |
|----------|------|----------|----------|------|---------|
|仮想ネットワーク | `vnet` |リソースグループ | 2-64 |英数字、アンダースコア、ピリオド、ハイフン | `vnet-shared-eastus2-001` |
|サブネット | `snet` |仮想ネットワーク | 1-80 |英数字、アンダースコア、ピリオド、ハイフン | `snet-shared-eastus2-001` |
|ネットワーク セキュリティ グループ | `nsg` |リソースグループ | 1-80 |英数字、アンダースコア、ピリオド、ハイフン | `nsg-weballow-001` |
|アプリケーションセキュリティグループ | `asg` |リソースグループ | 1-80 |英数字、アンダースコア、ピリオド、ハイフン | `asg-navigator-prod-001` |
|ネットワークインターフェース | `nic` |リソースグループ | 1-80 |英数字、アンダースコア、ピリオド、ハイフン | `nic-01-vmnavigator-prod-001` |
|パブリック IP アドレス | `pip` |リソースグループ | 1-80 |英数字、アンダースコア、ピリオド、ハイフン | `pip-navigator-prod-westus-001` |
|ロードバランサー (内部) | `lbi` |リソースグループ | 1-80 |英数字、アンダースコア、ピリオド、ハイフン | `lbi-navigator-prod-001` |
|ロードバランサ（外部） | `lbe` |リソースグループ | 1-80 |英数字、アンダースコア、ピリオド、ハイフン | `lbe-navigator-prod-001` |
|アプリケーションゲートウェイ | `agw` |リソースグループ | 1-80 |英数字、アンダースコア、ピリオド、ハイフン | `agw-navigator-prod-001` |
|ファイアウォール | `afw` |リソースグループ | 1-80 |英数字、アンダースコア、ピリオド、ハイフン | `afw-navigator-prod-001` |
|ファイアウォール ポリシー | `afwp` |リソースグループ | 1-80 |英数字、アンダースコア、ピリオド、ハイフン | `afwp-navigator-prod-001` |
|ルートテーブル | `rt` |リソースグループ | 1-80 |英数字、アンダースコア、ピリオド、ハイフン | `rt-navigator-prod-001` |
|仮想ネットワークゲートウェイ | `vgw` |リソースグループ | 1-80 |英数字、アンダースコア、ピリオド、ハイフン | `vgw-shared-eastus2-001` |
| VPN ゲートウェイ | `vpng` |リソースグループ | 1-80 |英数字、アンダースコア、ピリオド、ハイフン | `vpng-navigator-prod-001` |
|アズール バスティオン | `bas` |リソースグループ | 1-80 |英数字、アンダースコア、ピリオド、ハイフン | `bas-navigator-prod-001` |
|プライベートエンドポイント | `pep` |リソースグループ | 2-64 |英数字、アンダースコア、ピリオド、ハイフン | `pep-navigator-prod-001` |
|トラフィック マネージャーのプロフィール | `traf` |グローバル | 1-63 |英数字とハイフン (ピリオドなし) | `traf-navigator-prod` |
| ExpressRoute 回線 | `erc` |リソースグループ | 1-80 |英数字、アンダースコア、ピリオド、ハイフン | `erc-navigator-prod-001` |
| CDN プロファイル | `cdnp` |リソースグループ | 1-260 |英数字とハイフン | `cdnp-navigator-prod-001` |
|フロントドアのプロフィール | `afd` |リソースグループ | 5-64 |英数字とハイフン | `afd-navigator-prod` |

### コンピューティングとウェブ

|リソース |略語 |範囲 |長さ |有効な文字 |例 |
|----------|------|----------|----------|------|---------|
|仮想マシン | `vm` |リソースグループ | 1-15 (Windows) / 1-64 (Linux) |スペースを使用しない場合、または: `~ ! @ # $ % ^ & * ( ) = + _ [ ] { } \| ; : . ' " , < > / ?` | `vm-sql-test-001` |
| VM スケール セット | `vmss` |リソースグループ | 1-15 (Windows) / 1-64 (Linux) | VM と同じ | `vmss-navigator-prod-001` |
|可用性セット | `avail` |リソースグループ | 1-80 |英数字、アンダースコア、ピリオド、ハイフン | `avail-navigator-prod-001` |
| App Service プラン | `asp` |リソースグループ | 1-60 |英数字、ハイフン、Unicode | `asp-navigator-prod-001` |
|ウェブアプリ | `app` |グローバル | 2-60 |英数字、ハイフン、Unicode。ハイフンで開始/終了することはできません。 | `app-navigator-prod-001` |
|関数アプリ | `func` |グローバル | 2-60 |英数字、ハイフン、Unicode。ハイフンで開始/終了することはできません。 | `func-navigator-prod-001` |
|静的 Web アプリ | `stapp` |リソースグループ | — | — | `stapp-navigator-prod-001` |
| App Service 環境 | `ase` |リソースグループ | — | — | `ase-navigator-prod-001` |

### コンテナ

|リソース |略語 |範囲 |長さ |有効な文字 |例 |
|----------|------|----------|----------|------|---------|
| AKS クラスター | `aks` |リソースグループ | 1-63 |英数字、アンダースコア、ハイフン | `aks-navigator-prod-001` |
| AKS システム ノード プール | `npsystem` |管理されたクラスター | 1-12 (Linux) / 1-6 (Windows) |小文字と数字、数字で始めることはできません | `npsystem` |
| AKS ユーザー ノード プール | `np` |管理されたクラスター | 1-12 (Linux) / 1-6 (Windows) |小文字と数字、数字で始めることはできません | `npusers` |
|コンテナアプリ | `ca` |リソースグループ | 2-32 |小文字、数字、ハイフン。文字で始まり、英数字で終わります。 | `ca-navigator-prod-001` |
|コンテナアプリ環境 | `cae` |リソースグループ | — | — | `cae-navigator-prod-001` |
|コンテナインスタンス | `ci` |リソースグループ | 1-63 |小文字、数字、ハイフン。ハイフンで開始/終了することはできません。 | `ci-navigator-prod-001` |
|コンテナレジストリ | `cr` |グローバル | 5-50 | **英数字のみ - ハイフンは使用できません** | `crnavigatorprod001` |

### データベース

|リソース |略語 |範囲 |長さ |有効な文字 |例 |
|----------|------|----------|----------|------|---------|
| Azure SQL サーバー | `sql` |グローバル | 1-63 |小文字、数字、ハイフン。ハイフンで開始/終了することはできません。 | `sql-navigator-prod-001` |
| Azure SQL データベース | `sqldb` | SQLサーバー | 1-128 |使用できません: `<>*%&:\/?` | `sqldb-navigator-prod` |
| SQL マネージド インスタンス | `sqlmi` |グローバル | 1-63 |小文字、数字、ハイフン。ハイフンで開始/終了することはできません。 | `sqlmi-navigator-prod-001` |
| Azure Cosmos DB | `cosmos` |グローバル | 3-44 |小文字、数字、ハイフン。小文字または数字で始めてください。 | `cosmos-navigator-prod` |
| Azure マネージド Redis | `amr` |グローバル | 1-63 |英数字とハイフン。英数字で開始/終了します。 | `amr-navigator-prod-001` |
| MySQLサーバー | `mysql` |グローバル | 3-63 |小文字、ハイフン、数字。ハイフンで開始/終了することはできません。 | `mysql-navigator-prod-001` |
| PostgreSQLサーバー | `psql` |グローバル | 3-63 |小文字、ハイフン、数字。ハイフンで開始/終了することはできません。 | `psql-navigator-prod-001` |

### ストレージ

|リソース |略語 |範囲 |長さ |有効な文字 |例 |
|----------|------|----------|----------|------|---------|
|ストレージ アカウント | `st` |グローバル | 3-24 | **小文字と数字のみ - ハイフンは使用できません** | `stnavigatorprod001` |
|バックアップボールト | `bvault` |リソースグループ | 2-50 |英数字とハイフン。まずは手紙から始めましょう。 | `bvault-navigator-prod-001` |

### 安全

|リソース |略語 |範囲 |長さ |有効な文字 |例 |
|----------|------|----------|----------|------|---------|
|キー コンテナー | `kv` |グローバル | 3-24 |英数字とハイフン。文字で始まり、文字または数字で終わります。連続したハイフンは使用できません。 | `kv-navigator-prod-001` |
|マネージド ID | `id` |リソースグループ | 3-128 |英数字、ハイフン、アンダースコア。文字または数字で始めてください。 | `id-navigator-prod-001` |

### 統合

|リソース |略語 |範囲 |長さ |有効な文字 |例 |
|----------|------|----------|----------|------|---------|
| API管理 | `apim` |グローバル | 1-50 |英数字とハイフン。文字で始まり、英数字で終わります。 | `apim-navigator-prod` |
| Service Bus 名前空間 | `sbns` |グローバル | 6-50 |英数字とハイフン。文字で始まり、文字または数字で終わります。 | `sbns-navigator-prod` |
|サービスバスのキュー | `sbq` |サービスバス | 1-260 |英数字、ピリオド、ハイフン、アンダースコア、スラッシュ | `sbq-navigator` |
|サービスバスのトピック | `sbt` |サービスバス | 1-260 |英数字、ピリオド、ハイフン、アンダースコア、スラッシュ | `sbt-navigator` |
| Event Hubs 名前空間 | `evhns` |グローバル | 6-50 |英数字とハイフン。文字で始まり、文字または数字で終わります。 | `evhns-navigator-prod` |
|イベントハブ | `evh` | Event Hubs 名前空間 | 1-256 |英数字、ピリオド、ハイフン、アンダースコア | `evh-navigator` |
|ロジック アプリ | `logic` |リソースグループ | 1-43 |英数字、ハイフン、アンダースコア、ピリオド | `logic-navigator-prod-001` |

### AIと機械学習

|リソース |略語 |範囲 |長さ |有効な文字 |例 |
|----------|------|----------|----------|------|---------|
| Azure OpenAI サービス | `oai` |リソースグループ | 2-64 |英数字とハイフン | `oai-navigator-prod` |
| AI検索 | `srch` |グローバル | — | — | `srch-navigator-prod` |
| Azure ML ワークスペース | `mlw` |リソースグループ | 3-33 |英数字、ハイフン、アンダースコア | `mlw-navigator-prod` |
|鋳造拠点 | `hub` |リソースグループ | 3-33 |英数字、ハイフン、アンダースコア | `hub-navigator-prod` |
|ファウンドリハブプロジェクト | `proj` |鋳造拠点 | 3-33 |英数字、ハイフン、アンダースコア | `proj-navigator-prod` |
|ファウンドリアカウント | `aif` |リソースグループ | 2-64 |英数字とハイフン | `aif-navigator-prod` |
|ファウンドリアカウントプロジェクト | `proj` |ファウンドリアカウント | — | — | `proj-navigator-prod` |
|ファウンドリ ツール (マルチサービス) | `ais` |リソースグループ | 2-64 |英数字とハイフン | `ais-navigator-prod` |

### 分析とIoT

|リソース |略語 |範囲 |長さ |有効な文字 |例 |
|----------|------|----------|----------|------|---------|
| Azure データファクトリー | `adf` |グローバル | 3-63 |英数字とハイフン。英数字で開始/終了します。 | `adf-navigator-prod` |
| Azure Databricks ワークスペース | `dbw` |リソースグループ | 3-64 |英数字、アンダースコア、ハイフン | `dbw-navigator-prod-001` |
| Azure データ エクスプローラー クラスター | `dec` |グローバル | 4-22 |小文字と数字。まずは手紙から始めましょう。 | `decnavigatorprod` |
| Azure Synapse ワークスペース | `synw` |グローバル | 1-50 |小文字、ハイフン、数字。文字または数字で開始/終了します。 | `synw-navigator-prod` |
| IoTハブ | `iot` |グローバル | 3-50 |英数字とハイフン。ハイフンで終わることはできません。 | `iot-navigator-prod` |
| Event Grid トピック | `evgt` |地域 | 3-50 |英数字とハイフン | `evgt-navigator-prod` |

### 開発者ツール

|リソース |略語 |範囲 |長さ |有効な文字 |例 |
|----------|------|----------|----------|------|---------|
|アプリ構成ストア | `appcs` |グローバル | 5-50 |英数字とハイフン。連続するハイフンは 2 つまでです。 | `appcs-navigator-prod` |
|シグナルR | `sigr` |グローバル | 3-63 |英数字とハイフン。文字で始まり、文字または数字で終わります。 | `sigr-navigator-prod` |

---

## ハイフンを使用できないリソース

これらのリソースには、連結された小文字の英数字 (区切り文字なし) が必要です。

|リソース |略語 |パターン |
|----------|------|----------|
|ストレージ アカウント | `st` | `st{workload}{env}{instance}` → `stnavigatorprod001` |
|コンテナレジストリ | `cr` | `cr{workload}{env}{instance}` → `crnavigatorprod001` |
| Azure データ エクスプローラー クラスター | `dec` | `dec{workload}{env}` → `decnavigatorprod` |

---

## 例（CAF）
```
# Management
rg-navigator-prod
rg-webapp-database-dev

# Networking
vnet-shared-eastus2-001
snet-shared-eastus2-001
nsg-weballow-001
pip-dc1-shared-eastus2-001
lbe-navigator-prod-001

# Compute
vm-sql-test-001
vm-sharepoint-dev-001
vmss-navigator-prod-001
asp-navigator-prod-001
app-navigator-prod-001
func-navigator-prod-001

# Containers
aks-navigator-prod-001
ca-navigator-prod-001
cae-navigator-prod-001
crnavigatorprod001        # no hyphens!

# Databases
sql-navigator-prod-001
sqldb-navigator-prod
cosmos-navigator-prod
psql-navigator-prod-001

# Storage / Security
stnavigatorprod001        # no hyphens!
kv-navigator-prod-001
id-navigator-prod-001

# Integration
apim-navigator-prod
sbns-navigator-prod
evhns-navigator-prod

# Monitoring
log-navigator-prod-001
appi-navigator-prod-001

# AI
oai-navigator-prod
srch-navigator-prod
```

---

## してはいけないこと

- リソースタイプで必要な場合を除き、アンダースコアは使用しないでください。ハイフンを使用してください。
- リソースタイプの単語全体をスペルアウトしないでください (例: `storageaccount-myapp` → `stmyapp001` を使用)。
- 大文字は使用しないでください (リソースでは大文字と小文字が区別されません。慣例として小文字が使用されます)。
- 名前には機密データ (サブスクリプション ID、テナント ID、パスワード) を含めないでください。
- 実稼働環境であっても、環境セグメントをスキップしないでください。
- `#` は使用しないでください。Azure Resource Manager での URL 解析が中断されます。
- パブリック エンドポイントを持つリソースの名前には予約語や商標を使用しないでください。
- 2 つ以上の連続したハイフンを使用しないでください (例: `app--prod` は無効です)。