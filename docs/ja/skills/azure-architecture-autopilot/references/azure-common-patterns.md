# Azure 共通パターン（Stable）

このファイルには、Azure サービス全体で繰り返し使われる**ほぼ不変のパターン**のみを記載しています。  
API バージョン、SKU、リージョンなどの動的情報はここには含めません → `azure-dynamic-sources.md` を参照してください。

---

## 1. ネットワーク分離パターン

### Private Endpoint 3 コンポーネント構成

PE を使用するすべてのサービスでは、次の 3 コンポーネント構成が必要です。

1. **Private Endpoint** — pe-subnet に配置
2. **Private DNS Zone** + **VNet Link** (`registrationEnabled: false`)
3. **DNS Zone Group** — PE に関連付け

> いずれか 1 つでも欠けると、PE が存在していても DNS 名前解決に失敗し、接続エラーになります。

### PE サブネットの必須設定

```bicep
resource peSubnet 'Microsoft.Network/virtualNetworks/subnets' = {
  properties: {
    addressPrefix: peSubnetPrefix              // ← CIDR はパラメーターとして指定 — 既存ネットワークとの競合を防止
    privateEndpointNetworkPolicies: 'Disabled'  // ← 必須。未設定だと PE のデプロイが失敗
  }
}
```

### publicNetworkAccess パターン

PE を使用するサービスには、以下を必ず含めます。
```bicep
properties: {
  publicNetworkAccess: 'Disabled'
  networkAcls: {
    defaultAction: 'Deny'
  }
}
```

---

## 2. セキュリティパターン

### Key Vault

```bicep
properties: {
  enableRbacAuthorization: true    // Access Policy 方式は使用しない
  enableSoftDelete: true
  softDeleteRetentionInDays: 90
  enablePurgeProtection: true
}
```

### Managed Identity

AI サービスが他リソースにアクセスする場合:
```bicep
identity: {
  type: 'SystemAssigned'  // または 'UserAssigned'
}
```

### 機密情報

- `@secure()` デコレーターを使用する
- `.bicepparam` ファイルに平文を保存しない
- Key Vault 参照を使用する

---

## 3. 命名規則（CAF ベース）

```
rg-{project}-{env}          Resource Group
vnet-{project}-{env}        Virtual Network
st{project}{env}             Storage Account (特殊文字不可、英小文字+数字のみ)
kv-{project}-{env}           Key Vault
srch-{project}-{env}         AI Search
foundry-{project}-{env}      Cognitive Services (Foundry)
```

> 名前衝突防止: `uniqueString(resourceGroup().id)` の利用を推奨  
> ```bicep
> param storageName string = 'st${uniqueString(resourceGroup().id)}'
> ```

---

## 4. Bicep モジュール構成

```
<project>/
├── main.bicep              # オーケストレーション — モジュール呼び出し + パラメーター受け渡し
├── main.bicepparam         # 環境固有の値（機密情報を除く）
└── modules/
    ├── network.bicep           # VNet, Subnet
    ├── <service>.bicep         # サービスごとのモジュール
    ├── keyvault.bicep          # Key Vault
    └── private-endpoints.bicep # すべての PE + DNS Zone + VNet Link
```

### 依存関係管理

```bicep
// ✅ 正しい例: リソース参照による暗黙的依存関係
resource project '...' = {
  properties: {
    parentId: foundry.id  // foundry を参照 → foundry が自動的に先にデプロイされる
  }
}

// ❌ 非推奨: 明示的な dependsOn（必要な場合のみ使用）
```

---

## 5. PE Bicep 共通テンプレート

```bicep
// ── Private Endpoint ──
resource pe 'Microsoft.Network/privateEndpoints@<fetch>' = {
  name: 'pe-${serviceName}'
  location: location
  properties: {
    subnet: { id: peSubnetId }
    privateLinkServiceConnections: [{
      name: 'pls-${serviceName}'
      properties: {
        privateLinkServiceId: serviceId
        groupIds: ['<groupId>']  // ← サービスごとに異なる。service-gotchas.md を参照
      }
    }]
  }
}

// ── Private DNS Zone ──
resource dnsZone 'Microsoft.Network/privateDnsZones@<fetch>' = {
  name: '<dnsZoneName>'  // ← サービスごとに異なる
  location: 'global'
}

// ── VNet Link ──
resource vnetLink 'Microsoft.Network/privateDnsZones/virtualNetworkLinks@<fetch>' = {
  parent: dnsZone
  name: '${dnsZone.name}-link'
  location: 'global'
  properties: {
    virtualNetwork: { id: vnetId }
    registrationEnabled: false  // ← false である必要がある
  }
}

// ── DNS Zone Group ──
resource dnsGroup 'Microsoft.Network/privateEndpoints/privateDnsZoneGroups@<fetch>' = {
  parent: pe
  name: 'default'
  properties: {
    privateDnsZoneConfigs: [{
      name: 'config'
      properties: { privateDnsZoneId: dnsZone.id }
    }]
  }
}
```

> `@<fetch>`: デプロイ前に、MS Docs で最新の安定 API バージョンを必ず確認してください。

