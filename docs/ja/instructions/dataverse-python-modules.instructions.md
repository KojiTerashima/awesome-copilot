---
applyTo: '**'
---
# Dataverse SDK for Python — 完全モジュール リファレンス

## パッケージ階層

```
PowerPlatform.Dataverse
├── client
│   └── DataverseClient
├── core
│   ├── config (DataverseConfig)
│   └── errors (DataverseError, ValidationError, MetadataError, HttpError, SQLParseError)
├── data (OData operations, metadata, SQL, file upload)
├── extensions (将来の拡張のための placeholder)
├── models (data model と type のための placeholder)
└── utils (utility と adapter のための placeholder)
```

## core.config Module

client の接続設定と挙動を管理します。

### DataverseConfig Class

language、timeout、retry を保持するコンテナーです。不変です。

```python
from PowerPlatform.Dataverse.core.config import DataverseConfig

cfg = DataverseConfig(
    language_code=1033,        # Default English (US)
    http_retries=None,         # Reserved for future
    http_backoff=None,         # Reserved for future
    http_timeout=None          # Reserved for future
)

# Or use default static builder
cfg_default = DataverseConfig.from_env()
```

**主な属性:**
- `language_code: int = 1033` — ローカライズされた label と message 用の LCID。
- `http_retries: int | None` — (予約済み) 一時的エラーに対する最大 retry 回数。
- `http_backoff: float | None` — (予約済み) retry 間の backoff multiplier。
- `http_timeout: float | None` — (予約済み) 秒単位の request timeout。

## core.errors Module

SDK 操作のための構造化された例外階層です。

### DataverseError (Base)

SDK error の基本例外です。

```python
from PowerPlatform.Dataverse.core.errors import DataverseError

try:
    # SDK call
    pass
except DataverseError as e:
    print(f"Code: {e.code}")                # エラー分類
    print(f"Subcode: {e.subcode}")          # 特定エラー
    print(f"Message: {e.message}")          # 人間が読める内容
    print(f"Status: {e.status_code}")       # HTTP status (該当時)
    print(f"Transient: {e.is_transient}")   # retry 対象か?
    details = e.to_dict()                  # dict に変換
```

### ValidationError

データ操作中の validation failure です。

```python
from PowerPlatform.Dataverse.core.errors import ValidationError
```

### MetadataError

table/column の作成、削除、または参照に失敗したときの例外です。

```python
from PowerPlatform.Dataverse.core.errors import MetadataError

try:
    client.create_table("MyTable", {...})
except MetadataError as e:
    print(f"Metadata issue: {e.message}")
```

### HttpError

Web API の HTTP request failure (4xx、5xx など) です。

```python
from PowerPlatform.Dataverse.core.errors import HttpError

try:
    client.get("account", record_id)
except HttpError as e:
    print(f"HTTP {e.status_code}: {e.message}")
    print(f"Service error code: {e.service_error_code}")
    print(f"Correlation ID: {e.correlation_id}")
    print(f"Request ID: {e.request_id}")
    print(f"Retry-After: {e.retry_after} seconds")
    print(f"Transient (retry?): {e.is_transient}")  # 429, 503, 504
```

### SQLParseError

`query_sql()` 使用時の SQL query 構文エラーです。

```python
from PowerPlatform.Dataverse.core.errors import SQLParseError

try:
    client.query_sql("INVALID SQL HERE")
except SQLParseError as e:
    print(f"SQL parse error: {e.message}")
```

## data Package

低レベルの OData protocol、metadata、SQL、file 操作を扱います (内部委譲)。

`data` package は主に内部用途です。`client` module 内の高レベル `DataverseClient` が、これを包んで次を公開します:
- OData 経由の CRUD 操作
- metadata 管理 (table と column の作成/更新/削除)
- SQL query 実行
- file upload 処理

利用者は `DataverseClient` method (`create()`、`get()`、`update()`、`delete()`、`create_table()`、`query_sql()`、`upload_file()`) を通してこれらとやり取りします。

## extensions Package (Placeholder)

将来の拡張ポイント (例: custom adapter、middleware) 用に予約されています。

現在は空です。現時点の機能には core と client module を使用してください。

## models Package (Placeholder)

将来の data model 定義と型定義のために予約されています。

現在は空です。データ構造は `dict` (OData) として返され、JSON-serializable です。

## utils Package (Placeholder)

utility adapter と helper 用に予約されています。

現在は空です。helper function は将来のリリースで追加される場合があります。

## client Module

ユーザー向けのメイン API です。

### DataverseClient Class

すべての Dataverse 操作のための高レベル client です。

```python
from azure.identity import InteractiveBrowserCredential
from PowerPlatform.Dataverse.client import DataverseClient
from PowerPlatform.Dataverse.core.config import DataverseConfig

# credential を作成
credential = InteractiveBrowserCredential()

# 任意で設定
cfg = DataverseConfig(language_code=1033)

# client を作成
client = DataverseClient(
    base_url="https://org.crm.dynamics.com",
    credential=credential,
    config=cfg  # optional
)
```

#### CRUD メソッド

- `create(table_schema_name, records)` → `list[str]` — レコードを作成し、GUID を返す。
- `get(table_schema_name, record_id=None, select, filter, orderby, top, expand, page_size)` → レコード。
- `update(table_schema_name, ids, changes)` → `None` — レコードを更新する。
- `delete(table_schema_name, ids, use_bulk_delete=True)` → `str | None` — レコードを削除する。

#### Metadata メソッド

- `create_table(table_schema_name, columns, solution_unique_name, primary_column_schema_name)` → metadata dict。
- `create_columns(table_schema_name, columns)` → `list[str]`。
- `delete_columns(table_schema_name, columns)` → `list[str]`。
- `delete_table(table_schema_name)` → `None`。
- `get_table_info(table_schema_name)` → metadata dict または `None`。
- `list_tables()` → `list[str]`。

#### SQL と Utility

- `query_sql(sql)` → `list[dict]` — 読み取り専用 SQL を実行する。
- `upload_file(table_schema_name, record_id, file_name_attribute, path, mode, mime_type, if_none_match)` → `None` — file column へ upload する。
- `flush_cache(kind)` → `int` — SDK cache (`"picklist"` など) をクリアする。

## Import 一覧

```python
# Main client
from PowerPlatform.Dataverse.client import DataverseClient

# Configuration
from PowerPlatform.Dataverse.core.config import DataverseConfig

# Errors
from PowerPlatform.Dataverse.core.errors import (
    DataverseError,
    ValidationError,
    MetadataError,
    HttpError,
    SQLParseError,
)
```

## 参考資料

- Module docs: https://learn.microsoft.com/en-us/python/api/powerplatform-dataverse-client/
- Core: https://learn.microsoft.com/en-us/python/api/powerplatform-dataverse-client/powerplatform.dataverse.core
- Data: https://learn.microsoft.com/en-us/python/api/powerplatform-dataverse-client/powerplatform.dataverse.data
- Extensions: https://learn.microsoft.com/en-us/python/api/powerplatform-dataverse-client/powerplatform.dataverse.extensions
- Models: https://learn.microsoft.com/en-us/python/api/powerplatform-dataverse-client/powerplatform.dataverse.models
- Utils: https://learn.microsoft.com/en-us/python/api/powerplatform-dataverse-client/powerplatform.dataverse.utils
- Client: https://learn.microsoft.com/en-us/python/api/powerplatform-dataverse-client/powerplatform.dataverse.client
