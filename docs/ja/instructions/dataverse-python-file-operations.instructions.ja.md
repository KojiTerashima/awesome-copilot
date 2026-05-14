# Dataverse SDK for Python - ファイル操作と実践例

## 概要
PowerPlatform-DataverseClient-Python SDK を使用したファイル upload 操作、chunking 戦略、実用的な現場例をまとめた完全ガイドです。

---

## 1. ファイル Upload の基礎

### 小さいファイルの Upload (< 128 MB)
```python
from pathlib import Path
from PowerPlatform.Dataverse.client import DataverseClient

file_path = Path("document.pdf")
record_id = "account-guid"

# 小さいファイル向けの単一 PATCH upload
response = client.upload_file(
    table_name="account",
    record_id=record_id,
    file_column_name="new_documentfile",
    file_path=file_path
)

print(f"Upload successful: {response}")
```

**利用場面:** 128 MB 未満のドキュメント、画像、PDF

### Chunking を使う大きいファイル Upload
```python
from pathlib import Path

file_path = Path("large_video.mp4")
record_id = "account-guid"

# SDK が大きいファイルの chunking を自動処理
response = client.upload_file(
    table_name="account",
    record_id=record_id,
    file_column_name="new_videofile",
    file_path=file_path,
    chunk_size=4 * 1024 * 1024  # 4 MB chunk
)

print("Chunked upload complete")
```

**利用場面:** 大きな動画、database、archive など 128 MB 超

### 進捗追跡付き Upload
```python
import hashlib
from pathlib import Path

def calculate_file_hash(file_path):
    """ファイルの SHA-256 hash を計算する。"""
    hash_obj = hashlib.sha256()
    with open(file_path, 'rb') as f:
        for chunk in iter(lambda: f.read(1024*1024), b''):
            hash_obj.update(chunk)
    return hash_obj.hexdigest()

def upload_with_tracking(client, table_name, record_id, column_name, file_path):
    """検証追跡付きでファイルを upload する。"""
    file_path = Path(file_path)
    file_size = file_path.stat().st_size

    print(f"Starting upload: {file_path.name} ({file_size / 1024 / 1024:.2f} MB)")

    # upload 前に hash を計算
    original_hash = calculate_file_hash(file_path)
    print(f"File hash: {original_hash}")

    # upload 実行
    response = client.upload_file(
        table_name=table_name,
        record_id=record_id,
        file_column_name=column_name,
        file_path=file_path
    )

    print(f"✓ Upload complete")
    return response

# Usage
upload_with_tracking(client, "account", account_id, "new_documentfile", "report.pdf")
```

---

## 2. Upload 戦略と設定

### 自動 Chunking 判定
```python
def upload_file_smart(client, table_name, record_id, column_name, file_path):
    """戦略を自動選択して upload する。"""
    file_path = Path(file_path)
    file_size = file_path.stat().st_size
    max_single_patch = 128 * 1024 * 1024  # 128 MB

    if file_size <= max_single_patch:
        print(f"Using single PATCH (file < 128 MB)")
        chunk_size = None  # SDK が単一 request を使用
    else:
        print(f"Using chunked upload (file > 128 MB)")
        chunk_size = 4 * 1024 * 1024  # 4 MB chunk

    response = client.upload_file(
        table_name=table_name,
        record_id=record_id,
        file_column_name=column_name,
        file_path=file_path,
        chunk_size=chunk_size
    )

    return response

# Usage
upload_file_smart(client, "account", account_id, "new_largemedifile", "video.mp4")
```

### ファイルの一括 Upload
```python
from pathlib import Path
from PowerPlatform.Dataverse.core.errors import HttpError

def batch_upload_files(client, table_name, record_id, files_dict):
    """
    同一レコードの異なる column に複数ファイルを upload する。

    Args:
        table_name: テーブル名
        record_id: レコード ID
        files_dict: {"column_name": "file_path", ...}

    Returns:
        {"success": [...], "failed": [...]}
    """
    results = {"success": [], "failed": []}

    for column_name, file_path in files_dict.items():
        try:
            print(f"Uploading {Path(file_path).name} to {column_name}...")
            response = client.upload_file(
                table_name=table_name,
                record_id=record_id,
                file_column_name=column_name,
                file_path=file_path
            )
            results["success"].append({
                "column": column_name,
                "file": Path(file_path).name,
                "response": response
            })
            print(f"  ✓ Uploaded successfully")
        except HttpError as e:
            results["failed"].append({
                "column": column_name,
                "file": Path(file_path).name,
                "error": str(e)
            })
            print(f"  ❌ Upload failed: {e}")

    return results

# Usage
files = {
    "new_contractfile": "contract.pdf",
    "new_specfile": "specification.docx",
    "new_designfile": "design.png"
}
results = batch_upload_files(client, "account", account_id, files)
print(f"Success: {len(results['success'])}, Failed: {len(results['failed'])}")
```

### 失敗した Upload の再試行
```python
from pathlib import Path
import time
from PowerPlatform.Dataverse.core.errors import HttpError

def upload_with_retry(client, table_name, record_id, column_name, file_path, max_retries=3):
    """指数バックオフ retry ロジック付きで upload する。"""
    file_path = Path(file_path)

    for attempt in range(max_retries):
        try:
            print(f"Upload attempt {attempt + 1}/{max_retries}: {file_path.name}")
            response = client.upload_file(
                table_name=table_name,
                record_id=record_id,
                file_column_name=column_name,
                file_path=file_path,
                chunk_size=4 * 1024 * 1024
            )
            print(f"✓ Upload successful")
            return response
        except HttpError as e:
            if attempt == max_retries - 1:
                print(f"❌ Upload failed after {max_retries} attempts")
                raise

            # 指数バックオフ: 1s, 2s, 4s
            backoff_seconds = 2 ** attempt
            print(f"⚠ Upload failed. Retrying in {backoff_seconds}s...")
            time.sleep(backoff_seconds)

# Usage
upload_with_retry(client, "account", account_id, "new_documentfile", "contract.pdf")
```

---

## 3. 実務例

### 例 1: 顧客ドキュメント管理システム

```python
from pathlib import Path
from datetime import datetime
from enum import IntEnum
from PowerPlatform.Dataverse.client import DataverseClient
from azure.identity import ClientSecretCredential

class DocumentType(IntEnum):
    CONTRACT = 1
    INVOICE = 2
    SPECIFICATION = 3
    OTHER = 4

# Setup
credential = ClientSecretCredential(
    tenant_id="tenant-id",
    client_id="client-id",
    client_secret="client-secret"
)
client = DataverseClient("https://yourorg.crm.dynamics.com", credential)

def upload_customer_document(customer_id, doc_path, doc_type):
    """顧客向けドキュメントを upload する。"""
    doc_path = Path(doc_path)

    # ドキュメント レコード作成
    doc_record = {
        "new_documentname": doc_path.stem,
        "new_documenttype": doc_type,
        "new_customerid": customer_id,
        "new_uploadeddate": datetime.now().isoformat(),
        "new_filesize": doc_path.stat().st_size
    }

    doc_ids = client.create("new_customerdocument", doc_record)
    doc_id = doc_ids[0]

    # ファイル upload
    print(f"Uploading {doc_path.name}...")
    client.upload_file(
        table_name="new_customerdocument",
        record_id=doc_id,
        file_column_name="new_documentfile",
        file_path=doc_path
    )

    print(f"✓ Document uploaded and linked to customer")
    return doc_id

# Usage
customer_id = "customer-guid-here"
doc_id = upload_customer_document(
    customer_id,
    "contract.pdf",
    DocumentType.CONTRACT
)

# upload 済みドキュメントを照会
docs = client.get(
    "new_customerdocument",
    filter=f"new_customerid eq '{customer_id}'",
    select=["new_documentname", "new_documenttype", "new_uploadeddate"]
)

for page in docs:
    for doc in page:
        print(f"- {doc['new_documentname']} ({doc['new_uploadeddate']})")
```

### 例 2: サムネイル付きメディア ギャラリー

```python
from pathlib import Path
from enum import IntEnum
from PowerPlatform.Dataverse.client import DataverseClient

class MediaType(IntEnum):
    PHOTO = 1
    VIDEO = 2
    DOCUMENT = 3

def create_media_gallery(client, gallery_name, media_files):
    """
    複数ファイルを持つメディア ギャラリーを作成する。

    Args:
        gallery_name: ギャラリー名
        media_files: [{"file": path, "type": MediaType, "description": text}, ...]
    """
    # ギャラリー レコード作成
    gallery_ids = client.create("new_mediagallery", {
        "new_galleryname": gallery_name,
        "new_createddate": datetime.now().isoformat()
    })
    gallery_id = gallery_ids[0]

    # メディア項目を作成して upload
    for media_info in media_files:
        file_path = Path(media_info["file"])

        # メディア項目レコード作成
        item_ids = client.create("new_mediaitem", {
            "new_itemname": file_path.stem,
            "new_mediatype": media_info["type"],
            "new_description": media_info.get("description", ""),
            "new_galleryid": gallery_id,
            "new_filesize": file_path.stat().st_size
        })
        item_id = item_ids[0]

        # メディア ファイル upload
        print(f"Uploading {file_path.name}...")
        client.upload_file(
            table_name="new_mediaitem",
            record_id=item_id,
            file_column_name="new_mediafile",
            file_path=file_path
        )
        print(f"  ✓ {file_path.name}")

    return gallery_id

# Usage
media_files = [
    {"file": "photo1.jpg", "type": MediaType.PHOTO, "description": "Product shot 1"},
    {"file": "photo2.jpg", "type": MediaType.PHOTO, "description": "Product shot 2"},
    {"file": "demo.mp4", "type": MediaType.VIDEO, "description": "Product demo video"},
    {"file": "manual.pdf", "type": MediaType.DOCUMENT, "description": "User manual"}
]

gallery_id = create_media_gallery(client, "Q4 Product Launch", media_files)
print(f"Created gallery: {gallery_id}")
```

### 例 3: バックアップとアーカイブ システム

```python
from pathlib import Path
from datetime import datetime, timedelta
from PowerPlatform.Dataverse.client import DataverseClient
from PowerPlatform.Dataverse.core.errors import DataverseError
import json

def backup_table_data(client, table_name, output_dir):
    """
    テーブル データを JSON ファイルへバックアップし、アーカイブ レコードを作成する。
    """
    output_dir = Path(output_dir)
    output_dir.mkdir(exist_ok=True)

    backup_time = datetime.now()
    backup_file = output_dir / f"{table_name}_{backup_time.strftime('%Y%m%d_%H%M%S')}.json"

    print(f"Backing up {table_name}...")

    # 全レコード取得
    all_records = []
    for page in client.get(table_name, top=5000):
        all_records.extend(page)

    # JSON に書き出し
    with open(backup_file, 'w') as f:
        json.dump(all_records, f, indent=2, default=str)

    print(f"  ✓ Exported {len(all_records)} records")

    # Dataverse にバックアップ レコードを作成
    backup_ids = client.create("new_backuprecord", {
        "new_tablename": table_name,
        "new_recordcount": len(all_records),
        "new_backupdate": backup_time.isoformat(),
        "new_status": 1  # Completed
    })
    backup_id = backup_ids[0]

    # バックアップ ファイルを upload
    print(f"Uploading backup file...")
    client.upload_file(
        table_name="new_backuprecord",
        record_id=backup_id,
        file_column_name="new_backupfile",
        file_path=backup_file
    )

    return backup_id

# Usage
backup_id = backup_table_data(client, "account", "backups")
print(f"Backup created: {backup_id}")
```

### 例 4: レポート生成と保存の自動化

```python
from pathlib import Path
from datetime import datetime
from enum import IntEnum
from PowerPlatform.Dataverse.client import DataverseClient
import json

class ReportStatus(IntEnum):
    PENDING = 1
    PROCESSING = 2
    COMPLETED = 3
    FAILED = 4

def generate_and_store_report(client, report_type, data):
    """
    データからレポートを生成し Dataverse に保存する。
    """
    report_time = datetime.now()

    # レポート ファイルを生成 (疑似)
    report_file = Path(f"report_{report_type}_{report_time.strftime('%Y%m%d_%H%M%S')}.json")
    with open(report_file, 'w') as f:
        json.dump(data, f, indent=2)

    # レポート レコード作成
    report_ids = client.create("new_report", {
        "new_reportname": f"{report_type} Report",
        "new_reporttype": report_type,
        "new_generateddate": report_time.isoformat(),
        "new_status": ReportStatus.PROCESSING,
        "new_recordcount": len(data.get("records", []))
    })
    report_id = report_ids[0]

    try:
        # レポート ファイルを upload
        print(f"Uploading report: {report_file.name}")
        client.upload_file(
            table_name="new_report",
            record_id=report_id,
            file_column_name="new_reportfile",
            file_path=report_file
        )

        # status を completed に更新
        client.update("new_report", report_id, {
            "new_status": ReportStatus.COMPLETED
        })

        print(f"✓ Report stored successfully")
        return report_id

    except Exception as e:
        print(f"❌ Report generation failed: {e}")
        client.update("new_report", report_id, {
            "new_status": ReportStatus.FAILED,
            "new_errormessage": str(e)
        })
        raise
    finally:
        # 一時ファイルを cleanup
        report_file.unlink(missing_ok=True)

# Usage
sales_data = {
    "month": "January",
    "records": [
        {"product": "A", "sales": 10000},
        {"product": "B", "sales": 15000},
        {"product": "C", "sales": 8000}
    ]
}

report_id = generate_and_store_report(client, "SALES_SUMMARY", sales_data)
```

---

## 4. ファイル管理のベストプラクティス

### ファイルサイズ検証
```python
from pathlib import Path

def validate_file_for_upload(file_path, max_size_mb=500):
    """upload 前にファイルを検証する。"""
    file_path = Path(file_path)

    if not file_path.exists():
        raise FileNotFoundError(f"File not found: {file_path}")

    file_size = file_path.stat().st_size
    max_size_bytes = max_size_mb * 1024 * 1024

    if file_size > max_size_bytes:
        raise ValueError(f"File too large: {file_size / 1024 / 1024:.2f} MB > {max_size_mb} MB")

    return file_size

# Usage
try:
    size = validate_file_for_upload("document.pdf", max_size_mb=128)
    print(f"File valid: {size / 1024 / 1024:.2f} MB")
except (FileNotFoundError, ValueError) as e:
    print(f"Validation failed: {e}")
```

### 対応ファイル種別の検証
```python
from pathlib import Path

ALLOWED_EXTENSIONS = {'.pdf', '.docx', '.xlsx', '.jpg', '.png', '.mp4', '.zip'}

def validate_file_type(file_path):
    """拡張子を検証する。"""
    file_path = Path(file_path)

    if file_path.suffix.lower() not in ALLOWED_EXTENSIONS:
        raise ValueError(f"Unsupported file type: {file_path.suffix}")

    return True

# Usage
try:
    validate_file_type("document.pdf")
    print("File type valid")
except ValueError as e:
    print(f"Invalid: {e}")
```

### Upload Logging と Audit Trail
```python
from pathlib import Path
from datetime import datetime
import json

def log_file_upload(table_name, record_id, file_path, status, error=None):
    """監査証跡のためにファイル upload を記録する。"""
    file_path = Path(file_path)

    log_entry = {
        "timestamp": datetime.now().isoformat(),
        "table": table_name,
        "record_id": record_id,
        "file_name": file_path.name,
        "file_size": file_path.stat().st_size if file_path.exists() else 0,
        "status": status,
        "error": error
    }

    # log file に追記
    log_file = Path("upload_audit.log")
    with open(log_file, 'a') as f:
        f.write(json.dumps(log_entry) + "\n")

    return log_entry

# Usage in upload wrapper
def upload_with_logging(client, table_name, record_id, column_name, file_path):
    """監査 logging 付きで upload する。"""
    try:
        client.upload_file(
            table_name=table_name,
            record_id=record_id,
            file_column_name=column_name,
            file_path=file_path
        )
        log_file_upload(table_name, record_id, file_path, "SUCCESS")
    except Exception as e:
        log_file_upload(table_name, record_id, file_path, "FAILED", str(e))
        raise
```

---

## 5. ファイル操作のトラブルシューティング

### よくある問題と解決策

#### Issue: File Upload Timeout
```python
# 非常に大きなファイルでは、chunk size を戦略的に増やす
response = client.upload_file(
    table_name="account",
    record_id=record_id,
    file_column_name="new_file",
    file_path="large_file.zip",
    chunk_size=8 * 1024 * 1024  # 8 MB chunk
)
```

#### Issue: Insufficient Disk Space
```python
import shutil
from pathlib import Path

def check_upload_space(file_path):
    """ファイル + 一時バッファ分の空き容量があるか確認する。"""
    file_path = Path(file_path)
    file_size = file_path.stat().st_size

    # ディスク容量取得
    total, used, free = shutil.disk_usage(file_path.parent)

    # file_size + 10% バッファが必要
    required_space = file_size * 1.1

    if free < required_space:
        raise OSError(f"Insufficient disk space: {free / 1024 / 1024:.0f} MB free, {required_space / 1024 / 1024:.0f} MB needed")

    return True
```

#### Issue: File Corruption During Upload
```python
import hashlib

def verify_uploaded_file(local_path, remote_data):
    """upload 済みファイルの整合性を検証する。"""
    # ローカル hash を計算
    with open(local_path, 'rb') as f:
        local_hash = hashlib.sha256(f.read()).hexdigest()

    # metadata と比較
    remote_hash = remote_data.get("new_filehash")

    if local_hash != remote_hash:
        raise ValueError("File corruption detected: hash mismatch")

    return True
```

---

## Reference
- [Official File Upload Example](https://github.com/microsoft/PowerPlatform-DataverseClient-Python/blob/main/examples/advanced/file_upload.py)
- [File Upload Best Practices](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/file-column-data)
