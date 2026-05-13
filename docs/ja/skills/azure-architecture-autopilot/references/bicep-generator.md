# Bicep Generator Agent

Phase 1 で確定した最終アーキテクチャ仕様を受け取り、デプロイ可能な Bicep テンプレートを生成します。

## ステップ 0: 最新仕様の検証（Bicep 生成前に必須）

Bicep コードに API バージョンをハードコードしないでください。  
使用予定のサービスについては必ず MS Docs の Bicep リファレンスを取得し、使用前に最新の安定版 `apiVersion` を確認してください。

### 検証手順
1. 使用するサービス一覧を特定する
2. 各サービスの MS Docs URL を取得する（web_fetch ツールを使用）
3. ページ上で最新の安定版 API バージョンを確認する
4. そのバージョンで Bicep を記述する

### モデル デプロイ可用性チェック（Foundry/OpenAI モデル使用時に必須）

**Bicep を生成する前に**、ユーザーが指定したモデル名が対象リージョンで実際にデプロイ可能かを確認してください。  
モデルの可用性はリージョンごとに異なり、頻繁に変わるため、固定知識に依存してはいけません。

**検証方法（優先順）:**
1. MS Docs のモデル可用性ページを確認: https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/models
2. または Azure CLI で直接照会:
   ```powershell
   az cognitiveservices account list-models --name "<FOUNDRY_NAME>" --resource-group "<RG_NAME>" -o table
   ```
   （Foundry リソースがすでに存在する場合）

**モデルが対象リージョンで利用できない場合:**
- ユーザーに通知し、利用可能なリージョンまたは代替モデルを提案する
- ユーザー承認なしで別モデルや別リージョンに置き換えない

### サービス別 MS Docs URL

URL の完全なレジストリは `references/azure-dynamic-sources.md` にあります。取得時はこのファイルを参照してください。  
参照ファイルは `.github/skills/azure-architecture-autopilot/` 配下にあります。

> **重要**: 最新の安定版 `apiVersion` を確認するため、必ず web_fetch で URL から直接取得してください。参照ファイルや過去の会話にあるハードコード済みバージョンを盲目的に使わないでください。

> **子リソースも必ず検証**: 親リソースのページで子リソース（accounts/projects、accounts/deployments、privateDnsZones/virtualNetworkLinks、privateEndpoints/privateDnsZoneGroups など）の API バージョンも確認してください。親と子で API バージョンが異なる場合があります。

> **エラー/警告時も同じ原則**: what-if やデプロイ時に API バージョン関連のエラーが出ても、エラーメッセージ内のバージョンを「最新」としてそのまま適用してはいけません。修正前に必ず MS Docs の URL を再取得し、実際の最新安定版を確認してください。

---

## 情報参照の原則（Stable vs Dynamic）

### 常に取得する（Dynamic）
- API バージョン → `azure-dynamic-sources.md` の URL から取得
- モデル可用性（name, version, region）→ 取得
- SKU 一覧/価格 → 取得
- リージョン可用性 → 取得

### まず参照する（Stable）
- 必須プロパティパターン（`isHnsEnabled`、`allowProjectManagement` など）→ `service-gotchas.md`
- PE groupId と DNS Zone 対応（主要サービス）→ `service-gotchas.md`
- PE/セキュリティ/命名の共通パターン → `azure-common-patterns.md`
- AI/Data サービス構成ガイド → `ai-data.md`

> 安定情報に不安がある場合は MS Docs で再検証してください。ただし毎回取得する必要はありません。

---

## 未知サービス時のフォールバック ワークフロー

ユーザーが v1 スコープ（`ai-data.md`）に含まれないサービスを要求した場合:

1. **ユーザーへ通知**: 「このサービスは v1 のデフォルトスコープ外です。MS Docs を参照したベストエフォートで生成します。」
2. **API バージョン取得**: `https://learn.microsoft.com/en-us/azure/templates/microsoft.{provider}/{resourceType}` 形式で URL を組み立てて取得
3. **リソース種別/必須プロパティ特定**: 取得した Docs からリソース種別と必須プロパティを確認
4. **PE マッピング検証**: `https://learn.microsoft.com/en-us/azure/private-link/private-endpoint-dns` を取得して groupId/DNS Zone を確認
5. **共通パターン適用**: `azure-common-patterns.md` のセキュリティ/ネットワーク/命名パターンを適用
6. **Bicep 記述**: 上記情報に基づいてモジュールを生成
7. **レビュー担当へ引き継ぎ**: `az bicep build` でコンパイル検証

## 入力情報

Phase 1 完了時点で、以下の情報が確定している必要があります:

```
- services: [サービス一覧 + SKU]
- networking: private_endpoint を使用するか
- resource_group: リソース グループ名
- location: デプロイ先リージョン（Phase 1 でユーザー確認済み）
- subscription_id: Azure サブスクリプション ID
```

## 出力ファイル構成

```
<project-name>/
├── main.bicep              # メイン オーケストレーション — モジュール呼び出しとパラメーター受け渡し
├── main.bicepparam         # パラメーターファイル — 環境固有値（機密情報を除く）
└── modules/
    ├── network.bicep           # VNet、Subnet（pe-subnet を含む）
    ├── ai.bicep                # AI サービス（ユーザー要件に応じて構成）
    ├── storage.bicep           # ADLS Gen2（isHnsEnabled: true 必須）
    ├── fabric.bicep            # Microsoft Fabric Capacity（必要時のみ）
    ├── keyvault.bicep          # Key Vault
    ├── monitoring.bicep        # Application Insights、Log Analytics（Hub ベース構成時のみ必要）
    └── private-endpoints.bicep # すべての PE + Private DNS Zones + VNet Links + DNS Zone Groups
```

## モジュール責務

### `network.bicep`
- VNet — CIDR はパラメーターで受け取る（顧客環境の既存アドレス空間との競合を回避）
- pe-subnet — `privateEndpointNetworkPolicies: 'Disabled'` が必須
- 追加サブネットは必要に応じてパラメーターで対応

### `ai.bicep`
- **Microsoft Foundry リソース**（`Microsoft.CognitiveServices/accounts`, `kind: 'AIServices'`）— 最上位 AI リソース
  - `customSubDomainName: foundryName` 必須 — **作成後に変更不可。省略した場合はリソース削除・再作成が必要**
  - `identity: { type: 'SystemAssigned' }` 必須
  - `allowProjectManagement: true` 必須
  - モデル デプロイ（`Microsoft.CognitiveServices/accounts/deployments`）— Foundry リソース階層で実施
- **⚠️ Foundry Project**（`Microsoft.CognitiveServices/accounts/projects`）— **必ず子リソースとして作成**
  - リソース種別: `Microsoft.CognitiveServices/accounts/projects`（単独の `accounts` リソースとしては絶対に作成しない）
  - Bicep で `parent: foundryAccount` を使用
  - 誤り例: Project を別の `kind: 'AIServices'` アカウントとして作成 → ポータルで認識されない
  - 正しい例:
    ```bicep
    resource foundryProject 'Microsoft.CognitiveServices/accounts/projects@<apiVersion>' = {
      parent: foundryAccount
      name: 'project-${uniqueString(resourceGroup().id)}'
      location: location
      kind: 'AIServices'
      properties: {}
    }
    ```
- **Azure AI Search** — Semantic Ranking、ベクター検索設定
- Hub ベース（`Microsoft.MachineLearningServices/workspaces`）は、ユーザーが明示的に要求した場合、または ML 学習/オープンソースモデルが必要な場合のみ検討。標準的な AI/RAG ワークロードでは Foundry（AIServices）を既定選択とする

**⛔ CognitiveServices の禁止プロパティ:**
- `apiProperties.statisticsEnabled` — このプロパティは存在しません。絶対に使用しないでください。デプロイ時に `ApiPropertiesInvalid` エラーになります
- `apiProperties.qnaAzureSearchEndpointId` — QnA Maker 専用。Foundry では使用しない
- 未検証のプロパティを `properties.apiProperties` に恣意的に追加しない

### `storage.bicep`
- ADLS Gen2: `isHnsEnabled: true` ← **絶対に省略しない**
- Containers: raw, processed, curated（または要件に応じる）
- `allowBlobPublicAccess: false`, `minimumTlsVersion: 'TLS1_2'`

### `keyvault.bicep`
- `enableRbacAuthorization: true`（access policy モデルは使用しない）
- `enableSoftDelete: true`, `softDeleteRetentionInDays: 90`
- `enablePurgeProtection: true`

### `monitoring.bicep`
- Log Analytics Workspace
- Application Insights（Hub ベース構成時のみ必要 — Foundry AIServices では不要）

### `private-endpoints.bicep`
- サービスごとに 3 点セット:
  1. `Microsoft.Network/privateEndpoints`（pe-subnet に配置）
  2. `Microsoft.Network/privateDnsZones` + VNet Link（`registrationEnabled: false`）
  3. `Microsoft.Network/privateEndpoints/privateDnsZoneGroups`
- サービス別 DNS Zone 対応は `references/service-gotchas.md` を参照

**⚠️ Foundry/AIServices PE DNS ルール:**
- PE groupId: `account`
- DNS Zone Group には **2 つのゾーン** が必要:
  1. `privatelink.cognitiveservices.azure.com`
  2. `privatelink.openai.azure.com`
- 片方だけだと OpenAI API 呼び出しの DNS 解決に失敗 → 接続エラー

**⚠️ ADLS Gen2（isHnsEnabled: true）PE ルール:**
- 2 つの PE が必要:
  1. `blob` → `privatelink.blob.core.windows.net`
  2. `dfs` → `privatelink.dfs.core.windows.net`
- DFS PE がないと、Data Lake 操作（ファイルシステム作成、ディレクトリ操作）が失敗する

### `rbac.bicep`（または main.bicep にインライン記述）

**⚠️ RBAC ロール割り当て — 絶対に省略しない**

**Managed Identity（`identity.type: 'SystemAssigned'`）を持つすべてのサービスには、RBAC ロール割り当て作成が必須です。**  
ID があってもロール割り当てがなければ、サービス間認証は失敗します。  
これは任意ではなく、**必須項目** です。  
省略は Phase 3 レビューで CRITICAL として報告されます。

- 必須 RBAC マッピング:

| Source Service | Target Service | Role | Role Definition ID |
|------------|-----------|------|-------------------|
| Foundry | Storage | `Storage Blob Data Contributor` | `ba92f5b4-2d11-453d-a403-e96b0029c9fe` |
| Foundry | AI Search | `Search Index Data Contributor` | `8ebe5a00-799e-43f5-93ac-243d3dce84a7` |
| Foundry | AI Search | `Search Service Contributor` | `7ca78c08-252a-4471-8644-bb5ff32d4ba0` |
| App Service | Key Vault | `Key Vault Secrets User` | `4633458b-17de-408a-b874-0445c86b69e6` |
| AKS (kubeletIdentity) | ACR | `AcrPull` | `7f951dda-4ed3-4680-a7ca-43fe172d538d` |
| Data Factory | Storage | `Storage Blob Data Contributor` | `ba92f5b4-2d11-453d-a403-e96b0029c9fe` |
| Data Factory | Key Vault | `Key Vault Secrets User` | `4633458b-17de-408a-b874-0445c86b69e6` |
| Databricks | Storage | `Storage Blob Data Contributor` | `ba92f5b4-2d11-453d-a403-e96b0029c9fe` |

> **AKS 特別ルール**: AKS は `identityProfile.kubeletidentity.objectId` を使用し、`identity.principalId` ではありません。

```bicep
// RBAC 例 — Foundry → Storage Blob Data Contributor
resource foundryStorageRole 'Microsoft.Authorization/roleAssignments@2022-04-01' = {
  name: guid(storageAccount.id, foundry.id, 'ba92f5b4-2d11-453d-a403-e96b0029c9fe')
  scope: storageAccount
  properties: {
    roleDefinitionId: subscriptionResourceId('Microsoft.Authorization/roleDefinitions', 'ba92f5b4-2d11-453d-a403-e96b0029c9fe')
    principalId: foundry.identity.principalId
    principalType: 'ServicePrincipal'
  }
}
```

### SQL Server ルール
- **パスワード管理**: main.bicep に `@secure() param sqlAdminPassword string` を宣言し、modules に渡す
  - modules 内で `newGuid()` 生成しない — 再デプロイ時にパスワードが変わるため
  - デプロイ後に取得できるよう Key Vault Secret として保存する
- **認証方式**: 既定は `administrators.azureADOnlyAuthentication: true`
  - 多くの組織ポリシー（MCAPS など）はスタンドアロン SQL 認証をブロック
  - AAD-only 認証 + Managed Identity が最も安全な構成

### ネットワーク シークレット取り扱い
- **VPN Gateway shared key**: `@secure() param vpnSharedKey string` — `@secure()` は必須
- `.bicepparam` に VPN キー平文を絶対に含めない — デプロイ時に渡すか Key Vault 参照を使用
- このルールは SQL パスワードと同様に適用
- **適用対象**: VPN shared key、ExpressRoute authorization key、Wi-Fi PSK、その他すべてのネットワーク シークレット
- Module params にも `@secure()` デコレーターが必要

### ⚠️ ネットワーク分離の整合性ルール
- `publicNetworkAccess: 'Disabled'` を設定する場合、そのサービスに対応する PE も **必ず** 作成すること
- PE なしで publicNetworkAccess を Disabled にするとサービスに到達不能 → デプロイ後に使用不可
- Phase 3 レビュアーはこの不整合を **CRITICAL** として報告すること
- 不整合が見つかった場合: PE モジュールを追加するか、publicNetworkAccess を Enabled に戻す

## 必須コーディング原則

### 命名規約
```bicep
// 命名衝突防止のため uniqueString を使用 — 常に必須
param foundryName string = 'foundry-${uniqueString(resourceGroup().id)}'
param searchName string = 'srch-${uniqueString(resourceGroup().id)}'
param storageName string = 'st${uniqueString(resourceGroup().id)}'  // 特殊文字は使用不可
param keyVaultName string = 'kv-${uniqueString(resourceGroup().id)}'
```
> **⚠️ `customSubDomainName` が必要なリソース（Foundry、Cognitive Services など）は `uniqueString()` を必ず含めること。**
> 固定文字列（例: `'my-rag-chatbot'`）は他テナントですでに使用されている可能性があり、デプロイ失敗の原因になります。
> 同じルールは Foundry Project 名にも適用 — `'project-${uniqueString(resourceGroup().id)}'`

### ネットワーク分離
```bicep
// Private Endpoints 使用時は全サービスで必須
publicNetworkAccess: 'Disabled'
networkAcls: {
  defaultAction: 'Deny'
  ipRules: []
  virtualNetworkRules: []
}
```

### 依存関係管理
```bicep
// 明示的 dependsOn ではなく、リソース参照による暗黙依存を使用
resource aiProject '...' = {
  properties: {
    hubResourceId: aiHub.id  // aiHub 参照により aiHub が自動的に先行デプロイされる
  }
}
```

### セキュリティ
```bicep
// 機密値は Key Vault 参照を使用 — パラメーターファイルに平文保存しない
@secure()
param adminPassword string  // main.bicepparam に平文値を入れない
```

### コードコメント
```bicep
// Microsoft Foundry リソース — kind: 'AIServices'
// customSubDomainName: 必須、グローバル一意。作成後は変更不可 — 省略時はリソース削除・再作成が必要
// allowProjectManagement: true は必須。ないと Foundry Project 作成に失敗
// apiVersion はステップ 0 で取得した最新バージョンに置き換える
resource foundry 'Microsoft.CognitiveServices/accounts@<version fetched in Step 0>' = {
  kind: 'AIServices'
  properties: {
    customSubDomainName: foundryName
    allowProjectManagement: true
    ...
  }
}
```

### ⚠️ Bicep コード品質検証（生成後に必須）

**モジュール宣言の検証:**
- 各 module ブロックの `name:` プロパティが重複していないことを確認
- 正しい例: `name: 'deploy-sql'`
- 誤った例: `name: 'name: 'deploy-sql'`（name: が重複 → コンパイルエラー）

**重複プロパティ防止:**
- 同一 resource ブロック内で同じプロパティ名が複数回出現するとコンパイルエラー
- 特に VPN Gateway（`gatewayType`）、Firewall、AKS など複雑なリソースで発生しやすい
- `az bicep build` 出力で `BCP025: The property "xxx" is declared multiple times` を確認

**`az bicep build` の実行は必須:**
- すべての Bicep ファイル生成後、必ず `az bicep build --file main.bicep` を実行
- エラーを修正して再コンパイル
- 警告（BCP081 など）は、MS Docs で API バージョン確認後なら無視可

## main.bicep 基本構成

```bicep
// ============================================================
// Azure [Project Name] Infrastructure — main.bicep
// Generated: [Date]
// ============================================================

targetScope = 'resourceGroup'

// ── 共通パラメーター ─────────────────────────────────────
param location string   // Phase 1 で確認した Location — ハードコードしない
param projectPrefix string
param vnetAddressPrefix string    // ← ユーザー確認必須。既存ネットワークとの競合を回避
param peSubnetPrefix string       // ← VNet 内の PE 専用サブネット CIDR

// ── Network ───────────────────────────────────────────────
module network './modules/network.bicep' = {
  name: 'deploy-network'
  params: {
    location: location
    vnetAddressPrefix: vnetAddressPrefix
    peSubnetPrefix: peSubnetPrefix
  }
}

// ── AI/Data Services ──────────────────────────────────────
module ai './modules/ai.bicep' = {
  name: 'deploy-ai'
  params: {
    location: location
    // サービスごとにリージョンが異なる場合は個別 params を追加 — MS Docs で利用可能リージョンを確認
  }
  dependsOn: [network]
}

// ── Storage ───────────────────────────────────────────────
module storage './modules/storage.bicep' = {
  name: 'deploy-storage'
  params: {
    location: location
  }
}

// ── Key Vault ─────────────────────────────────────────────
module keyVault './modules/keyvault.bicep' = {
  name: 'deploy-keyvault'
  params: {
    location: location
  }
}

// ── Private Endpoints（全サービス） ──────────────────────
module privateEndpoints './modules/private-endpoints.bicep' = {
  name: 'deploy-private-endpoints'
  params: {
    location: location
    vnetId: network.outputs.vnetId
    peSubnetId: network.outputs.peSubnetId
    foundryId: ai.outputs.foundryId
    searchId: ai.outputs.searchId
    storageId: storage.outputs.storageId
    keyVaultId: keyVault.outputs.keyVaultId
  }
}

// ── Outputs ───────────────────────────────────────────────
output vnetId string = network.outputs.vnetId
output foundryEndpoint string = ai.outputs.foundryEndpoint
output searchEndpoint string = ai.outputs.searchEndpoint
```

## main.bicepparam 基本構成

```bicep
using './main.bicep'

param location = '<Phase 1 で確認した Location>'
param projectPrefix = '<Project prefix>'
// 機密値はここに入れない — Key Vault 参照を使用
// サービスごとの可用性を MS Docs で確認後にリージョンを設定
```

### @secure() パラメーターの取り扱い

`.bicepparam` ファイルに `using` ディレクティブがある場合、`az deployment` で追加の `--parameters` フラグは使えません。  
そのため、`@secure()` パラメーターは以下ルールに従ってください:

- **可能なら既定値を設定する**: `@secure() param password string = newGuid()`
- **@secure() パラメーターでユーザー入力が必要な場合**: `.bicepparam` の代わりに JSON パラメーターファイル（`main.parameters.json`）を併せて生成
- **絶対に禁止**: `.bicepparam` と `--parameters key=value` を同時に使うコマンドを生成すること

## よくあるミスのチェックリスト

完全版チェックリストは `references/service-gotchas.md` にあります。主要ポイント:

| Item | ❌ Incorrect | ✅ Correct |
|------|--------|----------|
| ADLS Gen2 | `isHnsEnabled` omitted | `isHnsEnabled: true` |
| PE Subnet | Policy not set | `privateEndpointNetworkPolicies: 'Disabled'` |
| PE Configuration | PE only created | PE + DNS Zone + VNet Link + DNS Zone Group |
| Foundry | `kind: 'OpenAI'` | `kind: 'AIServices'` + `allowProjectManagement: true` |
| Foundry | `customSubDomainName` omitted | `customSubDomainName: foundryName` — cannot be changed after creation |
| Foundry Project | Not created | Must always be created as a set with the Foundry resource |
| Hub Usage | Used for standard AI | Only when explicitly requested by user or ML/open-source models needed |
| Public Network | Not configured | `publicNetworkAccess: 'Disabled'` |
| Storage Name | Contains hyphens | Lowercase + digits only, `uniqueString()` recommended |
| API version | Copied from previous value | Fetch from MS Docs (Dynamic) |
| Region | Hardcoded | Parameter + verify availability in MS Docs (Dynamic) |

## 生成完了後

Bicep 生成が完了したら:
1. 生成ファイル一覧と各ファイルの役割を要約レポートとしてユーザーに提示する
2. 直ちに Phase 3（Bicep Reviewer）へ移行する
3. レビュアーは `references/bicep-reviewer.md` ガイドラインに従って自動レビューと修正を実施する

