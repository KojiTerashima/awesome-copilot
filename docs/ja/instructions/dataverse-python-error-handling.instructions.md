---
applyTo: '**'
---

# Dataverse SDK for Python — エラー処理とトラブルシューティング ガイド

Azure SDK のエラー処理パターンと Dataverse SDK 固有事項に関する Microsoft 公式ドキュメントに基づいています。

## 1. DataverseError Class 概要

Dataverse SDK for Python は、堅牢なエラー処理のために構造化された例外階層を提供します。

### DataverseError Constructor

```python
from PowerPlatform.Dataverse.core.errors import DataverseError

DataverseError(
    message: str,                          # 人が読めるエラーメッセージ
    code: str,                             # エラー分類 (例: "validation_error", "http_error")
    subcode: str | None = None,            # 任意の特定エラー識別子
    status_code: int | None = None,        # HTTP status code (該当時)
    details: Dict[str, Any] | None = None, # 追加の診断情報
    source: str | None = None,             # エラー source: "client" または "server"
    is_transient: bool = False             # retry で成功する可能性があるか
)
```

### 主なプロパティ

```python
try:
    client.get("account", record_id="invalid-id")
except DataverseError as e:
    print(f"Message: {e.message}")           # 人が読めるメッセージ
    print(f"Code: {e.code}")                 # エラー分類
    print(f"Subcode: {e.subcode}")           # 特定エラー種別
    print(f"Status Code: {e.status_code}")   # HTTP status (401, 403, 429 など)
    print(f"Source: {e.source}")             # "client" または "server"
    print(f"Is Transient: {e.is_transient}") # retry 可能か?
    print(f"Details: {e.details}")           # 追加 context

    # logging 用に辞書へ変換
    error_dict = e.to_dict()
```

---

## 2. よくあるエラー シナリオ

### 認証エラー (401)

**Cause**: 無効な credential、期限切れ token、または設定ミス。

```python
from PowerPlatform.Dataverse.client import DataverseClient
from PowerPlatform.Dataverse.core.errors import DataverseError
from azure.identity import InteractiveBrowserCredential

try:
    # 無効な credential または期限切れ token
    credential = InteractiveBrowserCredential()
    client = DataverseClient(
        base_url="https://invalid-org.crm.dynamics.com",
        credential=credential
    )
    records = client.get("account")
except DataverseError as e:
    if e.status_code == 401:
        print("Authentication failed. Check credentials and token expiration.")
        print(f"Details: {e.message}")
        # retry しない - まず credential を修正する
    else:
        raise
```

### 認可エラー (403)

**Cause**: 要求された操作に対する permission が user にない。

```python
try:
    # User に contact の読み取り権限がない
    records = client.get("contact")
except DataverseError as e:
    if e.status_code == 403:
        print("Access denied. User lacks required permissions.")
        print(f"Request ID for support: {e.details.get('request_id')}")
        # 管理者へエスカレーション
    else:
        raise
```

### Resource Not Found (404)

**Cause**: レコード、table、または resource が存在しない。

```python
try:
    # レコードが存在しない
    record = client.get("account", record_id="00000000-0000-0000-0000-000000000000")
except DataverseError as e:
    if e.status_code == 404:
        print("Resource not found. Using default data.")
        record = {"name": "Unknown", "id": None}
    else:
        raise
```

### Rate Limiting (429)

**Cause**: サービス保護制限を超えるリクエスト過多。

**Note**: SDK の組み込み retry サポートは最小限です。一時的な整合性問題は手動で扱います。

```python
import time

def create_with_retry(client, table_name, payload, max_retries=3):
    """Rate limiting 用 retry ロジック付きでレコードを作成する。"""
    for attempt in range(max_retries):
        try:
            result = client.create(table_name, payload)
            return result
        except DataverseError as e:
            if e.status_code == 429 and e.is_transient:
                wait_time = 2 ** attempt  # 指数バックオフ
                print(f"Rate limited. Retrying in {wait_time}s...")
                time.sleep(wait_time)
            else:
                raise

    raise Exception(f"Failed after {max_retries} retries")
```

### サーバー エラー (500, 502, 503, 504)

**Cause**: 一時的なサービス障害またはインフラ問題。

```python
try:
    result = client.create("account", {"name": "Acme"})
except DataverseError as e:
    if 500 <= e.status_code < 600:
        print(f"Server error ({e.status_code}). Service may be temporarily unavailable.")
        # 指数バックオフ付き retry ロジックを実装
    else:
        raise
```

### Validation Error (400)

**Cause**: 無効な request format、必須 field の不足、または business rule 違反。

```python
try:
    # 必須 field 不足または無効データ
    client.create("account", {"telephone1": "not-a-phone-number"})
except DataverseError as e:
    if e.status_code == 400:
        print(f"Validation error: {e.message}")
        if e.details:
            print(f"Details: {e.details}")
        # デバッグ用に validation issue を記録
    else:
        raise
```

---

## 3. エラー処理のベストプラクティス

### 具体的な例外処理を使う

常に一般例外より先に具体例外を捕捉します。

```python
from PowerPlatform.Dataverse.core.errors import DataverseError
from azure.core.exceptions import AzureError

try:
    records = client.get("account", filter="statecode eq 0", top=100)
except DataverseError as e:
    # Dataverse 固有の error を処理
    if e.status_code == 401:
        print("Re-authenticate required")
    elif e.status_code == 404:
        print("Resource not found")
    elif e.is_transient:
        print("Transient error - may retry")
    else:
        print(f"Operation failed: {e.message}")
except AzureError as e:
    # Azure SDK error を処理 (network、auth など)
    print(f"Azure error: {e}")
except Exception as e:
    # 想定外 error の catch-all
    print(f"Unexpected error: {e}")
```

### 賢い Retry ロジックを実装する

**次では retry しない**:
- 401 Unauthorized (認証失敗)
- 403 Forbidden (認可失敗)
- 400 Bad Request (client error)
- 404 Not Found (resource が最終的に出現する想定でない限り)

**次では retry を検討する**:
- 408 Request Timeout
- 429 Too Many Requests (指数バックオフ付き)
- 500 Internal Server Error
- 502 Bad Gateway
- 503 Service Unavailable
- 504 Gateway Timeout

```python
def should_retry(error: DataverseError) -> bool:
    """操作を retry すべきか判定する。"""
    if not error.is_transient:
        return False

    retryable_codes = {408, 429, 500, 502, 503, 504}
    return error.status_code in retryable_codes

def call_with_exponential_backoff(func, *args, max_attempts=3, **kwargs):
    """指数バックオフ retry 付きで関数を呼び出す。"""
    for attempt in range(max_attempts):
        try:
            return func(*args, **kwargs)
        except DataverseError as e:
            if should_retry(e) and attempt < max_attempts - 1:
                wait_time = 2 ** attempt  # 1s, 2s, 4s...
                print(f"Attempt {attempt + 1} failed. Retrying in {wait_time}s...")
                time.sleep(wait_time)
            else:
                raise
```

### 意味のあるエラー情報を抽出する

```python
import json
from datetime import datetime

def log_error_for_support(error: DataverseError):
    """診断情報付きで error を記録する。"""
    error_info = {
        "timestamp": datetime.utcnow().isoformat(),
        "error_type": type(error).__name__,
        "message": error.message,
        "code": error.code,
        "subcode": error.subcode,
        "status_code": error.status_code,
        "source": error.source,
        "is_transient": error.is_transient,
        "details": error.details
    }

    print(json.dumps(error_info, indent=2))

    # log file に保存、または monitoring service へ送信
    return error_info
```

### 一括操作を適切に扱う

```python
def bulk_create_with_error_tracking(client, table_name, payloads):
    """複数レコードを作成し、成功/失敗を追跡する。"""
    results = {
        "succeeded": [],
        "failed": []
    }

    for idx, payload in enumerate(payloads):
        try:
            record_ids = client.create(table_name, payload)
            results["succeeded"].append({
                "payload": payload,
                "ids": record_ids
            })
        except DataverseError as e:
            results["failed"].append({
                "index": idx,
                "payload": payload,
                "error": {
                    "message": e.message,
                    "code": e.code,
                    "status": e.status_code
                }
            })

    return results
```

---

## 4. 診断 Logging を有効にする

### Logging を設定する

```python
import logging
import sys

# root logger を設定
logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('dataverse_sdk.log'),
        logging.StreamHandler(sys.stdout)
    ]
)

# 特定 logger を設定
logging.getLogger('azure').setLevel(logging.DEBUG)
logging.getLogger('PowerPlatform').setLevel(logging.DEBUG)

# HTTP logging (機微データに注意)
logging.getLogger('azure.core.pipeline.policies.http_logging_policy').setLevel(logging.DEBUG)
```

### SDK レベル Logging を有効化

```python
from PowerPlatform.Dataverse.client import DataverseClient
from PowerPlatform.Dataverse.core.config import DataverseConfig
from azure.identity import InteractiveBrowserCredential

cfg = DataverseConfig()
cfg.logging_enable = True  # 詳細 logging を有効化

client = DataverseClient(
    base_url="https://myorg.crm.dynamics.com",
    credential=InteractiveBrowserCredential(),
    config=cfg
)

# これで SDK が詳細な HTTP request/response を記録する
records = client.get("account", top=10)
```

### Error Response を解析する

```python
import json

try:
    client.create("account", invalid_payload)
except DataverseError as e:
    # 構造化された error detail を抽出
    if e.details and isinstance(e.details, dict):
        error_code = e.details.get('error', {}).get('code')
        error_message = e.details.get('error', {}).get('message')

        print(f"Error Code: {error_code}")
        print(f"Error Message: {error_message}")

        # 一部 error には入れ子の details がある
        if 'error' in e.details and 'details' in e.details['error']:
            for detail in e.details['error']['details']:
                print(f"  - {detail.get('code')}: {detail.get('message')}")
```

---

## 5. Dataverse 固有のエラー処理

### OData Query Error を扱う

```python
try:
    # 無効な OData filter
    records = client.get(
        "account",
        filter="invalid_column eq 0"
    )
except DataverseError as e:
    if "invalid column" in e.message.lower():
        print("Check OData column names and syntax")
    else:
        print(f"Query error: {e.message}")
```

### File Upload Error を扱う

```python
try:
    client.upload_file(
        table_name="account",
        record_id=record_id,
        column_name="document_column",
        file_path="large_file.pdf"
    )
except DataverseError as e:
    if e.status_code == 413:
        print("File too large. Use chunked upload mode.")
    elif e.status_code == 400:
        print("Invalid column or file format.")
    else:
        raise
```

### Table Metadata 操作を扱う

```python
try:
    # custom table を作成
    table_def = {
        "SchemaName": "new_CustomTable",
        "DisplayName": "Custom Table"
    }
    client.create("EntityMetadata", table_def)
except DataverseError as e:
    if "already exists" in e.message:
        print("Table already exists")
    elif "permission" in e.message.lower():
        print("Insufficient permissions to create tables")
    else:
        raise
```

---

## 6. 監視とアラート

### 監視付きで Client 呼び出しを包む

```python
from functools import wraps
import time

def monitor_operation(operation_name):
    """SDK 操作を監視する decorator。"""
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            start_time = time.time()
            try:
                result = func(*args, **kwargs)
                duration = time.time() - start_time
                print(f"✓ {operation_name} completed in {duration:.2f}s")
                return result
            except DataverseError as e:
                duration = time.time() - start_time
                print(f"✗ {operation_name} failed after {duration:.2f}s")
                print(f"  Error: {e.code} ({e.status_code}): {e.message}")
                raise
        return wrapper
    return decorator

@monitor_operation("Fetch Accounts")
def get_accounts(client):
    return client.get("account", top=100)

# Usage
try:
    accounts = get_accounts(client)
except DataverseError:
    print("Operation failed - check logs for details")
```

---

## 7. よくあるトラブルシューティング チェックリスト

| Issue | Diagnosis | Solution |
|-------|-----------|----------|
| 401 Unauthorized | 期限切れ token または無効な credential | 有効な credential で再認証する |
| 403 Forbidden | User に permission がない | 管理者へアクセス権を依頼する |
| 404 Not Found | レコード/table が存在しない | schema name と record ID を確認する |
| 429 Rate Limited | リクエストが多すぎる | 指数バックオフ retry を実装する |
| 500+ Server Error | サービス障害 | 指数バックオフ付き retry、status page の確認 |
| 400 Bad Request | request format が不正 | OData 構文、field 名、必須 field を確認する |
| Network timeout | 接続問題 | ネットワーク確認、DataverseConfig で timeout を増やす |
| InvalidOperationException | plugin/workflow error | Dataverse 内の plugin log を確認する |

---

## 8. Logging ベストプラクティス

```python
import logging
import json
from datetime import datetime

class DataverseErrorHandler:
    """集中化されたエラー処理と logging。"""

    def __init__(self, log_file="dataverse_errors.log"):
        self.logger = logging.getLogger("DataverseSDK")
        handler = logging.FileHandler(log_file)
        formatter = logging.Formatter(
            '%(asctime)s - %(levelname)s - %(message)s'
        )
        handler.setFormatter(formatter)
        self.logger.addHandler(handler)
        self.logger.setLevel(logging.ERROR)

    def log_error(self, error: DataverseError, context: str = ""):
        """デバッグ用 context とともに error を記録する。"""
        error_record = {
            "timestamp": datetime.utcnow().isoformat(),
            "context": context,
            "error": error.to_dict()
        }

        self.logger.error(json.dumps(error_record, indent=2))

    def is_retryable(self, error: DataverseError) -> bool:
        """Error を retry すべきか確認する。"""
        return error.is_transient and error.status_code in {408, 429, 500, 502, 503, 504}

# Usage
error_handler = DataverseErrorHandler()

try:
    client.create("account", payload)
except DataverseError as e:
    error_handler.log_error(e, "create_account_batch_1")
    if error_handler.is_retryable(e):
        print("Will retry this operation")
    else:
        print("Operation failed permanently")
```

---

## 9. 関連資料

- [DataverseError API Reference](https://learn.microsoft.com/en-us/python/api/powerplatform-dataverse-client/powerplatform.dataverse.core.errors.dataverseerror)
- [Azure SDK Error Handling](https://learn.microsoft.com/en-us/azure/developer/python/sdk/fundamentals/errors)
- [Dataverse SDK Getting Started](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/sdk-python/get-started)
- [Service Protection API Limits](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/optimize-performance-create-update)
