# ドメインパック: AI/Data (v1)

Azure AI/Data ワークロードに特化したサービス構成ガイドです。  
v1 のスコープ: Foundry、AI Search、ADLS Gen2、Key Vault、Fabric、ADF、VNet/PE。

> 必須プロパティ / よくあるミス → `service-gotchas.md`  
> 動的情報（API バージョン、SKU、リージョン）→ `azure-dynamic-sources.md`  
> 共通パターン（PE、セキュリティ、命名）→ `azure-common-patterns.md`

---

## 1. Microsoft Foundry (CognitiveServices)

### リソース階層

```
Microsoft.CognitiveServices/accounts (kind: 'AIServices')
├── /projects          — Foundry Project（ポータルアクセスに必須）
└── /deployments       — モデル デプロイメント（GPT-4o、embedding など）
```

### Bicep コア構成

```bicep
// Foundry リソース
resource foundry 'Microsoft.CognitiveServices/accounts@<fetch>' = {
  name: foundryName
  location: location
  kind: 'AIServices'
  sku: { name: '<confirm with user>' }               // ← SKU は Phase 1 で MS Docs を確認後に確定
  identity: { type: 'SystemAssigned' }
  properties: {
    customSubDomainName: foundryName  // ← 必須・グローバル一意。作成後は変更不可。省略した場合は削除して再作成が必要
    allowProjectManagement: true
    publicNetworkAccess: 'Disabled'
    networkAcls: { defaultAction: 'Deny' }
  }
}

// Foundry Project — Foundry とセットで必ず作成
resource project 'Microsoft.CognitiveServices/accounts/projects@<fetch>' = {
  parent: foundry
  name: '${foundryName}-project'
  location: location
  sku: { name: '<same as parent>' }
  kind: 'AIServices'
  identity: { type: 'SystemAssigned' }
  properties: {}
}

// モデル デプロイメント — Foundry リソース階層で作成
resource deployment 'Microsoft.CognitiveServices/accounts/deployments@<fetch>' = {
  parent: foundry
  name: '<model-name>'                              // ← Phase 1 でユーザー確認済み
  sku: {
    name: '<deployment-type>'                        // ← GlobalStandard、Standard など — MS Docs で取得
    capacity: <confirm with user>                    // ← 容量ユニット — MS Docs で利用可能範囲を確認
  }
  properties: {
    model: {
      format: 'OpenAI'
      name: '<model-name>'                           // ← 利用可否の確認が必須（fetch）
      version: '<fetch>'                             // ← バージョンも取得する
    }
  }
}
```

> `@<fetch>`: `azure-dynamic-sources.md` の URL から API バージョンを確認してください。  
> モデル名 / バージョン / デプロイタイプ / capacity: すべて動的項目 — Phase 1 で MS Docs 取得後、ユーザー確認で確定。

---

## 2. Azure AI Search

### Bicep コア構成

```bicep
resource search 'Microsoft.Search/searchServices@<fetch>' = {
  name: searchName
  location: location
  sku: { name: '<confirm with user>' }
  identity: { type: 'SystemAssigned' }
  properties: {
    hostingMode: 'default'
    publicNetworkAccess: 'disabled'
    semanticSearch: '<confirm with user>'    // disabled | free | standard — MS Docs で確認
  }
}
```

### 設計メモ

- PE サポート: Basic SKU 以上（最新の制約を MS Docs で確認）
- Semantic Ranker: `semanticSearch` プロパティで有効化（`disabled` | `free` | `standard`）— SKU ごとの対応を MS Docs で確認
- ベクター検索: 有償 SKU でサポート（MS Docs で確認）
- RAG 構成では Foundry と併用されることが多い

---

## 3. ADLS Gen2 (Storage Account)

### Bicep コア構成

```bicep
resource storage 'Microsoft.Storage/storageAccounts@<fetch>' = {
  name: storageName        // 小文字 + 数字のみ、ハイフン不可
  location: location
  kind: 'StorageV2'
  sku: { name: 'Standard_LRS' }
  properties: {
    isHnsEnabled: true                 // ← 絶対に省略しない
    accessTier: 'Hot'
    allowBlobPublicAccess: false
    minimumTlsVersion: 'TLS1_2'
    publicNetworkAccess: 'Disabled'
    networkAcls: { defaultAction: 'Deny' }
  }
}

// Container
resource container 'Microsoft.Storage/storageAccounts/blobServices/containers@<fetch>' = {
  name: '${storage.name}/default/raw'
}
```

### 設計メモ

- `isHnsEnabled` は作成後に変更不可 → 省略した場合はリソース再作成が必要
- PE: ユースケースに応じて `blob` と `dfs` の両方の PE が必要な場合がある
- よく使うコンテナー: `raw`、`processed`、`curated`

---

## 4. Microsoft Fabric

### Bicep コア構成

```bicep
resource fabric 'Microsoft.Fabric/capacities@<fetch>' = {
  name: fabricName
  location: location
  sku: { name: '<confirm with user>', tier: 'Fabric' }
  properties: {
    administration: {
      members: [ '<admin-email>' ]    // ← 必須。ないとデプロイ失敗
    }
  }
}
```

### 設計メモ

- Bicep でプロビジョニングできるのは Capacity のみ
- Workspace、Lakehouse、Warehouse などはポータルで手動作成が必要
- 管理者メールはユーザー確認（`ask_user`）

### Phase 1 で追加する際の必須確認項目

会話中に Fabric を追加する場合、図更新前に以下を ask_user で必ず確認すること:

- [ ] **SKU/Capacity**: F2、F4、F8、... — MS Docs から利用可能 SKU を取得後、選択肢として提示
- [ ] **administration.members**: 管理者メール — 未設定だとデプロイ失敗

> ユーザーが指定していないサブワークロード（OneLake、data pipelines、Warehouse など）を勝手に含めないこと。Bicep でプロビジョニング可能なのは Capacity のみです。

---

## 5. Azure Data Factory

### Bicep コア構成

```bicep
resource adf 'Microsoft.DataFactory/factories@<fetch>' = {
  name: adfName
  location: location
  identity: { type: 'SystemAssigned' }
  properties: {
    publicNetworkAccess: 'Disabled'
  }
}
```

### 設計メモ

- Self-hosted Integration Runtime は Bicep 外で手動セットアップが必要
- 主にオンプレミス データ取り込みシナリオで使用
- PE groupId: `dataFactory`

---

## 6. AML / AI Hub (MachineLearningServices)

### 利用する条件

```
Decision Rule:
├─ 一般的な AI/RAG → Foundry（AIServices）を使用
└─ ML 学習やオープンソースモデルが必要 → AI Hub を検討
    └─ ユーザーが明示的に要求した場合のみ
```

### Bicep コア構成

```bicep
resource hub 'Microsoft.MachineLearningServices/workspaces@<fetch>' = {
  name: hubName
  location: location
  kind: 'Hub'
  sku: { name: '<confirm with user>', tier: '<confirm with user>' }  // 例: Basic/Basic — 利用可能 SKU を MS Docs で確認
  identity: { type: 'SystemAssigned' }
  properties: {
    friendlyName: hubName
    storageAccount: storage.id
    keyVault: keyVault.id
    applicationInsights: appInsights.id    // Hub では必須
    publicNetworkAccess: 'Disabled'
  }
}
```

### AI Hub の依存関係

Hub 利用時に追加で必要なリソース:
- Storage Account
- Key Vault
- Application Insights + Log Analytics Workspace
- Container Registry（任意）

---

## 7. 共通 AI/Data アーキテクチャ構成例

### RAG チャットボット

```
Foundry (AIServices) + Project
├── <chat-model> (chat)              — Phase 1 の利用可否確認後に確定
├── <embedding-model> (embedding)    — Phase 1 の利用可否確認後に確定
├── AI Search (vector + semantic)
├── ADLS Gen2 (document store)
└── Key Vault (secrets)
+ VNet/PE のフル構成
```

### データプラットフォーム

```
Fabric Capacity (analytics)
├── ADLS Gen2 (data lake)
├── ADF (ingestion)
└── Key Vault (secrets)
+ VNet/PE 構成
```

