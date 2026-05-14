# テストとコードの品質

## 目次
1. [conftest.py](#1-conftestpy)
2. [単体テスト](#2-単体テスト)
3. [バックエンド単体テスト](#3-backend-unit-tests)
4. [テストの実行](#4-running-tests)
5. [コード品質ツール](#5-code-quality-tools)
6. [プリコミットフック](#6-pre-commit-hooks)

---

## 1. `conftest.py`

`conftest.py` を使用して共有フィクスチャを定義します。フィクスチャに焦点を当て続けます。懸念ごとに 1 つのフィクスチャです。
非同期テストの場合は、`pyproject.toml` 内の `asyncio_mode = "auto"` とともに `pytest-asyncio` を使用します。「」パイソン
# テスト/conftest.py
pytestをインポートする
your_package.core から YourClient をインポート
your_package.backends.memory から MemoryBackend をインポート


@pytest.fixture
def Memory_backend() -> MemoryBackend:
    戻りメモリバックエンド()


@pytest.fixture
def client(memory_backend: MemoryBackend) -> YourClient:
    あなたのクライアントを返します(
        api_key="テストキー",
        バックエンド=メモリ_バックエンド、
    ）
「」---

## 2. 単体テスト

ハッピー パスとエッジ ケース (無効な入力、エラー状態など) の両方をテストします。「」パイソン
# テスト/test_core.py
pytestをインポートする
your_package から YourClient をインポート
your_package.Exceptions から YourPackageError をインポートします


def test_client_creates_with_valid_key():
    client = YourClient(api_key="sk-test")
    クライアントが None ではないことをアサートします


def test_client_raises_on_empty_key():
    pytest.raises(ValueError, match="api_key") を使用:
        あなたのクライアント(api_key="")


def test_client_raises_on_invalid_timeout():
    pytest.raises(ValueError, match="timeout") を使用:
        YourClient(api_key="sk-test"、タイムアウト=-1)


@pytest.mark.asyncio
async def test_process_returns_expected_result(クライアント: YourClient):
    result = await client.process({"input": "value"})
    結果で「出力」をアサート


@pytest.mark.asyncio
async def test_process_raises_on_invalid_input(クライアント: YourClient):
    pytest.raises(YourPackageError) を使用:
        await client.process({}) # 空の入力は失敗するはずです
「」---

## 3. バックエンド単体テスト

各バックエンドをライブラリの残りの部分から分離して独立してテストします。これが失敗を生む
診断が容易になり、抽象インターフェイスが実際に正しく実装されていることを確認できます。「」パイソン
# テスト/test_backends.py
pytestをインポートする
your_package.backends.memory から MemoryBackend をインポート


@pytest.mark.asyncio
非同期デフォルト test_set_and_get():
    バックエンド = MemoryBackend()
    await backend.set("key1", "value1")
    result = await backend.get("key1")
    アサート結果 == "値1"


@pytest.mark.asyncio
非同期デフォルト test_get_missing_key_returns_none():
    バックエンド = MemoryBackend()
    result = await backend.get("存在しない")
    アサート結果はNoneです


@pytest.mark.asyncio
非同期デフォルト test_delete_removes_key():
    バックエンド = MemoryBackend()
    await backend.set("key1", "value1")
    バックエンドを待つ.delete("key1")
    result = await backend.get("key1")
    アサート結果はNoneです


@pytest.mark.asyncio
非同期デフォルト test_ttl_expires_entry():
    非同期をインポートする
    バックエンド = MemoryBackend()
    await backend.set("key1", "value1", ttl=1)
    asyncio.sleep を待つ(1.1)
    result = await backend.get("key1")
    アサート結果はNoneです


@pytest.mark.asyncio
async def test_ Different_keys_are_independent():
    バックエンド = MemoryBackend()
    await backend.set("key1", "a")
    await backend.set("key2", "b")
    assert await backend.get("key1") == "a"
    assert await backend.get("key2") == "b"
    バックエンドを待つ.delete("key1")
    assert await backend.get("key2") == "b"
「」---

## 4. テストの実行「」バッシュ
pip install -e ".[dev]"
pytest # すべてのテスト
pytest --cov --cov-report=html # HTML カバレッジ レポートあり (ブラウザーで開きます)
pytest -k "test_middleware" # 名前でフィルターします
pytest -x # 最初の失敗時に停止
pytest -v # 詳細な出力
「」`pyproject.toml` のカバレッジ設定では、最小しきい値 (`fail_under = 80`) が強制されます。 CIは
それを下回ると失敗し、カバレッジ回帰が自動的に検出されます。

---

## 5. コード品質ツール

### Ruff (lint — flake8、pylint、その他多くのものを置き換えます)「」バッシュ
pip インストール ruff
ラフチェック。           # 問題がないか確認する
ラフチェック。 --fix # 安全な問題を自動修正する
「」Ruff は非常に高速で、Python リンティング エコシステムの大部分を置き換えます。で設定します
`pyproject.toml` — 完全な構成については、`references/pyproject-toml.md` を参照してください。

### 黒 (書式設定)「」バッシュ
ピップインストールブラック
黒。                # すべてのファイルをフォーマットします
黒。 --check # CI モード - ファイルを変更せずに問題を報告します
「」### isort (インポートソート)「」バッシュ
pip インストールイソルト
いろいろ。                # インポートをソートする
いろいろ。 --check-only # CI モード
「」常に `[tool.isort]` に `profile = "black"` を設定します。それ以外の場合は黒とアイソートが競合します。

### mypy (静的型チェック)「」バッシュ
pip インストール mypy
mypy your_package/ # パッケージソースのみを型チェックします
「」一般的な修正:

- `ignore_missing_imports = true` — 型指定されていないサードパーティのdepsを無視します
- `from __future__ import annotations` — PEP 563 の遅延評価を有効にします (Python 3.9 互換)
- `pip install types-redis` — Redis ライブラリのタイプ スタブ

### すべてを一度に実行する「」バッシュ
ラフチェック。 ＆＆ 黒 。 --check && isort 。 --check-only && mypy your_package/
「」---

## 6. 事前コミットフック

プリコミットでは、各コミットの前にすべての品質ツールが自動的に実行されるため、問題が CI に到達することはありません。
`pre-commit install` を使用してクローンごとに 1 回インストールします。```ヤムル
# .pre-commit-config.yaml
リポジトリ:
  - リポジトリ: https://github.com/astral-sh/ruff-pre-commit
    リビジョン: v0.4.4
    フック:
      - ID: ラフ
        引数: [--修正]
      - ID: ruff-format

  - リポジトリ: https://github.com/psf/black
    リビジョン: 24.4.2
    フック:
      - ID: 黒

  - リポジトリ: https://github.com/pycqa/isort
    リビジョン: 5.13.2
    フック:
      - ID: アイソート

  - リポジトリ: https://github.com/pre-commit/mirrors-mypy
    リビジョン: v1.10.0
    フック:
      - ID: mypy
        added_dependency: [types-redis] # 型指定された依存関係のスタブを追加します

  - リポジトリ: https://github.com/pre-commit/pre-commit-hooks
    リビジョン: v4.6.0
    フック:
      - ID: 末尾の空白
      - ID: ファイルの終わりの修正者
      - ID: check-yaml
      - ID: チェックトム
      - ID: チェックマージ競合
      - ID: デバッグステートメント
      - ID: ブランチへのコミットなし
        引数: [--ブランチ、マスター、--ブランチ、メイン]
「」

「」バッシュ
pip install プリコミット
pre-commit install # クローンごとに 1 回インストールします
pre-commit run --all-files # すべてのフックを手動で実行します (最初のインストール前に役立ちます)
「」`no-commit-to-branch` フックは、誤って `main`/`master` に直接コミットすることを防ぎます。
これにより、CI チェックがバイパスされます。常に機能ブランチで作業します。