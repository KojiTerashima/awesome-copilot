# アーキテクチャ パターン — バックエンド システム、構成、トランスポート、CLI

## 目次
1. [バックエンド システム (プラグイン/戦略パターン)](#1-backend-system-pluginstrategy-pattern)
2. [構成レイヤー (設定データクラス)](#2-config-layer-settings-dataclass)
3. [トランスポート層 (HTTP クライアント抽象化)](#3-transport-layer-http-client-abstraction)
4. [CLI サポート](#4-cli-support)
5. [コアクライアントでのバックエンドインジェクション](#5-backend-injection-in-core-client)
6. [決定ルール](#6-決定ルール)

---

## 1. バックエンド システム (プラグイン/戦略パターン)

明確な基本プロトコル、依存関係のないデフォルトを使用して `backends/` サブパッケージを構造化します。
実装、および追加の背後にあるオプションの重い実装。

### ディレクトリのレイアウト「」
あなたのパッケージ/
  バックエンド/
    __init__.py # BaseBackend + ファクトリをエクスポートします。プロトコル/ABCを保持します
    base.py # 抽象基本クラス (ABC) またはプロトコル定義
    Memory.py # デフォルトの依存性ゼロのメモリ内実装
    redis.py # オプションのより重い実装 (追加機能によって保護されています)
「」### `backends/base.py` — 抽象インターフェイス「」パイソン
# your_package/backends/base.py
__future__ からアノテーションをインポート

from abc import ABC、abstractmethod


クラスBaseBackend(ABC):
    """抽象的なストレージ/処理バックエンド。

    すべての具体的なバックエンドはこれらのメソッドを実装する必要があります。
    モジュール レベルで重い依存関係をインポートしないでください。依存関係はクラス内で保護してください。
    「」

    @abstractmethod
    def get(self, key: str) -> str |なし:
        """キーによって値を取得します。キーが存在しない場合は None を返します。"""
        ...

    @abstractmethod
    def set(self, key: str, value: str, ttl: int | None = None) -> なし:
        """オプションの TTL (秒) を使用して値を保存します。"""
        ...

    @abstractmethod
    def delete(self, key: str) -> なし:
        """キーを削除します。キーが存在しない場合は何もしません。"""
        ...

    def close(self) -> なし: # noqa: B027 (意図的に非抽象化)
        """オプションのクリーンアップ フック。接続を保持するバックエンドでオーバーライドします。"""
「」### `backends/memory.py` — デフォルトの Zero-Dep 実装「」パイソン
# your_package/backends/memory.py
__future__ からアノテーションをインポート

インポート時間
from collections.abc import イテレータ
contextlibからcontextmanagerをインポートします
スレッドインポートロックから

.base インポート BaseBackend から


クラスMemoryBackend(BaseBackend):
    """スレッドセーフなメモリ内バックエンド。外部依存関係は必要ありません。"""

    def __init__(self) -> なし:
        self._store: dict[str, tuple[str, float |なし]] = {}
        self._lock = ロック()

    def get(self, key: str) -> str |なし:
        self._lock を使用:
            エントリ = self._store.get(キー)
            エントリが「なし」の場合:
                なしを返す
            値、expires_at = エントリ
            expires_at が None ではなく、time.monotonic() >expires_at の場合:
                del self._store[キー]
                なしを返す
            戻り値

    def set(self, key: str, value: str, ttl: int | None = None) -> なし:
        expires_at = time.monotonic() + ttl でない場合は ttl それ以外は なし
        self._lock を使用:
            self._store[key] = (値、expires_at)

    def delete(self, key: str) -> なし:
        self._lock を使用:
            self._store.pop(キー、なし)
「」### `backends/redis.py` — オプションの強力な実装「」パイソン
# your_package/backends/redis.py
__future__ からアノテーションをインポート

.base インポート BaseBackend から


クラスRedisBackend(BaseBackend):
    """Redis による実装。必要なもの: pip install your-package[redis]"""

    def __init__(self, url: str = "redis://localhost:6379/0") -> なし:
        試してみてください:
            Redis を _redis としてインポートします
        ImportError を exc として除く:
            インポートエラーを発生させる(
                「RedisBackend には redis が必要です。」
                「次のようにインストールします: pip install your-package[redis]」
            ）excから
        self._client = _redis.from_url(url, decode_responses=True)

    def get(self, key: str) -> str |なし:
        return self._client.get(key) # type:ignore[戻り値]

    def set(self, key: str, value: str, ttl: int | None = None) -> なし:
        ttl が None でない場合:
            self._client.setex(キー、ttl、値)
        それ以外の場合:
            self._client.set(キー, 値)

    def delete(self, key: str) -> なし:
        self._client.delete(キー)

    def close(self) -> なし:
        self._client.close()
「」### `backends/__init__.py` — パブリック API + ファクトリ「」パイソン
# your_package/backends/__init__.py
__future__ からアノテーションをインポート

.base インポート BaseBackend から
.memoryインポートからMemoryBackend

__all__ = ["BaseBackend", "MemoryBackend", "get_backend"]


def get_backend(backend_type: str = "メモリ", **kwargs: object) -> BaseBackend:
    """ファクトリ: 要求されたバックエンド インスタンスを返します。

    引数:
        backend_type: "memory" (デフォルト) または "redis"。
        **kwargs: バックエンド コンストラクターに転送されます。
    「」
    backend_type == "メモリ"の場合:
        戻りメモリバックエンド()
    backend_type == "redis"の場合:
        from .redis import RedisBackend # 遅いインポート — redis はオプションです
        return RedisBackend(**kwargs) # type:ignore[arg-type]
    raise ValueError(f"不明なバックエンド タイプ: {backend_type!r}")
「」---

## 2. 構成レイヤー (設定データクラス)

すべての設定を 1 つの `config.py` モジュールに集中させます。魔法値の分散を避け、
`os.environ` はコードベース全体で呼び出します。

### `config.py`「」パイソン
# your_package/config.py
__future__ からアノテーションをインポート

OSをインポートする
データクラスからインポートデータクラス、フィールド


@データクラス
クラス設定:
    """パッケージのすべてのランタイム構成。

    属性:
        api_key: 認証資格情報。これを決して記録したり公開したりしないでください。
        timeout: HTTP リクエストのタイムアウト (秒単位)。
        retries: 一時的な失敗に対する再試行の最大数。
        base_url: API ベース URL。ローカルサーバーを使用したテストでオーバーライドします。
    「」

    API_キー: str
    タイムアウト: int = 30
    再試行: int = 3
    Base_url: str = "https://api.example.com/v1"

    def __post_init__(self) -> なし:
        self.api_key でない場合:
            raise ValueError("api_key を空にすることはできません")
        self.timeout < 1 の場合:
            raise ValueError("タイムアウトは 1 以上である必要があります")
        self.retries < 0 の場合:
            raise ValueError("再試行は 0 以上である必要があります")

    @クラスメソッド
    def from_env(cls) -> 「設定」:
        """環境変数から設定を構築します。

        必要な環境変数: YOUR_PACKAGE_API_KEY
        オプションの環境変数: YOUR_PACKAGE_TIMEOUT、YOUR_PACKAGE_RETRIES
        「」
        api_key = os.environ.get("YOUR_PACKAGE_API_KEY", "")
        タイムアウト = int(os.environ.get("YOUR_PACKAGE_TIMEOUT", "30"))
        retries = int(os.environ.get("YOUR_PACKAGE_RETRIES", "3"))
        return cls(api_key=api_key、timeout=タイムアウト、retries=再試行)
「」### Pydantic の使用 (オプション、大規模プロジェクトの場合)「」パイソン
# your_package/config.py — Pydantic v2 バリアント
__future__ からアノテーションをインポート

pydanticインポートフィールドから
pydantic_settings から BaseSettings をインポート


クラス設定(BaseSettings):
    api_key: str = フィールド(..., min_length=1)
    タイムアウト: int = フィールド(30, ge=1)
    再試行: int = フィールド(3, ge=0)
    Base_url: str = "https://api.example.com/v1"

    model_config = {"env_prefix": "YOUR_PACKAGE_"}
「」---

## 3. トランスポート層 (HTTP クライアントの抽象化)

HTTP に関するすべての懸念事項 (ヘッダー、再試行、タイムアウト、エラー解析) を専用のツールで分離します。
`transport/` サブパッケージ。コア クライアントは、`httpx` ではなく、トランスポート抽象化に依存します。
または `requests` を直接実行します。

### ディレクトリのレイアウト「」
あなたのパッケージ/
  輸送/
    __init__.py # HttpTransport を再エクスポートする
    http.py # 具体的な httpx ベースのトランスポート
「」### `transport/http.py`「」パイソン
# your_package/transport/http.py
__future__ からアノテーションをインポート

import Any と入力してから

httpx をインポートする

..config インポート設定から
from ..Exceptions import YourPackageError、RateLimitError、AuthenticationError


クラスHttpTransport：
    """認証、再試行、エラー マッピングを一元化する薄い httpx ラッパー。"""

    def __init__(self, settings: 設定) -> なし:
        self._settings = 設定
        self._client = httpx.Client(
            Base_url=設定.base_url,
            タイムアウト=設定.タイムアウト、
            headers={"認可": f"ベアラー {settings.api_key}"},
        ）

    デフォルトリクエスト(
        自分自身、
        メソッド: str、
        パス: str、
        *、
        json: dict[str, Any] |なし = なし、
        パラメータ: dict[str, Any] |なし = なし、
    ) -> dict[str, Any]:
        """HTTP リクエストを送信し、解析された JSON 本文を返します。

        発生するもの:
            認証エラー: 401。
            RateLimitError: 429 上。
            YourPackageError: 他のすべての 2xx 以外の応答。
        「」
        応答 = self._client.request(メソッド、パス、json=json、params=params)
        self._raise_for_status(応答)
        応答.json() を返す

    def _raise_for_status(self, 応答: httpx.Response) -> なし:
        応答ステータスコード == 401 の場合:
            raise AuthenticationError("API キーが無効か期限切れです。")
        応答ステータスコード == 429 の場合:
            raise RateLimitError("レート制限を超えました。バックオフして再試行してください。")
        応答がエラーの場合:
            YourPackageError(
                f「API エラー {response.status_code}: {response.text[:200]}」
            ）

    def close(self) -> なし:
        self._client.close()

    def __enter__(self) -> "HttpTransport":
        自分を返す

    def __exit__(self, *args: object) -> なし:
        self.close()
「」### 非同期バリアント「」パイソン
# your_package/transport/async_http.py
__future__ からアノテーションをインポート

import Any と入力してから

httpx をインポートする

..config インポート設定から
from ..Exceptions import YourPackageError、RateLimitError、AuthenticationError


クラスAsyncHttpTransport:
    """非同期 httpx ラッパー。`async with AsyncHttpTransport(...) as t:` とともに使用します。"""

    def __init__(self, settings: 設定) -> なし:
        self._settings = 設定
        self._client = httpx.AsyncClient(
            Base_url=設定.base_url,
            タイムアウト=設定.タイムアウト、
            headers={"認可": f"ベアラー {settings.api_key}"},
        ）

    非同期定義リクエスト(
        自分自身、
        メソッド: str、
        パス: str、
        *、
        json: dict[str, Any] |なし = なし、
        パラメータ: dict[str, Any] |なし = なし、
    ) -> dict[str, Any]:
        response = await self._client.request(メソッド, パス, json=json, params=params)
        self._raise_for_status(応答)
        応答.json() を返す

    def _raise_for_status(self, 応答: httpx.Response) -> なし:
        応答ステータスコード == 401 の場合:
            raise AuthenticationError("API キーが無効か期限切れです。")
        応答ステータスコード == 429 の場合:
            raise RateLimitError("レート制限を超えました。バックオフして再試行してください。")
        応答がエラーの場合:
            YourPackageError(
                f「API エラー {response.status_code}: {response.text[:200]}」
            ）

    async def aclose(self) -> なし:
        self._client.aclose() を待つ

    async def __aenter__(self) -> "AsyncHttpTransport":
        自分を返す

    async def __aexit__(self, *args: object) -> なし:
        self.aclose() を待つ
「」---

## 4. CLI のサポート

`pyproject.toml` の `[project.scripts]` を介して CLI エントリ ポイントを追加します。

### `pyproject.toml` エントリ```トムル
[プロジェクト.スクリプト]
your-cli = "your_package.cli:main"
「」インストール後、ユーザーは端末から直接 `your-cli --help` を実行できます。

### `cli.py` — クリックの使用「」パイソン
# your_package/cli.py
__future__ からアノテーションをインポート

インポートシステム

インポートをクリック

.configインポート設定から
.core インポート YourClient から


@click.group()
@click.version_option()
def main() -> なし:
    """パッケージ CLI — コマンド ラインから API と対話します。"""


@main.command()
@click.option("--api-key", envvar="YOUR_PACKAGE_API_KEY", required=True, help="API キー。")
@click.option("--timeout"、default=30、show_default=True、help="リクエストタイムアウト(秒)。")
@click.argument("クエリ")
def search(api_key: str、タイムアウト: int、クエリ: str) -> なし:
    """API を検索し、結果を出力します。"""
    設定 = 設定(api_key=api_key, timeout=タイムアウト)
    client = YourClient(設定=設定)
    試してみてください:
        結果 = client.search(クエリ)
        結果内のアイテムの場合:
            click.echo(アイテム)
    exc としての例外を除く:
        click.echo(f"エラー: {exc}", err=True)
        sys.exit(1)
「」### `cli.py` — Typer の使用 (最新の代替手段)「」パイソン
# your_package/cli.py
__future__ からアノテーションをインポート

インポートタイパー

.configインポート設定から
.core インポート YourClient から

app = typer.Typer(help="あなたのパッケージ CLI.")


@app.command()
デフォルト検索(
    クエリ: str = typer.Argument(..., help="検索クエリ。"),
    api_key: str = typer.Option(..., envvar="YOUR_PACKAGE_API_KEY"),
    タイムアウト: int = typer.Option(30, help="リクエストのタイムアウト (秒)。"),
) -> なし:
    """API を検索し、結果を出力します。"""
    設定 = 設定(api_key=api_key, timeout=タイムアウト)
    client = YourClient(設定=設定)
    結果 = client.search(クエリ)
    結果内のアイテムの場合:
        typer.echo(アイテム)


def main() -> なし:
    アプリ()
「」---

## 5. コアクライアントでのバックエンドインジェクション

**重要:** は常に `backend` をコンストラクター引数として受け入れます。バックエンドをインスタンス化しないでください
コンストラクター内でフォールバック パラメーターを使用しないと、テストが不可能になります。「」パイソン
# your_package/core.py
__future__ からアノテーションをインポート

.backends.base から BaseBackend をインポート
from .backends.memory import MemoryBackend
.configインポート設定から


クラス YourClient:
    """プライマリ クライアント。テストを容易にするために、挿入されたバックエンドを受け入れます。

    引数:
        設定: 解決された構成。実稼働環境には、Settings.from_env() を使用します。
        バックエンド: ストレージ/処理バックエンド。 None の場合、デフォルトは MemoryBackend です。
        タイムアウト: 非推奨 — 代わりに設定オブジェクトを渡します。
        再試行: 非推奨 — 代わりに設定オブジェクトを渡します。
    「」

    def __init__(
        自分自身、
        API キー: str |なし = なし、
        *、
        設定: 設定 |なし = なし、
        バックエンド: BaseBackend |なし = なし、
        タイムアウト: int = 30、
        再試行: int = 3、
    ) -> なし:
        設定が「なし」の場合:
            api_key が None の場合:
                raise ValueError("「api_key」または「settings」のいずれかを指定してください。")
            settings = 設定(api_key=api_key、timeout=タイムアウト、retries=再試行)
        self._settings = 設定
        # CORRECT — ハードコードではなく、デフォルトで挿入されます
        self.backend: BaseBackend = バックエンドが None でない場合はバックエンド、それ以外の場合は MemoryBackend()

    # ...メソッド
「」### アンチパターン — 絶対にやってはいけないこと「」パイソン
# BAD: バックエンドをハードコーディングします。テスト中に交換することは不可能
クラス YourClient:
    def __init__(self, api_key: str) -> なし:
        self.backend = MemoryBackend() # ← インジェクションは不可能

# BAD: インポート内のパッケージ名リテラルをハードコードします。
from your_package.backends.memory import MemoryBackend # your_package 自体でのみ問題ありません
# パッケージ内で相対インポートを使用します。
from .backends.memory import MemoryBackend # ← 正しい
「」---

## 6. 決定ルール「」
パッケージは外部状態 (キャッシュ、DB、キュー) と対話しますか?
§── YES → バックエンドを追加/ BaseBackend + MemoryBackend で
│ extras_require の後ろにオプションの重いバックエンドを追加します
│
└── いいえ → バックエンド/完全にスキップします。 core.py をシンプルにする

パッケージは外部 HTTP API を呼び出しますか?
§── はい → Transport/http.py を追加します。設定経由で挿入
│
└── NO → 輸送をスキップ/

パッケージにはコマンドライン インターフェイスが必要ですか?
§── はい、簡単 (1 ～ 3 コマンド) → argparse を使用するか、
│ pyproject.tomlに[project.scripts]を追加
│
§── はい、複雑（サブコマンド、プラグイン） → クリックまたはタイパーを使用
│
└── いいえ → cli.py をスキップ

実行時の動作はユーザー指定の構成に依存しますか?
§── YES → 設定データクラスを使用して config.py を追加
│ 本番環境での使用のために、Settings.from_env() を公開する
│
└── NO → コンストラクタ内でパラメータを直接受け入れる
「」
