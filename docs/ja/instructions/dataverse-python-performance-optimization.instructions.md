---
applyTo: '**'
---

# Dataverse SDK for Python — パフォーマンスと最適化ガイド

Microsoft 公式の Dataverse および Azure SDK のパフォーマンス ガイダンスに基づいています。

## 1. パフォーマンス概要

Dataverse SDK for Python は Python 開発者向けに最適化されていますが、preview にはいくつか制限があります。
- **Minimal retry policy**: 既定で retry されるのは network error のみ
- **No DeleteMultiple**: 個別 delete を使うか、代わりに status を更新する
- **Limited OData batching**: 汎用 OData batching は未サポート
- **SQL limitations**: JOIN なし、WHERE/TOP/ORDER BY も制限あり

これらの制限には、回避策と最適化戦略で対処します。

---

## 2. Query 最適化

### Select で Column を絞る

```python
# ❌ SLOW - すべての column を取得
accounts = client.get("account", top=100)

# ✅ FAST - 必要な column だけ取得
accounts = client.get(
    "account",
    select=["accountid", "name", "telephone1", "creditlimit"],
    top=100
)
```

**Impact**: payload size と memory usage を 30〜50% 削減します。

---

### Filter を効率的に使う

```python
# ❌ SLOW - 全件取得して Python 側で filter
all_accounts = client.get("account")
active_accounts = [a for a in all_accounts if a.get("statecode") == 0]

# ✅ FAST - server 側で filter
accounts = client.get(
    "account",
    filter="statecode eq 0",
    top=100
)
```

**OData filter の例**:
```python
# Equals
filter="statecode eq 0"

# String contains
filter="contains(name, 'Acme')"

# Multiple conditions
filter="statecode eq 0 and createdon gt 2025-01-01Z"

# Not equals
filter="statecode ne 2"
```

---

### 安定した Paging のために Order by を使う

```python
# pagination で一貫した順序を確保
accounts = client.get(
    "account",
    orderby=["createdon desc", "name asc"],
    page_size=100
)

for page in accounts:
    process_page(page)
```

---

## 3. Pagination のベストプラクティス

### Lazy Pagination (推奨)

```python
# ✅ BEST - generator が 1 ページずつ返す
pages = client.get(
    "account",
    top=5000,              # 総件数上限
    page_size=200          # 1 ページのサイズ (hint)
)

for page in pages:  # 各反復で 1 ページ取得
    for record in page:
        process_record(record)  # すぐ処理する
```

**Benefits**:
- メモリ効率がよい (必要時にページを読み込む)
- 最初の結果が速い
- 必要なら途中で止められる

### 全件をメモリに読み込まない

```python
# ❌ SLOW - 100,000 件を一度に読み込む
all_records = list(client.get("account", top=100000))
process(all_records)

# ✅ FAST - 逐次処理する
for page in client.get("account", top=100000, page_size=5000):
    process(page)
```

---

## 4. Batch 操作

### Bulk Create (推奨)

```python
# ✅ BEST - 複数レコードを 1 回で送る
payloads = [
    {"name": f"Account {i}", "telephone1": f"555-{i:04d}"}
    for i in range(1000)
]
ids = client.create("account", payloads)  # 1 回の API call で多数作成
```

### Bulk Update - Broadcast Mode

```python
# ✅ FAST - 多数レコードに同じ更新を適用
account_ids = ["id1", "id2", "id3", "..."]
client.update("account", account_ids, {"statecode": 1})  # 1 回の call
```

### Bulk Update - Per-Record Mode

```python
# ✅ ACCEPTABLE - レコードごとに異なる更新
account_ids = ["id1", "id2", "id3"]
updates = [
    {"telephone1": "555-0100"},
    {"telephone1": "555-0200"},
    {"telephone1": "555-0300"},
]
client.update("account", account_ids, updates)
```

### Batch Size のチューニング

table の複雑さに応じて (Microsoft ガイダンスより):

| Table Type | Batch Size | Max Threads |
|------------|-----------|-------------|
| OOB (Account, Contact, Lead) | 200-300 | 30 |
| Simple (few lookups) | ≤10 | 50 |
| Moderately complex | ≤100 | 30 |
| Large/complex (>100 cols, >20 lookups) | 10-20 | 10-20 |

```python
def bulk_create_optimized(client, table_name, payloads, batch_size=200):
    """最適な batch size でレコードを作成する。"""
    for i in range(0, len(payloads), batch_size):
        batch = payloads[i:i + batch_size]
        ids = client.create(table_name, batch)
        print(f"Created {len(ids)} records")
        yield ids
```

---

## 5. Connection 管理

### Client Instance を再利用する

```python
# ❌ BAD - 毎回新しい接続を作る
def process_batch():
    for batch in batches:
        client = DataverseClient(...)  # Expensive!
        client.create("account", batch)

# ✅ GOOD - 接続を再利用
client = DataverseClient(...)  # 一度だけ作る

def process_batch():
    for batch in batches:
        client.create("account", batch)  # 再利用
```

### グローバル Client Instance

```python
# singleton_client.py
from azure.identity import DefaultAzureCredential
from PowerPlatform.Dataverse.client import DataverseClient

_client = None

def get_client():
    global _client
    if _client is None:
        _client = DataverseClient(
            base_url="https://myorg.crm.dynamics.com",
            credential=DefaultAzureCredential()
        )
    return _client

# main.py
from singleton_client import get_client

client = get_client()
records = client.get("account")
```

### Connection Timeout 設定

```python
from PowerPlatform.Dataverse.core.config import DataverseConfig

cfg = DataverseConfig()
cfg.http_timeout = 30         # request timeout
cfg.connection_timeout = 5    # connection timeout

client = DataverseClient(
    base_url="https://myorg.crm.dynamics.com",
    credential=credential,
    config=cfg
)
```

---

## 6. Async 操作 (将来機能)

現在は同期的ですが、将来の async に備えます:

```python
# 将来の async support に向けた推奨パターン
import asyncio

async def get_accounts_async(client):
    """将来の async SDK 用パターン。"""
    # SDK が async をサポートしたら:
    # accounts = await client.get("account")
    # 今は executor で sync を使う
    loop = asyncio.get_event_loop()
    accounts = await loop.run_in_executor(
        None,
        lambda: list(client.get("account"))
    )
    return accounts

# Usage
accounts = asyncio.run(get_accounts_async(client))
```

---

## 7. File Upload 最適化

### 小さいファイル (<128 MB)

```python
# ✅ FAST - 単一 request
client.upload_file(
    table_name="account",
    record_id=record_id,
    column_name="document_column",
    file_path="small_file.pdf"
)
```

### 大きいファイル (>128 MB)

```python
# ✅ OPTIMIZED - chunk upload
client.upload_file(
    table_name="account",
    record_id=record_id,
    column_name="document_column",
    file_path="large_file.pdf",
    mode='chunk',
    if_none_match=True
)

# SDK は自動的に:
# 1. ファイルを 4MB chunk に分割
# 2. chunk を並列 upload
# 3. server 側で結合
```

---

## 8. OData Query 最適化

### SQL Alternative (単純クエリ)

```python
# ✅ SOMETIMES FASTER - SELECT のみなら直接 SQL
# 制限: 単一 SELECT、任意の WHERE/TOP/ORDER BY のみ
records = client.get(
    "account",
    sql="SELECT accountid, name FROM account WHERE statecode = 0 ORDER BY name"
)
```

### 複雑クエリ

```python
# ❌ NOT SUPPORTED - JOIN や複雑な WHERE
sql="SELECT a.accountid, c.fullname FROM account a JOIN contact c ON a.accountid = c.parentcustomerid"

# ✅ WORKAROUND - account を取得してから各 account の contact を取得
accounts = client.get("account", select=["accountid", "name"])
for account in accounts:
    contacts = client.get(
        "contact",
        filter=f"parentcustomerid eq '{account['accountid']}'"
    )
    process(account, contacts)
```

---

## 9. メモリ管理

### 大規模データセットを段階的に処理する

```python
import gc

def process_large_table(client, table_name):
    """メモリ問題なく数百万件を処理する。"""

    for page in client.get(table_name, page_size=5000):
        for record in page:
            result = process_record(record)
            save_result(result)

        # ページごとに garbage collection を強制
        gc.collect()
```

### Chunking を使った DataFrame 連携

```python
import pandas as pd

def load_to_dataframe_chunked(client, table_name, chunk_size=10000):
    """データを chunk 単位で DataFrame に読み込む。"""

    dfs = []
    for page in client.get(table_name, page_size=1000):
        df_chunk = pd.DataFrame(page)
        dfs.append(df_chunk)

        # chunk 閾値に達したら結合
        if len(dfs) >= chunk_size // 1000:
            df = pd.concat(dfs, ignore_index=True)
            process_chunk(df)
            dfs = []

    # 残りを処理
    if dfs:
        df = pd.concat(dfs, ignore_index=True)
        process_chunk(df)
```

---

## 10. Rate Limiting の処理

SDK の retry サポートは最小限なので、手動実装します:

```python
import time
from PowerPlatform.Dataverse.core.errors import DataverseError

def call_with_backoff(func, max_retries=3):
    """Rate limit に対する指数バックオフ付きで関数を呼ぶ。"""

    for attempt in range(max_retries):
        try:
            return func()
        except DataverseError as e:
            if e.status_code == 429:  # Too Many Requests
                if attempt < max_retries - 1:
                    wait_time = 2 ** attempt  # 1s, 2s, 4s
                    print(f"Rate limited. Waiting {wait_time}s...")
                    time.sleep(wait_time)
                else:
                    raise
            else:
                raise

# Usage
ids = call_with_backoff(
    lambda: client.create("account", payload)
)
```

---

## 11. Transaction Consistency (既知の制限)

SDK にはトランザクション保証がありません:

```python
# ⚠️ 一括操作が部分失敗すると、一部レコードだけ作成される可能性がある

def create_with_consistency_check(client, table_name, payloads):
    """レコードを作成し、すべて成功したか検証する。"""

    try:
        ids = client.create(table_name, payloads)

        # すべてのレコードが作成されたか確認
        created = client.get(
            table_name,
            filter=f"isof(Microsoft.Dynamics.CRM.{table_name})"
        )

        if len(ids) != count_created:
            print(f"⚠️ Only {count_created}/{len(ids)} records created")
            # 部分失敗を処理
    except Exception as e:
        print(f"Creation failed: {e}")
        # 何が作成されたか確認
```

---

## 12. パフォーマンスの監視

### 操作時間を記録する

```python
import time
import logging

logger = logging.getLogger("dataverse")

def monitored_operation(operation_name):
    """操作パフォーマンスを監視する decorator。"""
    def decorator(func):
        def wrapper(*args, **kwargs):
            start = time.time()
            try:
                result = func(*args, **kwargs)
                duration = time.time() - start
                logger.info(f"{operation_name}: {duration:.2f}s")
                return result
            except Exception as e:
                duration = time.time() - start
                logger.error(f"{operation_name} failed after {duration:.2f}s: {e}")
                raise
        return wrapper
    return decorator

@monitored_operation("Bulk Create Accounts")
def create_accounts(client, payloads):
    return client.create("account", payloads)
```

---

## 13. パフォーマンス チェックリスト

| Item | Status | Notes |
|------|--------|-------|
| client instance を再利用 | ☐ | 1 回作って再利用 |
| select で column を絞る | ☐ | 必要なデータだけ取得 |
| OData で server 側 filter | ☐ | 全件取得してから filter しない |
| page_size 付き pagination | ☐ | 段階的に処理 |
| batch 操作を使う | ☐ | 複数件に create/update を使う |
| table type ごとに batch size 調整 | ☐ | OOB=200-300, Simple=≤10 |
| rate limiting (429) を処理 | ☐ | 指数バックオフを実装 |
| 大きいファイルは chunk upload | ☐ | >128MB は SDK が処理 |
| 操作時間を監視 | ☐ | 分析用に timing を記録 |
| 本番相当データでテスト | ☐ | 性能はデータ量で変わる |

---

## 14. 関連資料

- [Dataverse Web API Performance](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/optimize-performance-create-update)
- [OData Query Options](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/webapi/query-data-web-api)
- [SDK Working with Data](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/sdk-python/work-data)
