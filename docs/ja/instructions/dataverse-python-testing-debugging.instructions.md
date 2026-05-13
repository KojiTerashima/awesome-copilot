---
applyTo: '**'
---

# Dataverse SDK for Python — テストとデバッグ戦略

公式の Azure Functions および pytest のテスト パターンに基づいています。

## 1. テスト概要

### Dataverse SDK のテスト ピラミッド

```
         Integration Tests  <- 実際の Dataverse でテスト
              /\
             /  \
            /Unit Tests (Mocked)\
           /____________________\
          < Framework Tests
```

---

## 2. Mocking を使った Unit Testing

### テスト環境のセットアップ

```bash
# テスト依存をインストール
pip install pytest pytest-cov unittest-mock
```

### DataverseClient を mock する

```python
# tests/test_operations.py
import pytest
from unittest.mock import Mock, patch, MagicMock
from PowerPlatform.Dataverse.client import DataverseClient

@pytest.fixture
def mock_client():
    """Mock 済み DataverseClient を提供する。"""
    client = Mock(spec=DataverseClient)
    return client

def test_create_account(mock_client):
    """Mock client を使った account 作成のテスト。"""

    # mock response を設定
    mock_client.create.return_value = ["id-123"]

    # 関数を呼び出す
    from my_app import create_account
    result = create_account(mock_client, {"name": "Acme"})

    # 検証
    assert result == "id-123"
    mock_client.create.assert_called_once_with("account", {"name": "Acme"})

def test_create_account_error(mock_client):
    """Account 作成時のエラー処理をテストする。"""
    from PowerPlatform.Dataverse.core.errors import DataverseError

    # error を投げるよう mock を設定
    mock_client.create.side_effect = DataverseError(
        message="Account exists",
        code="validation_error",
        status_code=400
    )

    # error が送出されることを確認
    from my_app import create_account
    with pytest.raises(DataverseError):
        create_account(mock_client, {"name": "Acme"})
```

### テスト用データ構造

```python
# tests/fixtures.py
import pytest

@pytest.fixture
def sample_account():
    """テスト用の sample account レコード。"""
    return {
        "accountid": "id-123",
        "name": "Acme Inc",
        "telephone1": "555-0100",
        "statecode": 0,
        "createdon": "2025-01-01T00:00:00Z"
    }

@pytest.fixture
def sample_accounts(sample_account):
    """複数の sample account。"""
    return [
        sample_account,
        {**sample_account, "accountid": "id-124", "name": "Fabrikam"},
        {**sample_account, "accountid": "id-125", "name": "Contoso"},
    ]

# Usage in tests
def test_process_accounts(mock_client, sample_accounts):
    mock_client.get.return_value = iter([sample_accounts])
    # 処理をテスト
```

---

## 3. よくある Mock パターン

### Pagination を伴う Get を mock する

```python
def test_pagination(mock_client, sample_accounts):
    """ページ分割された結果の扱いをテストする。"""

    # mock はページを持つ generator を返す
    mock_client.get.return_value = iter([
        sample_accounts[:2],  # Page 1
        sample_accounts[2:]   # Page 2
    ])

    from my_app import process_all_accounts
    result = process_all_accounts(mock_client)

    assert len(result) == 3  # すべてのページが処理された
```

### 一括操作を mock する

```python
def test_bulk_create(mock_client):
    """一括 account 作成をテストする。"""

    payloads = [
        {"name": "Account 1"},
        {"name": "Account 2"},
    ]

    # mock は ID のリストを返す
    mock_client.create.return_value = ["id-1", "id-2"]

    from my_app import create_accounts
    ids = create_accounts(mock_client, payloads)

    assert len(ids) == 2
    mock_client.create.assert_called_once_with("account", payloads)
```

### Error を mock する

```python
def test_rate_limiting_retry(mock_client):
    """rate limiting 時の retry ロジックをテストする。"""
    from PowerPlatform.Dataverse.core.errors import DataverseError

    # 失敗後に成功する mock
    error = DataverseError(
        message="Too many requests",
        code="http_error",
        status_code=429,
        is_transient=True
    )
    mock_client.create.side_effect = [error, ["id-123"]]

    from my_app import create_with_retry
    result = create_with_retry(mock_client, "account", {})

    assert result == "id-123"
    assert mock_client.create.call_count == 2  # retry した
```

---

## 4. Integration Testing

### ローカル開発テスト

```python
# tests/test_integration.py
import pytest
from azure.identity import InteractiveBrowserCredential
from PowerPlatform.Dataverse.client import DataverseClient

@pytest.fixture
def dataverse_client():
    """Integration test 用の実クライアント。"""
    client = DataverseClient(
        base_url="https://myorg-dev.crm.dynamics.com",
        credential=InteractiveBrowserCredential()
    )
    return client

@pytest.mark.integration
def test_create_and_retrieve_account(dataverse_client):
    """Account の作成と取得をテストする (実 Dataverse に対して)。"""

    # 作成
    account_id = dataverse_client.create("account", {
        "name": "Test Account"
    })[0]

    # 取得
    account = dataverse_client.get("account", account_id)

    # 検証
    assert account["name"] == "Test Account"

    # Cleanup
    dataverse_client.delete("account", account_id)
```

### テストの分離

```python
# tests/conftest.py
import pytest

@pytest.fixture(scope="function")
def test_account(dataverse_client):
    """テスト用 account を作成し、テスト後に cleanup する。"""

    account_id = dataverse_client.create("account", {
        "name": "Test Account"
    })[0]

    yield account_id

    # Cleanup
    try:
        dataverse_client.delete("account", account_id)
    except:
        pass  # すでに削除済み

# Usage
def test_update_account(dataverse_client, test_account):
    """Account 更新をテストする。"""
    dataverse_client.update("account", test_account, {"telephone1": "555-0100"})

    account = dataverse_client.get("account", test_account)
    assert account["telephone1"] == "555-0100"
```

---

## 5. Pytest 設定

### pytest.ini

```ini
[pytest]
# 既定では integration test をスキップ
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*

markers =
    integration: integration test としてマーク (実行: -m integration)
    slow: slow test としてマーク
    unit: unit test としてマーク
```

### テスト実行

```bash
# Unit test のみ
pytest

# Unit + integration
pytest -m "unit or integration"

# Integration のみ
pytest -m integration

# coverage 付き
pytest --cov=my_app tests/

# 特定 test
pytest tests/test_operations.py::test_create_account
```

---

## 6. Coverage 分析

### Coverage Report を生成

```bash
# coverage 付きでテスト実行
pytest --cov=my_app --cov-report=html tests/

# coverage を表示
open htmlcov/index.html  # macOS
start htmlcov/index.html  # Windows
```

### Coverage 設定 (.coveragerc)

```ini
[run]
branch = True
source = my_app

[report]
exclude_lines =
    pragma: no cover
    def __repr__
    raise AssertionError
    raise NotImplementedError
    if __name__ == .__main__.:

[html]
directory = htmlcov
```

---

## 7. print/logging によるデバッグ

### Debug Logging を有効化

```python
import logging
import sys

# logging を設定
logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.StreamHandler(sys.stdout),
        logging.FileHandler('debug.log')
    ]
)

# SDK logging を有効化
logging.getLogger('PowerPlatform').setLevel(logging.DEBUG)
logging.getLogger('azure').setLevel(logging.DEBUG)

# In test
def test_with_logging(mock_client):
    logger = logging.getLogger(__name__)
    logger.debug("Starting test")

    result = my_function(mock_client)

    logger.debug(f"Result: {result}")
```

### Pytest の出力キャプチャ

```bash
# テスト中の print/logging 出力を表示
pytest -s tests/

# failure 時のみ短い traceback を表示
pytest --tb=short tests/
```

---

## 8. Performance Testing

### 操作時間の計測

```python
import pytest
import time

def test_bulk_create_performance(dataverse_client):
    """一括 create の performance をテストする。"""

    payloads = [{"name": f"Account {i}"} for i in range(1000)]

    start = time.time()
    ids = dataverse_client.create("account", payloads)
    duration = time.time() - start

    assert len(ids) == 1000
    assert duration < 10  # 10 秒未満で完了すべき

    print(f"Created 1000 records in {duration:.2f}s ({1000/duration:.0f} records/s)")
```

### Pytest Benchmark Plugin

```bash
pip install pytest-benchmark
```

```python
def test_query_performance(benchmark, dataverse_client):
    """query performance を benchmark する。"""

    def get_accounts():
        return list(dataverse_client.get("account", top=100))

    result = benchmark(get_accounts)
    assert len(result) <= 100
```

---

## 9. よくあるテスト パターン

### Retry ロジックのテスト

```python
def test_retry_on_transient_error(mock_client):
    """一時的 error に対する retry をテストする。"""
    from PowerPlatform.Dataverse.core.errors import DataverseError

    error = DataverseError(
        message="Timeout",
        code="http_error",
        status_code=408,
        is_transient=True
    )

    # 失敗してから成功
    mock_client.create.side_effect = [error, ["id-123"]]

    from my_app import create_with_retry
    result = create_with_retry(mock_client, "account", {})

    assert result == "id-123"
```

### Filter Builder のテスト

```python
def test_filter_builder():
    """OData filter 生成をテストする。"""
    from my_app import build_account_filter

    # テスト ケース
    assert build_account_filter(status="active") == "statecode eq 0"
    assert build_account_filter(name="Acme") == "contains(name, 'Acme')"
    assert build_account_filter(status="active", name="Acme") \
        == "statecode eq 0 and contains(name, 'Acme')"
```

### エラー処理のテスト

```python
def test_handles_missing_record(mock_client):
    """404 error の処理をテストする。"""
    from PowerPlatform.Dataverse.core.errors import DataverseError

    mock_client.get.side_effect = DataverseError(
        message="Not found",
        code="http_error",
        status_code=404
    )

    from my_app import get_account_safe
    result = get_account_safe(mock_client, "invalid-id")

    assert result is None  # raise せず None を返す
```

---

## 10. デバッグ チェックリスト

| Issue | Debug Steps |
|-------|-------------|
| Test fails unexpectedly | `-s` フラグを付けて print 出力を見る |
| Mock not called | method 名と parameter が正確に一致しているか確認 |
| Real API failing | credential、URL、permission を確認 |
| Rate limiting in tests | delay を入れるか batch を小さくする |
| Data not found | レコードが作成されており cleanup されていないことを確認 |
| Assertion errors | actual 値と expected 値を出力 |

---

## 11. 関連資料

- [Pytest Documentation](https://docs.pytest.org/)
- [unittest.mock Reference](https://docs.python.org/3/library/unittest.mock.html)
- [Azure Functions Testing](https://learn.microsoft.com/en-us/azure/azure-functions/functions-reference-python#unit-testing)
- [Dataverse SDK Examples](https://github.com/microsoft/PowerPlatform-DataverseClient-Python/tree/main/examples)
