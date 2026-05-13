# Dataverse SDK for Python - ベストプラクティス ガイド

## 概要
Microsoft 公式の PowerPlatform-DataverseClient-Python リポジトリ、サンプル、推奨ワークフローから抽出した、本番対応のパターンとベストプラクティスです。

## 1. インストールと環境セットアップ

### 本番インストール
```bash
# PyPI から公開済み SDK をインストール
pip install PowerPlatform-Dataverse-Client

# 認証用に Azure Identity をインストール
pip install azure-identity

# 任意: データ操作用の pandas 連携
pip install pandas
```

### 開発用インストール
```bash
# リポジトリを clone
git clone https://github.com/microsoft/PowerPlatform-DataverseClient-Python.git
cd PowerPlatform-DataverseClient-Python

# ライブ開発のため editable mode でインストール
pip install -e .

# 開発依存をインストール
pip install pytest pytest-cov black isort mypy ruff
```

### Python バージョン サポート
- **Minimum**: Python 3.10
- **Recommended**: 最良の性能のため Python 3.11+
- **Supported**: Python 3.10, 3.11, 3.12, 3.13, 3.14

### インストール確認
```python
from PowerPlatform.Dataverse import __version__
from PowerPlatform.Dataverse.client import DataverseClient
from azure.identity import InteractiveBrowserCredential

print(f"SDK Version: {__version__}")
print("Installation successful!")
```

---

## 2. 認証パターン

### 対話型開発 (ブラウザー ベース)
```python
from azure.identity import InteractiveBrowserCredential
from PowerPlatform.Dataverse.client import DataverseClient

credential = InteractiveBrowserCredential()
client = DataverseClient("https://yourorg.crm.dynamics.com", credential)
```

**利用場面:** ローカル開発、対話的テスト、単一ユーザー シナリオ。

### 本番 (Client Secret)
```python
from azure.identity import ClientSecretCredential
from PowerPlatform.Dataverse.client import DataverseClient

credential = ClientSecretCredential(
    tenant_id="your-tenant-id",
    client_id="your-client-id",
    client_secret="your-client-secret"
)
client = DataverseClient("https://yourorg.crm.dynamics.com", credential)
```

**利用場面:** サーバーサイド アプリケーション、Azure 自動化、定期ジョブ。

### 証明書ベース認証
```python
from azure.identity import ClientCertificateCredential
from PowerPlatform.Dataverse.client import DataverseClient

credential = ClientCertificateCredential(
    tenant_id="your-tenant-id",
    client_id="your-client-id",
    certificate_path="path/to/certificate.pem"
)
client = DataverseClient("https://yourorg.crm.dynamics.com", credential)
```

**利用場面:** 高度に安全な環境、certificate-pinning 要件。

### Azure CLI 認証
```python
from azure.identity import AzureCliCredential
from PowerPlatform.Dataverse.client import DataverseClient

credential = AzureCliCredential()
client = DataverseClient("https://yourorg.crm.dynamics.com", credential)
```

**利用場面:** Azure CLI をインストール済みのローカル テスト、Azure DevOps pipeline。

---

## 3. Singleton Client パターン

**ベストプラクティス**: `DataverseClient` instance は 1 つ作成し、アプリケーション全体で再利用します。

```python
# ❌ ANTI-PATTERN: client を毎回新規作成
def fetch_account(account_id):
    credential = InteractiveBrowserCredential()
    client = DataverseClient("https://yourorg.crm.dynamics.com", credential)
    return client.get("account", account_id)

# ✅ PATTERN: singleton client
class DataverseService:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            credential = InteractiveBrowserCredential()
            cls._instance = DataverseClient(
                "https://yourorg.crm.dynamics.com",
                credential
            )
        return cls._instance

# Usage
service = DataverseService()
account = service.get("account", account_id)
```

---

## 4. 設定の最適化

### 接続設定
```python
from PowerPlatform.Dataverse.core.config import DataverseConfig
from PowerPlatform.Dataverse.client import DataverseClient
from azure.identity import ClientSecretCredential

config = DataverseConfig(
    language_code=1033,  # 英語 (US)
    # Note: http_retries, http_backoff, http_timeout are reserved for internal use
)

credential = ClientSecretCredential(tenant_id, client_id, client_secret)
client = DataverseClient("https://yourorg.crm.dynamics.com", credential, config)
```

**主な設定項目:**
- `language_code`: API response の言語 (既定: 英語の 1033)

---

## 5. CRUD 操作のベストプラクティス

### Create 操作

#### 単一レコード
```python
record_data = {
    "name": "Contoso Ltd",
    "telephone1": "555-0100",
    "creditlimit": 100000.00,
}
created_ids = client.create("account", record_data)
record_id = created_ids[0]
print(f"Created: {record_id}")
```

#### 一括作成 (自動最適化)
```python
# SDK は 2 件以上の配列に対して自動的に CreateMultiple を使用
records = [
    {"name": f"Company {i}", "creditlimit": 50000 + (i * 1000)}
    for i in range(100)
]
created_ids = client.create("account", records)
print(f"Created {len(created_ids)} records")
```

**Performance**: 一括作成は内部的に最適化されるため、手動 batching は不要です。

### Read 操作

#### ID による単一レコード取得
```python
account = client.get("account", "account-guid-here")
print(account.get("name"))
```

#### filter と select を使ったクエリ
```python
# ページ分割された結果を返す (generator)
for page in client.get(
    "account",
    filter="creditlimit gt 50000",
    select=["name", "creditlimit", "telephone1"],
    orderby="name",
    top=100
):
    for account in page:
        print(f"{account['name']}: ${account['creditlimit']}")
```

**主なパラメーター:**
- `filter`: OData filter (**lowercase** の logical name を使用すること)
- `select`: 取得する field (性能向上)
- `orderby`: 結果の並び順
- `top`: 1 ページあたりの最大レコード数 (既定: 5000)
- `page_size`: pagination 用のページ サイズ上書き

#### SQL クエリ (読み取り専用)
```python
# SQL query は読み取り専用。複雑な分析に使う
results = client.query_sql("""
    SELECT TOP 10 name, creditlimit
    FROM account
    WHERE creditlimit > 50000
    ORDER BY name
""")

for row in results:
    print(f"{row['name']}: ${row['creditlimit']}")
```

**制限事項:**
- 読み取り専用 (SELECT のみ、DML 不可)
- 複雑な join や分析に有用
- 組織ポリシーで無効化されている場合がある

### Update 操作

#### 単一レコード
```python
client.update("account", "account-guid", {
    "creditlimit": 150000.00,
    "name": "Updated Company Name"
})
```

#### 一括更新 (同じ変更を一括適用)
```python
# 同じデータで選択レコードをすべて更新
account_ids = ["id1", "id2", "id3"]
client.update("account", account_ids, {
    "industrycode": 1,  # Retail
    "accountmanagerid": "manager-guid"
})
```

#### ペア更新 (1:1 のレコード更新)
```python
# レコードごとに異なる更新が必要な場合は複数回呼び出す
updates = {
    "id1": {"creditlimit": 100000},
    "id2": {"creditlimit": 200000},
    "id3": {"creditlimit": 300000},
}
for record_id, data in updates.items():
    client.update("account", record_id, data)
```

### Delete 操作

#### 単一レコード
```python
client.delete("account", "account-guid")
```

#### 一括削除 (最適化)
```python
# SDK は大きな ID リストに対して自動的に BulkDelete を使用
record_ids = ["id1", "id2", "id3", ...]
client.delete("account", record_ids, use_bulk_delete=True)
```

---

## 6. エラー処理と復旧

### 例外階層
```python
from PowerPlatform.Dataverse.core.errors import (
    DataverseError,           # 基底クラス
    ValidationError,          # validation failure
    MetadataError,           # table/column 操作
    HttpError,               # HTTP レベルの error
    SQLParseError            # SQL query 構文エラー
)

try:
    client.create("account", {"name": None})  # Invalid
except ValidationError as e:
    print(f"Validation failed: {e}")
    # validation 固有ロジックを処理
except DataverseError as e:
    print(f"General SDK error: {e}")
    # その他の SDK error を処理
```

### Retry ロジック パターン
```python
import time
from PowerPlatform.Dataverse.core.errors import HttpError

def create_with_retry(table_name, record_data, max_retries=3):
    """指数バックオフ付き retry ロジックでレコードを作成する。"""
    for attempt in range(max_retries):
        try:
            return client.create(table_name, record_data)
        except HttpError as e:
            if attempt == max_retries - 1:
                raise

            # 指数バックオフ: 1s, 2s, 4s
            backoff_seconds = 2 ** attempt
            print(f"Attempt {attempt + 1} failed. Retrying in {backoff_seconds}s...")
            time.sleep(backoff_seconds)

# Usage
created_ids = create_with_retry("account", {"name": "Contoso"})
```

### 429 (Request Rate Limit) の処理
```python
import time
from PowerPlatform.Dataverse.core.errors import HttpError

try:
    accounts = client.get("account", top=5000)
except HttpError as e:
    if "429" in str(e):
        # rate limit に達したので待って再試行
        print("Rate limited. Waiting 60 seconds...")
        time.sleep(60)
        accounts = client.get("account", top=5000)
    else:
        raise
```

---

## 7. Table と Column の管理

### Custom Table の作成
```python
from enum import IntEnum

class Priority(IntEnum):
    LOW = 1
    MEDIUM = 2
    HIGH = 3

# 型付きで column を定義
columns = {
    "new_Title": "string",
    "new_Quantity": "int",
    "new_Amount": "decimal",
    "new_Completed": "bool",
    "new_Priority": Priority,  # option set / picklist を作成
    "new_CreatedDate": "datetime"
}

table_info = client.create_table(
    "new_CustomTable",
    primary_column_schema_name="new_Name",
    columns=columns
)

print(f"Created table: {table_info['table_schema_name']}")
```

### Table Metadata の取得
```python
table_info = client.get_table_info("account")
print(f"Schema Name: {table_info['table_schema_name']}")
print(f"Logical Name: {table_info['table_logical_name']}")
print(f"Entity Set: {table_info['entity_set_name']}")
print(f"Primary ID: {table_info['primary_id_attribute']}")
```

### すべての Table を一覧取得
```python
tables = client.list_tables()
for table in tables:
    print(f"{table['table_schema_name']} ({table['table_logical_name']})")
```

### Column 管理
```python
# 既存 table に column を追加
client.create_columns("new_CustomTable", {
    "new_Status": "string",
    "new_Priority": "int"
})

# Column を削除
client.delete_columns("new_CustomTable", ["new_Status", "new_Priority"])

# Table を削除
client.delete_table("new_CustomTable")
```

---

## 8. Paging と大量結果セット

### Pagination パターン
```python
# すべての account をページ単位で取得
all_accounts = []
for page in client.get(
    "account",
    top=500,      # 1 ページあたりレコード数
    page_size=500
):
    all_accounts.extend(page)
    print(f"Retrieved page with {len(page)} records")

print(f"Total: {len(all_accounts)} records")
```

### continuation token を使った手動 paging
```python
# 複雑な paging シナリオ向け
skip_count = 0
page_size = 1000

while True:
    page = client.get("account", top=page_size, skip=skip_count)
    if not page:
        break

    print(f"Page {skip_count // page_size + 1}: {len(page)} records")
    skip_count += page_size
```

---

## 9. File 操作

### 小さいファイルの upload (< 128 MB)
```python
from pathlib import Path

file_path = Path("document.pdf")
record_id = "account-guid"

# 単一 PATCH upload
response = client.upload_file(
    table_name="account",
    record_id=record_id,
    file_column_name="new_documentfile",
    file_path=file_path
)
print(f"Upload successful: {response}")
```

### 大きいファイルの chunk upload
```python
from pathlib import Path

file_path = Path("large_video.mp4")
record_id = "account-guid"

# SDK が大きなファイルを自動で chunk 化
response = client.upload_file(
    table_name="account",
    record_id=record_id,
    file_column_name="new_videofile",
    file_path=file_path,
    chunk_size=4 * 1024 * 1024  # 4 MB chunk
)
print(f"Chunked upload complete")
```

---

## 10. OData Filter 最適化

### 大文字小文字ルール
```python
# ❌ WRONG: 大文字の logical name
results = client.get("account", filter="Name eq 'Contoso'")

# ✅ CORRECT: lowercase の logical name
results = client.get("account", filter="name eq 'Contoso'")

# ✅ 値は必要に応じて大文字小文字を区別する
results = client.get("account", filter="name eq 'Contoso Ltd'")
```

### Filter 式の例
```python
# Equality
client.get("account", filter="name eq 'Contoso'")

# Greater than / Less than
client.get("account", filter="creditlimit gt 50000")
client.get("account", filter="createdon lt 2024-01-01")

# String contains
client.get("account", filter="contains(name, 'Ltd')")

# AND/OR operations
client.get("account", filter="(name eq 'Contoso') and (creditlimit gt 50000)")
client.get("account", filter="(industrycode eq 1) or (industrycode eq 2)")

# NOT operation
client.get("account", filter="not(statecode eq 1)")
```

### Select と Expand
```python
# 特定 column のみ選択 (性能向上)
client.get("account", select=["name", "creditlimit", "telephone1"])

# 関連レコードを expand
client.get(
    "account",
    expand=["parentaccountid($select=name)"],
    select=["name", "parentaccountid"]
)
```

---

## 11. Cache 管理

### Cache の flush
```python
# 一括操作のあとに SDK 内部 cache をクリア
client.flush_cache()

# 有効な場面:
# - metadata 変更 (table/column の作成)
# - 大量削除
# - metadata の同期
```

---

## 12. Performance ベストプラクティス

### Do's ✅
1. **`select` parameter を使う**: 必要な column だけを取得する
   ```python
   client.get("account", select=["name", "creditlimit"])
   ```

2. **一括操作を使う**: 複数レコードをまとめて作成/更新する
   ```python
   ids = client.create("account", [record1, record2, record3])
   ```

3. **paging を使う**: すべてのレコードを一度に読み込まない
   ```python
   for page in client.get("account", top=1000):
       process_page(page)
   ```

4. **client instance を再利用する**: 1 回作って何度も使う
   ```python
   client = DataverseClient(url, credential)  # Once
   # アプリ全体で再利用
   ```

5. **server 側で filter する**: Dataverse 側で絞り込んでから返してもらう
   ```python
   client.get("account", filter="creditlimit gt 50000")
   ```

### Don'ts ❌
1. **すべての column を取らない**: 必要なものを指定する
   ```python
   # Slow
   client.get("account")
   ```

2. **ループで 1 件ずつ作成しない**: batch 化する
   ```python
   # Slow
   for record in records:
       client.create("account", record)
   ```

3. **すべての結果を一度に読み込まない**: pagination を使う
   ```python
   # Slow
   all_accounts = list(client.get("account"))
   ```

4. **client を繰り返し新規作成しない**: singleton を再利用する
   ```python
   # Inefficient
   for i in range(100):
       client = DataverseClient(url, credential)
   ```

---

## 13. よくあるパターンのまとめ

### Pattern: Upsert (作成または更新)
```python
def upsert_account(name, data):
    """account を作成し、存在する場合は更新する。"""
    try:
        # 既存を探す
        results = list(client.get("account", filter=f"name eq '{name}'"))
        if results:
            account_id = results[0]['accountid']
            client.update("account", account_id, data)
            return account_id, "updated"
        else:
            ids = client.create("account", {"name": name, **data})
            return ids[0], "created"
    except Exception as e:
        print(f"Upsert failed: {e}")
        raise
```

### Pattern: エラー回復付き一括操作
```python
def create_with_recovery(records):
    """レコードを作成し、レコード単位で error を追跡する。"""
    results = {"success": [], "failed": []}

    try:
        ids = client.create("account", records)
        results["success"] = ids
    except Exception as e:
        # 一括が失敗したら 1 件ずつ試す
        for i, record in enumerate(records):
            try:
                ids = client.create("account", record)
                results["success"].append(ids[0])
            except Exception as e:
                results["failed"].append({"index": i, "record": record, "error": str(e)})

    return results
```

---

## 14. 依存関係とバージョン

### Core Dependencies
- **azure-identity** >= 1.17.0 (認証)
- **azure-core** >= 1.30.2 (HTTP client)
- **requests** >= 2.32.0 (HTTP request)
- **Python** >= 3.10

### Optional Dependencies
- **pandas** (データ操作)
- **reportlab** (file example 用 PDF 生成)

### Development Tools
- **pytest** >= 7.0.0 (テスト)
- **black** >= 23.0.0 (コード整形)
- **mypy** >= 1.0.0 (型チェック)
- **ruff** >= 0.1.0 (lint)

---

## 15. よくある問題のトラブルシューティング

### ImportError: No module named 'PowerPlatform'
```bash
# インストール確認
pip show PowerPlatform-Dataverse-Client

# 再インストール
pip install --upgrade PowerPlatform-Dataverse-Client

# 仮想環境が有効か確認
which python  # venv path が表示されるはず
```

### 認証失敗
```python
# credential に Dataverse access があるか確認
# テストにはまず interactive auth を試す
from azure.identity import InteractiveBrowserCredential
credential = InteractiveBrowserCredential(
    tenant_id="your-tenant-id"  # tenant が複数ある場合は指定
)

# org URL 形式を確認
# ✓ https://yourorg.crm.dynamics.com
# ❌ https://yourorg.crm.dynamics.com/
# ❌ https://yourorg.crm4.dynamics.com (regional)
```

### HTTP 429 Rate Limiting
```python
# request 頻度を下げる
# 指数バックオフを実装する (Error Handling セクション参照)
# page size を下げる
client.get("account", top=500)  # 5000 の代わり
```

### MetadataError: Table Not Found
```python
# table の存在確認 (schema name は存在確認では大文字小文字を区別しないが、API では区別する)
tables = client.list_tables()
print([t['table_schema_name'] for t in tables])

# 正確な schema name を使う
table_info = client.get_table_info("new_customprefixed_table")
```

### SQL Query Not Enabled
```python
# query_sql() には org config が必要
# 無効なら OData にフォールバック
try:
    results = client.query_sql("SELECT * FROM account")
except Exception:
    # OData にフォールバック
    results = client.get("account")
```

---

## 参考リンク
- [Official Repository](https://github.com/microsoft/PowerPlatform-DataverseClient-Python)
- [PyPI Package](https://pypi.org/project/PowerPlatform-Dataverse-Client/)
- [Azure Identity Documentation](https://learn.microsoft.com/en-us/python/api/overview/azure/identity-readme)
- [Dataverse Web API Documentation](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/webapi/overview)
