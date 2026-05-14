---
name: entra-agent-user
description: 'Agent Identity から Microsoft Entra ID に Agent User を作成し、AI エージェントが Microsoft 365 や Azure 環境でユーザー ID 機能を持つデジタル ワーカーとして動作できるようにします。'
---

# SKILL: Creating Agent Users in Microsoft Entra Agent ID

## Overview

**agent user** は、AI エージェントがデジタル ワーカーとして振る舞えるようにする Microsoft Entra ID の特殊な user identity です。適切なセキュリティ境界を維持しつつ、ユーザー ID を厳密に要求する API やサービス (例: Exchange mailbox、Teams、org chart) にエージェントがアクセスできるようになります。

agent user は、通常の agent identity が `idtyp=app` を受け取るのとは異なり、`idtyp=user` を持つ token を受け取ります。

---

## Prerequisites

- Agent ID 機能を持つ **Microsoft Entra tenant**
- **agent identity blueprint** から作成された **agent identity** (`ServiceIdentity` 型の service principal)
- 次のいずれかの **permission**:
  - `AgentIdUser.ReadWrite.IdentityParentedBy` (最小権限)
  - `AgentIdUser.ReadWrite.All`
  - `User.ReadWrite.All`
- 呼び出し元は少なくとも **Agent ID Administrator** ロールを持っている必要があります (delegated scenario)

> **Important:** `identityParentId` は、通常の application service principal ではなく、真の agent identity (agent identity blueprint で作成されたもの) を参照していなければなりません。service principal に `@odata.type: #microsoft.graph.agentIdentity` と `servicePrincipalType: ServiceIdentity` があることを確認して検証できます。

---

## Architecture

```
Agent Identity Blueprint (application template)
    │
    ├── Agent Identity (service principal - ServiceIdentity)
    │       │
    │       └── Agent User (user - agentUser) ← 1:1 relationship
    │
    └── Agent Identity Blueprint Principal (service principal in tenant)
```

| Component | Type | Token Claim | Purpose |
|---|---|---|---|
| Agent Identity | Service Principal | `idtyp=app` | Backend/API operations |
| Agent User | User (`agentUser`) | `idtyp=user` | M365 でデジタル ワーカーとして振る舞う |

---

## Step 1: Verify the Agent Identity Exists

agent user を作成する前に、その agent identity が正しい `agentIdentity` 型であることを確認します。

```http
GET https://graph.microsoft.com/beta/servicePrincipals/{agent-identity-id}
Authorization: Bearer <token>
```

応答に次が含まれていることを確認してください。
```json
{
  "@odata.type": "#microsoft.graph.agentIdentity",
  "servicePrincipalType": "ServiceIdentity",
  "agentIdentityBlueprintId": "<blueprint-id>"
}
```

### PowerShell

```powershell
Connect-MgGraph -Scopes "Application.Read.All" -TenantId "<tenant>" -UseDeviceCode -NoWelcome
Invoke-MgGraphRequest -Method GET `
  -Uri "https://graph.microsoft.com/beta/servicePrincipals/<agent-identity-id>" | ConvertTo-Json -Depth 3
```

> **Common mistake:** app registration の `appId` や通常の application service principal の `id` を使うと失敗します。blueprint から作成された agent identity だけが使えます。

---

## Step 2: Create the Agent User

### HTTP Request

```http
POST https://graph.microsoft.com/beta/users/microsoft.graph.agentUser
Content-Type: application/json
Authorization: Bearer <token>

{
  "accountEnabled": true,
  "displayName": "My Agent User",
  "mailNickname": "my-agent-user",
  "userPrincipalName": "my-agent-user@yourtenant.onmicrosoft.com",
  "identityParentId": "<agent-identity-object-id>"
}
```

### Required Properties

| Property | Type | Description |
|---|---|---|
| `accountEnabled` | Boolean | アカウントを有効にするには `true` |
| `displayName` | String | 人が読みやすい名前 |
| `mailNickname` | String | メール エイリアス (空白や特殊文字なし) |
| `userPrincipalName` | String | UPN — tenant 内で一意である必要がある (`alias@verified-domain`) |
| `identityParentId` | String | 親 agent identity の object ID |

### PowerShell

```powershell
Connect-MgGraph -Scopes "User.ReadWrite.All" -TenantId "<tenant>" -UseDeviceCode -NoWelcome

$body = @{
  accountEnabled    = $true
  displayName       = "My Agent User"
  mailNickname      = "my-agent-user"
  userPrincipalName = "my-agent-user@yourtenant.onmicrosoft.com"
  identityParentId  = "<agent-identity-object-id>"
} | ConvertTo-Json

Invoke-MgGraphRequest -Method POST `
  -Uri "https://graph.microsoft.com/beta/users/microsoft.graph.agentUser" `
  -Body $body -ContentType "application/json" | ConvertTo-Json -Depth 3
```

### Key Notes

- **No password** — agent user は password を持てません。認証には親 agent identity の credential を使います。
- **1:1 relationship** — 各 agent identity は最大 1 つの agent user しか持てません。2 つ目を作成しようとすると `400 Bad Request` になります。
- `userPrincipalName` は一意でなければなりません。既存ユーザーの UPN を再利用してはいけません。

---

## Step 3: Assign a Manager (Optional)

manager を割り当てると、agent user を org chart (例: Teams) に表示できます。

```http
PUT https://graph.microsoft.com/beta/users/{agent-user-id}/manager/$ref
Content-Type: application/json
Authorization: Bearer <token>

{
  "@odata.id": "https://graph.microsoft.com/beta/users/{manager-user-id}"
}
```

### PowerShell

```powershell
$managerBody = '{"@odata.id":"https://graph.microsoft.com/beta/users/<manager-user-id>"}'
Invoke-MgGraphRequest -Method PUT `
  -Uri "https://graph.microsoft.com/beta/users/<agent-user-id>/manager/`$ref" `
  -Body $managerBody -ContentType "application/json"
```

---

## Step 4: Set Usage Location and Assign Licenses (Optional)

mailbox や Teams presence などを持たせるには license が必要です。先に usage location を設定する必要があります。

### Set Usage Location

```http
PATCH https://graph.microsoft.com/beta/users/{agent-user-id}
Content-Type: application/json
Authorization: Bearer <token>

{
  "usageLocation": "US"
}
```

### List Available Licenses

```http
GET https://graph.microsoft.com/beta/subscribedSkus?$select=skuPartNumber,skuId,consumedUnits,prepaidUnits
Authorization: Bearer <token>
```

`Organization.Read.All` permission が必要です。

### Assign a License

```http
POST https://graph.microsoft.com/beta/users/{agent-user-id}/assignLicense
Content-Type: application/json
Authorization: Bearer <token>

{
  "addLicenses": [
    { "skuId": "<sku-id>" }
  ],
  "removeLicenses": []
}
```

### PowerShell (all in one)

```powershell
Connect-MgGraph -Scopes "User.ReadWrite.All","Organization.Read.All" -TenantId "<tenant>" -NoWelcome

# Set usage location
Invoke-MgGraphRequest -Method PATCH `
  -Uri "https://graph.microsoft.com/beta/users/<agent-user-id>" `
  -Body '{"usageLocation":"US"}' -ContentType "application/json"

# Assign license
$licenseBody = '{"addLicenses":[{"skuId":"<sku-id>"}],"removeLicenses":[]}'
Invoke-MgGraphRequest -Method POST `
  -Uri "https://graph.microsoft.com/beta/users/<agent-user-id>/assignLicense" `
  -Body $licenseBody -ContentType "application/json"
```

> **Tip:** **Entra admin center** の Identity → Users → All users → agent user を選択 → Licenses and apps から license を割り当てることもできます。

---

## Provisioning Times

| Service | Estimated Time |
|---|---|
| Exchange mailbox | 5–30 分 |
| Teams availability | 15 分 – 24 時間 |
| Org chart / People search | 最大 24–48 時間 |
| SharePoint / OneDrive | 5–30 分 |
| Global Address List | 最大 24 時間 |

---

## Agent User Capabilities

- ✅ Microsoft Entra group に追加できる (dynamic group を含む)
- ✅ user 専用 API にアクセスできる (`idtyp=user` token)
- ✅ mailbox、calendar、contacts を所有できる
- ✅ Teams chat や channel に参加できる
- ✅ org chart や People search に表示される
- ✅ administrative unit に追加できる
- ✅ license を割り当てられる

## Agent User Security Constraints

- ❌ password、passkey、interactive sign-in を持てない
- ❌ privileged admin role を割り当てられない
- ❌ role-assignable group に追加できない
- ❌ 既定では guest user に近い permission である
- ❌ custom role assignment は利用できない

---

## Troubleshooting

| Error | Cause | Fix |
|---|---|---|
| `Agent user IdentityParent does not exist` | `identityParentId` が存在しない、または agent identity ではない object を指している | ID が通常 app ではなく `agentIdentity` service principal であることを確認する |
| `400 Bad Request` (identityParentId already linked) | その agent identity にはすでに agent user がある | 各 agent identity がサポートする agent user は 1 つだけ |
| `409 Conflict` on UPN | `userPrincipalName` がすでに使われている | 一意の UPN を使う |
| License assignment fails | usage location が設定されていない | license 割り当て前に `usageLocation` を設定する |

---

## References

- [Agent identities](https://learn.microsoft.com/en-us/entra/agent-id/identity-platform/agent-identities)
- [Agent users](https://learn.microsoft.com/en-us/entra/agent-id/identity-platform/agent-users)
- [Agent service principals](https://learn.microsoft.com/en-us/entra/agent-id/identity-platform/agent-service-principals)
- [Create agent identity blueprint](https://learn.microsoft.com/en-us/entra/agent-id/identity-platform/create-blueprint)
- [Create agent identities](https://learn.microsoft.com/en-us/entra/agent-id/identity-platform/create-delete-agent-identities)
- [agentUser resource type (Graph API)](https://learn.microsoft.com/en-us/graph/api/resources/agentuser?view=graph-rest-beta)
- [Create agentUser (Graph API)](https://learn.microsoft.com/en-us/graph/api/agentuser-post?view=graph-rest-beta)
