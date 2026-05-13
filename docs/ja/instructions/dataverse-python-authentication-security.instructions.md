---
applyTo: '**'
---

# Dataverse SDK for Python — 認証とセキュリティ パターン

Microsoft 公式の Azure SDK 認証ドキュメントと Dataverse SDK ベストプラクティスに基づいています。

## 1. 認証概要

Dataverse SDK for Python は、トークン ベース認証のために Azure Identity credential を使用します。この方法は最小権限の原則に従い、ローカル開発、クラウド デプロイ、オンプレミス環境で動作します。

### なぜトークン ベース認証なのか?

**接続文字列に対する利点**:
- アプリに必要な特定権限だけを確立できる (最小権限の原則)
- credential は意図したアプリだけにスコープされる
- managed identity を使えば、保存や漏えいの対象となる secret がない
- コード変更なしで環境間をシームレスに移動できる

---

## 2. Credential 種別と選定

### Interactive Browser Credential (ローカル開発)

**Use for**: ローカル開発中の開発者ワークステーション。

```python
from azure.identity import InteractiveBrowserCredential
from PowerPlatform.Dataverse.client import DataverseClient

# 認証用にブラウザーを開く
credential = InteractiveBrowserCredential()
client = DataverseClient(
    base_url="https://myorg.crm.dynamics.com",
    credential=credential
)

# 初回はサインインを求められ、その後はキャッシュ済みトークンを使用
records = client.get("account")
```

**利用場面**:
- ✅ 対話的な開発とテスト
- ✅ UI を持つデスクトップ アプリケーション
- ❌ バックグラウンド サービスや定期ジョブ

---

### Default Azure Credential (すべての環境で推奨)

**Use for**: 複数環境 (dev → test → production) で動くアプリ。

```python
from azure.identity import DefaultAzureCredential
from PowerPlatform.Dataverse.client import DataverseClient

# 次の順序で credential を試行:
# 1. 環境変数 (app service principal)
# 2. Azure CLI credential (ローカル開発)
# 3. Azure PowerShell credential (ローカル開発)
# 4. Managed identity (Azure 上で実行時)
credential = DefaultAzureCredential()

client = DataverseClient(
    base_url="https://myorg.crm.dynamics.com",
    credential=credential
)

records = client.get("account")
```

**利点**:
- 単一のコード パスでどこでも動作する
- 環境ごとの分岐ロジックが不要
- 利用可能な credential を自動検出する
- 本番アプリで推奨される

**Credential chain**:
1. 環境変数 (`AZURE_CLIENT_ID`, `AZURE_TENANT_ID`, `AZURE_CLIENT_SECRET`)
2. Visual Studio Code login
3. Azure CLI (`az login`)
4. Azure PowerShell (`Connect-AzAccount`)
5. Managed identity (Azure VM、App Service、AKS など)

---

### Client Secret Credential (Service Principal)

**Use for**: 無人認証 (定期ジョブ、script、オンプレミス サービス)。

```python
from azure.identity import ClientSecretCredential
from PowerPlatform.Dataverse.client import DataverseClient
import os

credential = ClientSecretCredential(
    tenant_id=os.environ["AZURE_TENANT_ID"],
    client_id=os.environ["AZURE_CLIENT_ID"],
    client_secret=os.environ["AZURE_CLIENT_SECRET"]
)

client = DataverseClient(
    base_url="https://myorg.crm.dynamics.com",
    credential=credential
)

records = client.get("account")
```

**セットアップ手順**:
1. Azure AD で app registration を作成する
2. client secret を作成する (安全に保持すること)
3. app に Dataverse permission を付与する
4. credential を環境変数または secure vault に保存する

**セキュリティ上の懸念**:
- ⚠️ credential を source code にハードコードしない
- ⚠️ secret は Azure Key Vault または環境変数に保存する
- ⚠️ credential は定期的にローテーションする
- ⚠️ 必要最小限の permission を使う

---

### Managed Identity Credential (Azure リソース)

**Use for**: Azure 上にホストされるアプリ (App Service、Azure Functions、AKS、VM)。

```python
from azure.identity import ManagedIdentityCredential
from PowerPlatform.Dataverse.client import DataverseClient

# secret 不要 - Azure が identity を管理
credential = ManagedIdentityCredential()

client = DataverseClient(
    base_url="https://myorg.crm.dynamics.com",
    credential=credential
)

records = client.get("account")
```

**利点**:
- ✅ 管理する secret がない
- ✅ token refresh が自動
- ✅ 高いセキュリティ
- ✅ Azure service に組み込み

**セットアップ**:
1. Azure リソース (App Service、VM など) で managed identity を有効化する
2. managed identity に Dataverse permission を付与する
3. コードは自動的にその identity を使用する

---

## 3. 環境別設定

### ローカル開発

```python
# .env file (git-ignored)
DATAVERSE_URL=https://myorg-dev.crm.dynamics.com

# Python code
import os
from azure.identity import DefaultAzureCredential
from PowerPlatform.Dataverse.client import DataverseClient

# Azure CLI credential を使用
credential = DefaultAzureCredential()
client = DataverseClient(
    base_url=os.environ["DATAVERSE_URL"],
    credential=credential
)
```

**Setup**: 開発者アカウントで `az login` を実行

---

### Azure App Service / Azure Functions

```python
from azure.identity import ManagedIdentityCredential
from PowerPlatform.Dataverse.client import DataverseClient

# managed identity を自動使用
credential = ManagedIdentityCredential()
client = DataverseClient(
    base_url="https://myorg.crm.dynamics.com",
    credential=credential
)
```

**Setup**: App Service で managed identity を有効化し、Dataverse で permission を付与

---

### On-Premises / Third-Party Hosting

```python
import os
from azure.identity import ClientSecretCredential
from PowerPlatform.Dataverse.client import DataverseClient

credential = ClientSecretCredential(
    tenant_id=os.environ["AZURE_TENANT_ID"],
    client_id=os.environ["AZURE_CLIENT_ID"],
    client_secret=os.environ["AZURE_CLIENT_SECRET"]
)

client = DataverseClient(
    base_url="https://myorg.crm.dynamics.com",
    credential=credential
)
```

**Setup**: service principal を作成し、credential を安全に保存し、Dataverse permission を付与

---

## 4. Client 設定と接続設定

### 基本設定

```python
from PowerPlatform.Dataverse.core.config import DataverseConfig
from azure.identity import DefaultAzureCredential
from PowerPlatform.Dataverse.client import DataverseClient

cfg = DataverseConfig()
cfg.logging_enable = True  # 詳細 logging を有効化

client = DataverseClient(
    base_url="https://myorg.crm.dynamics.com",
    credential=DefaultAzureCredential(),
    config=cfg
)
```

### HTTP チューニング

```python
from PowerPlatform.Dataverse.core.config import DataverseConfig

cfg = DataverseConfig()

# timeout 設定
cfg.http_timeout = 30          # request timeout (秒)

# retry 設定
cfg.http_retries = 3           # retry 回数
cfg.http_backoff = 1           # 初期 backoff (秒)

# connection reuse
cfg.connection_timeout = 5     # connection timeout

client = DataverseClient(
    base_url="https://myorg.crm.dynamics.com",
    credential=credential,
    config=cfg
)
```

---

## 5. セキュリティ ベストプラクティス

### 1. Credential をハードコードしない

```python
# ❌ BAD - これはしないこと!
credential = ClientSecretCredential(
    tenant_id="your-tenant-id",
    client_id="your-client-id",
    client_secret="your-secret-key"  # EXPOSED!
)

# ✅ GOOD - 環境変数を使う
import os
credential = ClientSecretCredential(
    tenant_id=os.environ["AZURE_TENANT_ID"],
    client_id=os.environ["AZURE_CLIENT_ID"],
    client_secret=os.environ["AZURE_CLIENT_SECRET"]
)
```

### 2. Secret を安全に保管する

**Development**:
```bash
# .env file (git-ignored)
AZURE_TENANT_ID=your-tenant-id
AZURE_CLIENT_ID=your-client-id
AZURE_CLIENT_SECRET=your-secret-key
```

**Production**:
```python
from azure.keyvault.secrets import SecretClient
from azure.identity import DefaultAzureCredential

# Azure Key Vault から secret を取得
credential = DefaultAzureCredential()
client = SecretClient(
    vault_url="https://mykeyvault.vault.azure.net",
    credential=credential
)

secret = client.get_secret("dataverse-client-secret")
```

### 3. 最小権限の原則を実装する

```python
# 最小限の permission を付与:
# - 読み取りだけなら read のみ
# - 可能なら特定 table のみ
# - 時間制限付き credential (自動ローテーション)
# - 共有 secret より managed identity を使う
```

### 4. 認証イベントを監視する

```python
import logging

logger = logging.getLogger("dataverse_auth")

try:
    client = DataverseClient(
        base_url="https://myorg.crm.dynamics.com",
        credential=credential
    )
    logger.info("Successfully authenticated to Dataverse")
except Exception as e:
    logger.error(f"Authentication failed: {e}")
    raise
```

### 5. Token 期限切れを処理する

```python
from azure.core.exceptions import ClientAuthenticationError
import time

def create_with_auth_retry(client, table_name, payload, max_retries=2):
    """Token 期限切れ時に retry しながらレコードを作成する。"""
    for attempt in range(max_retries):
        try:
            return client.create(table_name, payload)
        except ClientAuthenticationError:
            if attempt < max_retries - 1:
                logger.warning("Token expired, retrying...")
                time.sleep(1)
            else:
                raise
```

---

## 6. マルチテナント アプリケーション

### Tenant-Aware Client

```python
from azure.identity import DefaultAzureCredential
from PowerPlatform.Dataverse.client import DataverseClient

def get_client_for_tenant(tenant_id: str) -> DataverseClient:
    """特定 tenant 向けの DataverseClient を返す。"""
    credential = DefaultAzureCredential()

    # Dataverse URL は tenant 固有の org を含む
    base_url = f"https://{get_org_for_tenant(tenant_id)}.crm.dynamics.com"

    return DataverseClient(
        base_url=base_url,
        credential=credential
    )

def get_org_for_tenant(tenant_id: str) -> str:
    """Tenant を Dataverse organization に対応付ける。"""
    # 実装はマルチテナント戦略によって異なる
    # database lookup、設定ファイルなどが考えられる
    pass
```

---

## 7. 認証のトラブルシューティング

### Error: "Access Denied" (403)

```python
try:
    client.get("account")
except DataverseError as e:
    if e.status_code == 403:
        print("User/app lacks Dataverse permissions")
        print("Ensure Dataverse security role is assigned")
```

### Error: "Invalid Credentials" (401)

```python
# credential source を確認
from azure.identity import DefaultAzureCredential

try:
    cred = DefaultAzureCredential(exclude_cli_credential=False,
                                  exclude_powershell_credential=False)
    # 再認証を強制
    import subprocess
    subprocess.run(["az", "login"])
except Exception as e:
    print(f"Authentication failed: {e}")
```

### Error: "Invalid Tenant"

```python
# tenant ID を確認
import json
from azure.identity import DefaultAzureCredential

credential = DefaultAzureCredential()
token = credential.get_token("https://dataverse.dynamics.com/.default")

# token を decode して tenant を確認
import base64
payload = base64.b64decode(token.token.split('.')[1] + '==')
claims = json.loads(payload)
print(f"Token tenant: {claims.get('tid')}")
```

---

## 8. Credential ライフサイクル

### Token Refresh

Azure Identity は token refresh を自動で処理します。

```python
# token は自動的に cache され、refresh される
credential = DefaultAzureCredential()

# 初回呼び出しで token を取得
client.get("account")

# 以後の呼び出しは cache 済み token を再利用
client.get("contact")

# token が期限切れになれば SDK が自動 refresh
```

### Session 管理

```python
class DataverseSession:
    """DataverseClient のライフサイクルを管理する。"""

    def __init__(self, base_url: str):
        from azure.identity import DefaultAzureCredential

        self.client = DataverseClient(
            base_url=base_url,
            credential=DefaultAzureCredential()
        )

    def __enter__(self):
        return self.client

    def __exit__(self, exc_type, exc_val, exc_tb):
        # 必要なら cleanup
        pass

# Usage
with DataverseSession("https://myorg.crm.dynamics.com") as client:
    records = client.get("account")
```

---

## 9. Dataverse 固有のセキュリティ

### Row-Level Security (RLS)

ユーザーの Dataverse security role がアクセス可能なレコードを決定します。

```python
from azure.identity import InteractiveBrowserCredential
from PowerPlatform.Dataverse.client import DataverseClient

# 各 user は自分の credential を使う client を取得
def get_user_client(user_username: str) -> DataverseClient:
    # User は既に認証済みである必要がある
    credential = InteractiveBrowserCredential()

    client = DataverseClient(
        base_url="https://myorg.crm.dynamics.com",
        credential=credential
    )

    # User は自分にアクセス権のあるレコードだけを見られる
    return client
```

### Security Role

必要最小限の role を割り当てます:
- **System Administrator**: 全権限 (アプリ用途では避ける)
- **Sales Manager**: 営業 table + reporting
- **Service Representative**: service case + knowledge
- **Custom**: 特定 table permission を持つ role を作成

---

## 10. 関連資料

- [Azure Identity Client Library](https://learn.microsoft.com/en-us/python/api/azure-identity)
- [Authenticate to Azure Services](https://learn.microsoft.com/en-us/azure/developer/python/sdk/authentication/overview)
- [Azure Key Vault for Secrets](https://learn.microsoft.com/en-us/azure/key-vault/general/overview)
- [Dataverse Security Model](https://learn.microsoft.com/en-us/power-platform/admin/security/security-overview)
