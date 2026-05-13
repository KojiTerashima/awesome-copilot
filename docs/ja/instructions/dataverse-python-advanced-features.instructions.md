# Dataverse SDK for Python - 高度機能ガイド

## 概要
enum、複雑な filtering、SQL query、metadata 操作、本番パターンを含む Dataverse SDK の高度機能に関する包括的ガイドです。Microsoft の公式 walkthrough 例に基づいています。

## 1. Option Set と Picklist の扱い

### 型安全性のために IntEnum を使う
```python
from enum import IntEnum
from PowerPlatform.Dataverse.client import DataverseClient

# picklist 用 enum を定義
class Priority(IntEnum):
    LOW = 1
    MEDIUM = 2
    HIGH = 3

class Priority(IntEnum):
    COLD = 1
    WARM = 2
    HOT = 3

# enum 値を使ってレコードを作成
record_data = {
    "new_title": "Important Task",
    "new_priority": Priority.HIGH,  # 自動的に int に変換される
}

ids = client.create("new_tasktable", record_data)
```

### Formatted Value の扱い
```python
# レコード取得時、picklist 値は整数として返る
record = client.get("new_tasktable", record_id)

priority_int = record.get("new_priority")  # Returns: 3
priority_formatted = record.get("new_priority@OData.Community.Display.V1.FormattedValue")  # Returns: "High"

print(f"Priority (Raw): {priority_int}")
print(f"Priority (Formatted): {priority_formatted}")
```

### Enum Column を持つ Table の作成
```python
from enum import IntEnum

class TaskStatus(IntEnum):
    NOT_STARTED = 0
    IN_PROGRESS = 1
    COMPLETED = 2

class TaskPriority(IntEnum):
    LOW = 1
    MEDIUM = 2
    HIGH = 3

# enum class を column type として渡す
columns = {
    "new_Title": "string",
    "new_Description": "string",
    "new_Status": TaskStatus,      # option set column を作成
    "new_Priority": TaskPriority,  # option set column を作成
    "new_Amount": "decimal",
    "new_DueDate": "datetime"
}

table_info = client.create_table(
    "new_TaskManagement",
    primary_column_schema_name="new_Title",
    columns=columns
)

print(f"Created table with {len(columns)} columns including enums")
```

---

## 2. 高度な Filtering と Query

### 複雑な OData Filter
```python
# Simple equality
filter1 = "name eq 'Contoso'"

# Comparison operators
filter2 = "creditlimit gt 50000"
filter3 = "createdon lt 2024-01-01"

# String operations
filter4 = "contains(name, 'Ltd')"
filter5 = "startswith(name, 'Con')"
filter6 = "endswith(name, 'Ltd')"

# AND 条件
filter7 = "(name eq 'Contoso') and (creditlimit gt 50000)"

# OR 条件
filter8 = "(industrycode eq 1) or (industrycode eq 2)"

# 否定
filter9 = "not(statecode eq 1)"

# 入れ子の複雑条件
filter10 = "(creditlimit gt 50000) and ((industrycode eq 1) or (industrycode eq 2))"

# get() 呼び出しで使用
results = client.get("account", filter=filter10, select=["name", "creditlimit"])
```

### 関連レコード付き取得 (Expand)
```python
# 親 account 情報を expand
accounts = client.get(
    "account",
    filter="creditlimit gt 100000",
    expand=["parentaccountid($select=name,creditlimit)"],
    select=["accountid", "name", "creditlimit", "parentaccountid"]
)

for page in accounts:
    for account in page:
        parent_name = account.get("_parentaccountid_value")
        print(f"Account: {account['name']}, Parent: {parent_name}")
```

### 複雑分析用 SQL Query
```python
# SQL query は読み取り専用だが、分析用途では強力
sql = """
SELECT
    a.name as AccountName,
    a.creditlimit,
    COUNT(c.contactid) as ContactCount
FROM account a
LEFT JOIN contact c ON a.accountid = c.parentcustomerid
WHERE a.creditlimit > 50000
GROUP BY a.accountid, a.name, a.creditlimit
ORDER BY ContactCount DESC
"""

results = client.query_sql(sql)
for row in results:
    print(f"{row['AccountName']}: {row['ContactCount']} contacts")
```

### SQL Query の Paging
```python
# SQL query は既定でページ分割結果を返す
sql = "SELECT TOP 10000 name, creditlimit FROM account ORDER BY name"

all_results = []
for page in client.query_sql(sql):
    all_results.extend(page)
    print(f"Retrieved {len(page)} rows")

print(f"Total: {len(all_results)} rows")
```

---

## 3. Metadata 操作

### 複雑な Table の作成
```python
from enum import IntEnum
from datetime import datetime

class TaskStatus(IntEnum):
    NEW = 1
    OPEN = 2
    CLOSED = 3

# さまざまな column type を持つ table を作成
columns = {
    "new_Subject": "string",
    "new_Description": "string",
    "new_Category": "string",
    "new_Priority": "int",
    "new_Status": TaskStatus,
    "new_EstimatedHours": "decimal",
    "new_DueDate": "datetime",
    "new_IsOverdue": "bool",
    "new_Notes": "string"
}

table_info = client.create_table(
    "new_WorkItem",
    primary_column_schema_name="new_Subject",
    columns=columns
)

print(f"✓ Created table: {table_info['table_schema_name']}")
print(f"  Primary Key: {table_info['primary_id_attribute']}")
print(f"  Columns: {', '.join(table_info.get('columns_created', []))}")
```

### Table Metadata の確認
```python
# 詳細な table 情報を取得
table_info = client.get_table_info("account")

print(f"Schema Name: {table_info.get('table_schema_name')}")
print(f"Logical Name: {table_info.get('table_logical_name')}")
print(f"Display Name: {table_info.get('table_display_name')}")
print(f"Entity Set: {table_info.get('entity_set_name')}")
print(f"Primary ID: {table_info.get('primary_id_attribute')}")
print(f"Primary Name: {table_info.get('primary_name_attribute')}")
```

### 組織内の全 Table を一覧取得
```python
# すべての table を取得 (結果セットが大きい場合あり)
all_tables = []
for page in client.list_tables():
    all_tables.extend(page)
    print(f"Retrieved {len(page)} tables in this page")

print(f"\nTotal tables: {len(all_tables)}")

# custom table だけ絞り込み
custom_tables = [t for t in all_tables if t['table_schema_name'].startswith('new_')]
print(f"Custom tables: {len(custom_tables)}")
for table in custom_tables[:5]:
    print(f"  - {table['table_schema_name']}")
```

### 動的な Column 管理
```python
# 既存 table に column を追加
client.create_columns("new_TaskTable", {
    "new_Department": "string",
    "new_Budget": "decimal",
    "new_ApprovedDate": "datetime"
})

# 特定 column を削除
client.delete_columns("new_TaskTable", [
    "new_OldField1",
    "new_OldField2"
])

# Table 全体を削除
client.delete_table("new_TaskTable")
```

---

## 4. 単一レコード操作と複数レコード操作

### 単一レコード操作
```python
# Create single
record_id = client.create("account", {"name": "Contoso"})[0]

# ID 指定で 1 件取得
account = client.get("account", record_id)

# 1 件更新
client.update("account", record_id, {"creditlimit": 100000})

# 1 件削除
client.delete("account", record_id)
```

### 複数レコード操作

#### 複数レコードの作成
```python
# レコードのリストを作成
records = [
    {"name": "Company A", "creditlimit": 50000},
    {"name": "Company B", "creditlimit": 75000},
    {"name": "Company C", "creditlimit": 100000},
]

created_ids = client.create("account", records)
print(f"Created {len(created_ids)} records: {created_ids}")
```

#### 複数レコードの更新 (同報更新)
```python
# 複数レコードに同じ更新を適用
account_ids = ["id1", "id2", "id3"]
client.update("account", account_ids, {
    "industrycode": 1,  # Retail
    "accountmanagerid": "manager-guid"
})
print(f"Updated {len(account_ids)} records with same data")
```

#### 複数レコードの削除
```python
# 最適化された bulk delete で複数削除
record_ids = ["id1", "id2", "id3", "id4", "id5"]
client.delete("account", record_ids, use_bulk_delete=True)
print(f"Deleted {len(record_ids)} records")
```

---

## 5. データ操作パターン

### 取得 → 変更 → 更新 パターン
```python
# 1 件取得
account = client.get("account", record_id)

# ローカルで変更
original_amount = account.get("creditlimit", 0)
new_amount = original_amount + 10000

# 更新を戻す
client.update("account", record_id, {"creditlimit": new_amount})
print(f"Updated creditlimit: {original_amount} → {new_amount}")
```

### Batch 処理パターン
```python
# paging 付きで batch 取得
batch_size = 100
processed = 0

for page in client.get("account", top=batch_size, filter="statecode eq 0"):
    # 各ページを処理
    batch_updates = []
    for account in page:
        if account.get("creditlimit", 0) > 100000:
            batch_updates.append({
                "id": account['accountid'],
                "accountmanagerid": "senior-manager-guid"
            })

    # Batch update
    for update in batch_updates:
        client.update("account", update['id'], {"accountmanagerid": update['accountmanagerid']})
        processed += 1

print(f"Processed {processed} accounts")
```

### 条件付き操作パターン
```python
from PowerPlatform.Dataverse.core.errors import DataverseError

def safe_update(table, record_id, data, check_field=None, check_value=None):
    """事前条件チェック付きで更新する。"""
    try:
        if check_field and check_value:
            # 更新前に条件を確認
            record = client.get(table, record_id, select=[check_field])
            if record.get(check_field) != check_value:
                print(f"Condition not met: {check_field} != {check_value}")
                return False

        client.update(table, record_id, data)
        return True
    except DataverseError as e:
        print(f"Update failed: {e}")
        return False

# Usage
safe_update("account", account_id, {"creditlimit": 100000}, "statecode", 0)
```

---

## 6. Formatted Value と表示

### Formatted Value の取得
```python
# option set や money field を含むレコードを取得すると、
# 表示用 formatted value も利用できる

record = client.get(
    "account",
    record_id,
    select=["name", "creditlimit", "industrycode"]
)

# 生の値
name = record.get("name")  # "Contoso Ltd"
limit = record.get("creditlimit")  # 100000.00
industry = record.get("industrycode")  # 1

# formatted value (OData response に含まれる)
limit_formatted = record.get("creditlimit@OData.Community.Display.V1.FormattedValue")
industry_formatted = record.get("industrycode@OData.Community.Display.V1.FormattedValue")

print(f"Name: {name}")
print(f"Credit Limit: {limit_formatted or limit}")  # "100,000.00" or 100000.00
print(f"Industry: {industry_formatted or industry}")  # "Technology" or 1
```

---

## 7. パフォーマンス最適化

### Column 選択戦略
```python
# ❌ すべての column を取得 (遅い、帯域も多く使う)
account = client.get("account", record_id)

# ✅ 必要な column だけ取得 (高速、効率的)
account = client.get(
    "account",
    record_id,
    select=["accountid", "name", "creditlimit", "telephone1"]
)
```

### Server 側で filter する
```python
# ❌ 全件取得してローカルで filter (非効率)
all_accounts = []
for page in client.get("account"):
    all_accounts.extend(page)
large_accounts = [a for a in all_accounts if a.get("creditlimit", 0) > 100000]

# ✅ server 側で filter し、一致分だけ取得 (効率的)
large_accounts = []
for page in client.get("account", filter="creditlimit gt 100000"):
    large_accounts.extend(page)
```

### 大量結果の Paging
```python
# ❌ 一度に全件読み込み (メモリ負荷大)
all_accounts = list(client.get("account"))

# ✅ ページ単位で処理 (メモリ効率が高い)
processed = 0
for page in client.get("account", top=1000):
    for account in page:
        process_account(account)
        processed += 1
    print(f"Processed: {processed}")
```

### Batch 操作
```python
# ❌ ループで 1 件ずつ create (遅い)
for account_data in accounts:
    client.create("account", account_data)

# ✅ Batch create (高速、最適化済み)
created_ids = client.create("account", accounts)
```

---

## 8. 高度シナリオでのエラー処理

### Metadata Error の処理
```python
from PowerPlatform.Dataverse.core.errors import MetadataError

try:
    table_info = client.create_table("new_CustomTable", {"name": "string"})
except MetadataError as e:
    print(f"Metadata operation failed: {e}")
    # table 作成固有の error を処理
```

### Validation Error の処理
```python
from PowerPlatform.Dataverse.core.errors import ValidationError

try:
    client.create("account", {"name": None})  # Invalid: name required
except ValidationError as e:
    print(f"Validation error: {e}")
    # validation 固有の error を処理
```

### HTTP Error の処理
```python
from PowerPlatform.Dataverse.core.errors import HttpError

try:
    client.get("account", "invalid-guid")
except HttpError as e:
    if "404" in str(e):
        print("Record not found")
    elif "403" in str(e):
        print("Access denied")
    else:
        print(f"HTTP error: {e}")
```

### SQL Error の処理
```python
from PowerPlatform.Dataverse.core.errors import SQLParseError

try:
    results = client.query_sql("SELECT INVALID SYNTAX")
except SQLParseError as e:
    print(f"SQL parse error: {e}")
```

---

## 9. 関連の扱い

### 関連レコードの作成
```python
# 親 account を作成
parent_ids = client.create("account", {
    "name": "Parent Company",
    "creditlimit": 500000
})
parent_id = parent_ids[0]

# 親参照付きで子 account を作成
children = [
    {"name": "Subsidiary A", "parentaccountid": parent_id},
    {"name": "Subsidiary B", "parentaccountid": parent_id},
    {"name": "Subsidiary C", "parentaccountid": parent_id},
]
child_ids = client.create("account", children)
print(f"Created {len(child_ids)} child accounts")
```

### 関連レコードのクエリ
```python
# 子 account を持つ account を取得
account = client.get("account", account_id)

# 子 account をクエリ
children = client.get(
    "account",
    filter=f"parentaccountid eq {account_id}",
    select=["accountid", "name", "creditlimit"]
)

for page in children:
    for child in page:
        print(f"  - {child['name']}: ${child['creditlimit']}")
```

---

## 10. Cleanup と Housekeeping

### SDK Cache のクリア
```python
# 一括操作後に metadata cache をクリア
client.flush_cache()

# 有効な場面:
# - 大量 delete 操作のあと
# - table/column の作成または削除
# - 環境間の metadata 同期
```

### 安全な Table 削除
```python
from PowerPlatform.Dataverse.core.errors import MetadataError

def delete_table_safe(table_name):
    """エラー処理付きで table を削除する。"""
    try:
        # table の存在確認
        table_info = client.get_table_info(table_name)
        if not table_info:
            print(f"Table {table_name} not found")
            return False

        # 削除
        client.delete_table(table_name)
        print(f"✓ Deleted table: {table_name}")

        # cache をクリア
        client.flush_cache()
        return True

    except MetadataError as e:
        print(f"❌ Failed to delete table: {e}")
        return False

delete_table_safe("new_TempTable")
```

---

## 11. 包括例: 完全ワークフロー

```python
from enum import IntEnum
from azure.identity import InteractiveBrowserCredential
from PowerPlatform.Dataverse.client import DataverseClient
from PowerPlatform.Dataverse.core.errors import DataverseError, MetadataError

class TaskStatus(IntEnum):
    NEW = 1
    IN_PROGRESS = 2
    COMPLETED = 3

class TaskPriority(IntEnum):
    LOW = 1
    MEDIUM = 2
    HIGH = 3

# Setup
credential = InteractiveBrowserCredential()
client = DataverseClient("https://yourorg.crm.dynamics.com", credential)

try:
    # 1. table を作成
    print("Creating table...")
    table_info = client.create_table(
        "new_ProjectTask",
        primary_column_schema_name="new_Title",
        columns={
            "new_Description": "string",
            "new_Status": TaskStatus,
            "new_Priority": TaskPriority,
            "new_DueDate": "datetime",
            "new_EstimatedHours": "decimal"
        }
    )
    print(f"✓ Created table: {table_info['table_schema_name']}")

    # 2. レコードを作成
    print("\nCreating tasks...")
    tasks = [
        {
            "new_Title": "Design system",
            "new_Description": "Create design system architecture",
            "new_Status": TaskStatus.NEW,
            "new_Priority": TaskPriority.HIGH,
            "new_EstimatedHours": 40.0
        },
        {
            "new_Title": "Implement UI",
            "new_Description": "Build React components",
            "new_Status": TaskStatus.IN_PROGRESS,
            "new_Priority": TaskPriority.HIGH,
            "new_EstimatedHours": 80.0
        },
        {
            "new_Title": "Write tests",
            "new_Description": "Unit and integration tests",
            "new_Status": TaskStatus.NEW,
            "new_Priority": TaskPriority.MEDIUM,
            "new_EstimatedHours": 30.0
        }
    ]
    task_ids = client.create("new_ProjectTask", tasks)
    print(f"✓ Created {len(task_ids)} tasks")

    # 3. query と filter
    print("\nQuerying high-priority tasks...")
    high_priority = client.get(
        "new_ProjectTask",
        filter="new_priority eq 3",
        select=["new_Title", "new_Priority", "new_EstimatedHours"]
    )
    for page in high_priority:
        for task in page:
            print(f"  - {task['new_title']}: {task['new_estimatedhours']} hours")

    # 4. レコード更新
    print("\nUpdating task status...")
    client.update("new_ProjectTask", task_ids[1], {
        "new_Status": TaskStatus.COMPLETED,
        "new_EstimatedHours": 85.5
    })
    print("✓ Updated task status")

    # 5. Cleanup
    print("\nCleaning up...")
    client.delete_table("new_ProjectTask")
    print("✓ Deleted table")

    # cache をクリア
    client.flush_cache()

except (MetadataError, DataverseError) as e:
    print(f"❌ Error: {e}")
```

---

## 参考資料
- [Official Walkthrough Example](https://github.com/microsoft/PowerPlatform-DataverseClient-Python/blob/main/examples/advanced/walkthrough.py)
- [OData Filter Syntax](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/webapi/query-data-web-api)
- [Table/Column Metadata](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/webapi/create-update-entity-definitions-using-web-api)
