---
applyTo: '**'
---
# Dataverse SDK for Python — API リファレンス ガイド

## DataverseClient Class
Dataverse とやり取りするためのメイン client です。base URL と Azure credential で初期化します。

### 主なメソッド

#### create(table_schema_name, records)
単一または一括でレコードを作成します。GUID のリストを返します。

```python
# 単一レコード
ids = client.create("account", {"name": "Acme"})
print(ids[0])  # 最初の GUID

# 一括作成
ids = client.create("account", [{"name": "Contoso"}, {"name": "Fabrikam"}])
```

#### get(table_schema_name, record_id=None, select, filter, orderby, top, expand, page_size)
単一レコードを取得するか、OData オプションを使って複数レコードをクエリします。

```python
# 単一レコード
record = client.get("account", record_id="guid-here")

# filter と paging を使ったクエリ
for batch in client.get(
    "account",
    filter="statecode eq 0",
    select=["name", "telephone1"],
    orderby=["createdon desc"],
    top=100,
    page_size=50
):
    for record in batch:
        print(record["name"])
```

#### update(table_schema_name, ids, changes)
単一または一括でレコードを更新します。

```python
# 単一更新
client.update("account", "guid-here", {"telephone1": "555-0100"})

# Broadcast: 同じ変更を複数 ID に適用
client.update("account", [id1, id2, id3], {"statecode": 1})

# Paired: 1 対 1 の対応付け
client.update("account", [id1, id2], [{"name": "A"}, {"name": "B"}])
```

#### delete(table_schema_name, ids, use_bulk_delete=True)
単一または一括でレコードを削除します。

```python
# 単一削除
client.delete("account", "guid-here")

# 一括削除 (async)
job_id = client.delete("account", [id1, id2, id3])
```

#### create_table(table_schema_name, columns, solution_unique_name=None, primary_column_schema_name=None)
custom table を作成します。

```python
from enum import IntEnum

class ItemStatus(IntEnum):
    ACTIVE = 1
    INACTIVE = 2
    __labels__ = {
        1033: {"ACTIVE": "Active", "INACTIVE": "Inactive"}
    }

info = client.create_table("new_MyTable", {
    "new_Title": "string",
    "new_Quantity": "int",
    "new_Price": "decimal",
    "new_Active": "bool",
    "new_Status": ItemStatus
})
print(info["entity_logical_name"])
```

#### create_columns(table_schema_name, columns)
既存 table に column を追加します。

```python
created = client.create_columns("new_MyTable", {
    "new_Notes": "string",
    "new_Count": "int"
})
```

#### delete_columns(table_schema_name, columns)
table から column を削除します。

```python
removed = client.delete_columns("new_MyTable", ["new_Notes", "new_Count"])
```

#### delete_table(table_schema_name)
custom table を削除します (元に戻せません)。

```python
client.delete_table("new_MyTable")
```

#### get_table_info(table_schema_name)
table metadata を取得します。

```python
info = client.get_table_info("new_MyTable")
if info:
    print(info["table_logical_name"])
    print(info["entity_set_name"])
```

#### list_tables()
すべての custom table を一覧表示します。

```python
tables = client.list_tables()
for table in tables:
    print(table)
```

#### flush_cache(kind)
SDK cache (例: picklist label) をクリアします。

```python
removed = client.flush_cache("picklist")
```

## DataverseConfig Class
client の挙動 (timeout、retry、language) を設定します。

```python
from PowerPlatform.Dataverse.core.config import DataverseConfig

cfg = DataverseConfig()
cfg.http_retries = 3
cfg.http_backoff = 1.0
cfg.http_timeout = 30
cfg.language_code = 1033  # 英語

client = DataverseClient(base_url=url, credential=cred, config=cfg)
```

## エラー処理
SDK 固有例外には `DataverseError` を捕捉します。retry すべきかは `is_transient` を確認します。

```python
from PowerPlatform.Dataverse.core.errors import DataverseError

try:
    client.create("account", {"name": "Test"})
except DataverseError as e:
    print(f"Code: {e.code}")
    print(f"Message: {e.message}")
    print(f"Transient: {e.is_transient}")
    print(f"Details: {e.to_dict()}")
```

## OData Filter のヒント
- filter expression では正確な logical name (lowercase) を使う
- `select` 内の column 名は自動的に lowercase 化される
- `expand` 内の navigation property 名は大文字小文字を区別する

## 参考資料
- API docs: https://learn.microsoft.com/en-us/python/api/powerplatform-dataverse-client/powerplatform.dataverse.client.dataverseclient
- Config docs: https://learn.microsoft.com/en-us/python/api/powerplatform-dataverse-client/powerplatform.dataverse.core.config.dataverseconfig
- Errors: https://learn.microsoft.com/en-us/python/api/powerplatform-dataverse-client/powerplatform.dataverse.core.errors
