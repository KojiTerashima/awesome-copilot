# ライブラリコアパターン、OOP/SOLID、およびタイプヒント

## 目次
1. [OOP とソリッドの原則](#1-oop--solid-principles)
2. [タイプヒントのベストプラクティス](#2-type-hints-best-practices)
3. [コアクラス設計](#3-core-class-design)
4. [ファクトリー/ビルダーパターン](#4-factory--builder-pattern)
5. [構成パターン](#5-構成パターン)
6. [`__init__.py` — 明示的なパブリック API](#6-__init__py--explicit-public-api)
7. [オプションのバックエンド (プラグイン パターン)](#7-optional-backends-plugin-pattern)

---

## 1. OOP とソリッドの原則

これらの原則を適用して、保守可能、テスト可能、拡張可能なパッケージを作成します。
**過剰なエンジニアリングを行わないでください** — すべてではなく、実際の問題を解決する原則を適用します
すぐに。

### S — 単一責任の原則

各クラス/モジュールには、**変更する理由を 1 つ**持つ必要があります。「」パイソン
# 悪い例: 1 つのクラスがデータ、検証、および永続性を処理します
クラスユーザーマネージャー:
    def validate(self, user): ...
    def save_to_db(self, user): ...
    def send_email(self, user): ...

# 良い点: 責任を分担する
クラスUserValidator:
    def validate(self, user: User) -> なし: ...

クラス UserRepository:
    def save(self, user: User) -> なし: ...

クラスUserNotifier:
    def notification(self, user: User) -> なし: ...
「」### O — オープン/クローズの原則

拡張の場合は開き、変更の場合は閉じます。 **プロトコルまたは ABC** を拡張ポイントとして使用します。「」パイソン
from abc import ABC、abstractmethod

クラスStorageBackend(ABC):
    """インターフェイスを一度定義すれば、新しい実装のために決して変更しないでください。"""
    @abstractmethod
    def get(self, key: str) -> str |なし: ...
    @abstractmethod
    def set(self, key: str, value: str) -> なし: ...

class MemoryBackend(StorageBackend): # サブクラス化による拡張
    ...

class RedisBackend(StorageBackend): # StorageBackend を変更せずに新しい impl を追加します
    ...
「」### L — リスコフ置換原理

サブクラスはその基本クラスと置換可能でなければなりません。サブクラス内のコントラクトを決して狭めないでください。「」パイソン
クラスBaseProcessor:
    def process(self, data: dict) -> dict: ...

# BAD: 有効な辞書に対して TypeError を発生させます — 置換可能性を壊します
クラス StrictProcessor(BaseProcessor):
    def process(self, data: dict) -> dict:
        データでない場合:
            raise TypeError("Must have data") # Base はこれを決して発生させませんでした

# GOOD: ベースが受け入れるものを受け入れ、同じ契約を履行します
クラス StrictProcessor(BaseProcessor):
    def process(self, data: dict) -> dict:
        データでない場合:
            return {} # グレースフル — 同じ戻り値の型、新しい例外なし
「」### I — インターフェース分離の原則

大規模なモノリシック ABC よりも、**小規模で焦点を絞ったプロトコル** を優先します。「」パイソン
# 悪い例: すべての実装者に読み取り+書き込み+削除+リストの処理を強制する
クラス BigStorage(ABC):
    @abstractmethod
    def read(self): ...
    @abstractmethod
    def write(self): ...
    @abstractmethod
    def delete(self): ...
    @abstractmethod
    def list_all(self): ... # すべてのバックエンドがこれを必要とするわけではありません

# 良い点: 個別のプロトコル — クライアントは必要なものだけに依存します
import プロトコルの入力から

クラス読み取り可能(プロトコル):
    def read(self, key: str) -> str |なし: ...

クラス書き込み可能(プロトコル):
    def write(self, key: str, value: str) -> なし: ...

クラス削除可能(プロトコル):
    def delete(self, key: str) -> なし: ...
「」### D — 依存関係逆転の原則

高レベルのモジュールは、具体的な実装ではなく、**抽象化** (プロトコル/ABC) に依存します。
`__init__` (コンストラクター インジェクション) 経由で依存関係を渡します。「」パイソン
# 悪い点: 高レベルのクラスが独自の依存関係を作成する
クラスApiClient:
    def __init__(self) -> なし:
        self._cache = RedisCache() # Redis と密結合

# 良い: 抽象化に依存します。呼び出し現場にコンクリートを注入する
クラスApiClient:
    def __init__(self,cache: CacheBackend) -> なし: # CacheBackend はプロトコルです
        self._cache = キャッシュ

# ユーザーコード (またはテスト):
client = ApiClient(cache=RedisCache()) # 実数
client = ApiClient(cache=MemoryCache()) # テスト
「」### 継承よりも構成

深い継承チェーンよりも、含まれるオブジェクトへの委任を優先します。「」パイソン
# これを優先します (構成):
クラス YourClient:
    def __init__(self, バックエンド: StorageBackend, http: HttpTransport) -> なし:
        self._backend = バックエンド
        self._http = http

# これは避けてください (深い継承):
クラス YourClient(BaseClient, CacheMixin, RetryMixin, LoggingMixin):
    ... # 壊れやすい、テストが難しい、MRO の混乱
「」### 例外階層

パッケージには必ず基本例外を定義してください。その下にレイヤーの詳細があります。「」パイソン
# your_package/例外.py
クラス YourPackageError(Exception):
    """基本例外 — パッケージ エラーをキャッチするにはこれをキャッチします。"""

クラスConfigurationError(YourPackageError):
    """パッケージが正しく構成されていない場合に発生します。"""

クラス AuthenticationError(YourPackageError):
    """認証失敗時に発生します。"""

クラス RateLimitError(YourPackageError):
    """レート制限を超えると発生します。"""
    def __init__(self, retry_after: int) -> なし:
        self.retry_after = 再試行後
        super().__init__(f"レート制限あり。{retry_after}s 後に再試行してください。")
「」---

## 2. タイプヒントのベストプラクティス

PEP 484 (型ヒント)、PEP 526 (変数アノテーション)、PEP 544 (プロトコル) に従います。
PEP 561 (型付きパッケージ)。これらは高品質ライブラリにとってはオプションではありません。「」パイソン
from __future__ import annotations # PEP 563 の遅延評価を有効にする — 常にこれを追加します

# ARGUMENT の場合: 抽象型/プロトコル型を優先します (呼び出し元にとってより柔軟です)
from collections.abc import イテラブル、マッピング、シーケンス、呼び出し可能

def process_items(items: Iterable[str]) -> list[int]: ... # ✓ 任意の反復可能オブジェクトを受け入れます
def process_items(items: list[str]) -> list[int]: ... # ✗ 制限が多すぎる

# 戻り値の型の場合: 具象型を優先します (呼び出し元は何を取得するかを正確に知っています)
def get_names() -> list[str]: ... # ✓ 具体的
def get_names() -> Iterable[str]: ... # ✗ 呼び出し元はインデックスを作成できません

# X を使用する | Y 構文 (Python 3.10 以降)。Union[X, Y] または Optional[X] ではありません。
def find(キー: str) -> str |なし: ... # ✓ モダン
def find(key: str) -> Optional[str]: ... # ✗ 古いスタイル

# どれも共用体の最後にあってはなりません
def get(key: str) -> str |整数 |なし: ... # ✓

# どれも避ける — 型チェックを完全に無効にします
def process(data: Any) -> Any: ... # ✗ すべての安全性を失います
def process(data: dict[str, object]) -> dict[str, object]: # ✓

# パラメータが文字通り何でも受け入れる場合は、Any の代わりにオブジェクトを使用します
def log(値: オブジェクト) -> なし: ... # ✓

# Union 戻り値の型は避けてください。すべての呼び出しサイトで isinstance() チェックが必要です
def get_value() -> str | int: ... # ✗ 呼び出し元に分岐を強制します
「」### プロトコルと ABC「」パイソン
import Protocol、runtime_checkable の入力から
from abc import ABC、abstractmethod

# 実装クラスを制御しない場合はプロトコルを使用します (ダックタイピング)
@runtime_checkable # isinstance() チェックを実行時に機能させる
クラスシリアル化可能(プロトコル):
    def to_dict(self) -> dict[str, object]: ...

# クラス階層を制御し、デフォルトの実装が必要な場合は、ABC を使用します
クラスBaseBackend(ABC):
    @abstractmethod
    async def get(self, key: str) -> str |なし: ...

    def get_or_default(self, キー: str, デフォルト: str) -> str:
        結果 = self.get(キー)
        結果がデフォルトでない場合は結果を返します。それ以外の場合はデフォルトです。
「」### TypeVar とジェネリックス「」パイソン
import TypeVar、Generic の入力から

T = TypeVar("T")
T_co = TypeVar("T_co", covariant=True) # 読み取り専用コンテナの場合

クラス リポジトリ(ジェネリック[T]):
    """タイプセーフな汎用リポジトリ。"""
    def __init__(self, model_class: type[T]) -> なし:
        self._store: リスト[T] = []

    def add(self, item: T) -> なし:
        self._store.append(アイテム)

    def get_all(self) -> リスト[T]:
        戻りリスト(self._store)
「」### データコンテナのデータクラス「」パイソン
データクラスからインポートデータクラス、フィールド

@dataclass(frozen=True) #frozen=True → 不変、ハッシュ可能 (構成/キーに適しています)
クラス構成:
    API_キー: str
    タイムアウト: int = 30
    ヘッダー: dict[str, str] = フィールド(default_factory=dict)

    def __post_init__(self) -> なし:
        self.api_key でない場合:
            raise ValueError("api_key を空にすることはできません")
「」### TYPE_CHECKING ガード (循環インポートを避ける)「」パイソン
__future__ からアノテーションをインポート
import TYPE_CHECKING の入力から

TYPE_CHECKING の場合:
    from your_package.models import HeavyModel # 型チェック中にのみインポートされる

def プロセス(モデル: "HeavyModel") -> なし:
    ...
「」### 複数の署名のオーバーロード「」パイソン
import オーバーロードの入力によるもの

@オーバーロード
def get(キー: str、デフォルト: なし = ...) -> str |なし: ...
@オーバーロード
def get(キー: str, デフォルト: str) -> str: ...
def get(キー: str, デフォルト: str | なし = なし) -> str |なし:
    ... # 単一の実装で両方を処理します
「」---

## 3. コアクラスの設計

ライブラリのメインクラスには、明確で最小限の `__init__` と、すべてに対して適切なデフォルトが必要です。
パラメータを使用し、無効な入力に対して `TypeError` / `ValueError` を早期に発生させます。これにより混乱が防止されます
構築時ではなく呼び出し時にエラーが発生します。「」パイソン
# your_package/core.py
__future__ からアノテーションをインポート

your_package.Exceptions から YourPackageError をインポートします


クラス YourClient:
    「」
    <目的> の主要なエントリ ポイント。

    引数:
        api_key: 必要な認証資格情報。
        timeout: リクエストのタイムアウト (秒単位)。デフォルトは 30 です。
        retries: 再試行の回数。デフォルトは 3 です。

    発生するもの:
        ValueError: api_key が空であるか、タイムアウトが正でない場合。

    例:
        >>> your_package から YourClient をインポート
        >>> client = YourClient(api_key="sk-...")
        >>> 結果 = client.process(data)
    「」

    def __init__(
        自分自身、
        api_key: str、
        タイムアウト: int = 30、
        再試行: int = 3、
    ) -> なし:
        api_key でない場合:
            raise ValueError("api_key を空にすることはできません")
        タイムアウト <= 0 の場合:
            raise ValueError("タイムアウトは正の値でなければなりません")
        self._api_key = api_key
        self.timeout = タイムアウト
        self.retries = 再試行

    def process(self, data: dict) -> dict:
        「」
        データを処理して結果を返します。

        引数:
            data: 処理する入力辞書。

        戻り値:
            辞書として処理した結果。

        発生するもの:
            YourPackageError: 処理が失敗した場合。
        「」
        ...
「」### デザインルール

- `__init__` 内のすべての設定を受け入れ、メソッド呼び出しに分散されません。
- 構築時に検証 - 明確なメッセージで迅速に失敗します。
- `__init__` 署名を安定した状態に保ちます。新しい **キーワードのみ** 引数をデフォルトで追加すると逆方向になります
  互換性があります。位置引数の削除または並べ替えは破壊的な変更です。

---

## 4. ファクトリー/ビルダーパターン

ユーザーが事前構成されたインスタンスを作成する必要がある場合は、ファクトリー関数を使用します。これにより混乱が避けられます
`__init__` は 12 個のキーワード引数を持ち、一般的なケースを単純にしています。「」パイソン
# your_package/factory.py
__future__ からアノテーションをインポート

your_package.core から YourClient をインポート
your_package.backends.memory から MemoryBackend をインポート


def create_client(
    api_key: str、
    *、
    タイムアウト: int = 30、
    再試行: int = 3、
    バックエンド: str = "メモリ",
    バックエンド URL: str |なし = なし、
) -> あなたのクライアント:
    「」
    構成された YourClient を返すファクトリ。

    引数:
        api_key: 必須の API キー。
        timeout: リクエストのタイムアウト (秒単位)。
        retries: 再試行の回数。
        バックエンド: ストレージ バックエンド タイプ。 「メモリ」または「Redis」のいずれか。
        backend_url: 選択したバックエンドの接続 URL。

    例:
        >>> client = create_client(api_key="sk-...", backend="redis", backend_url="redis://localhost")
    「」
    バックエンド == "redis" の場合:
        your_package.backends.redis から RedisBackend をインポート
        _backend = RedisBackend(url=backend_url または "redis://localhost:6379")
    それ以外の場合:
        _backend = メモリバックエンド()

    return YourClient(api_key=api_key、timeout=タイムアウト、retries=再試行、backend=_backend)
「」**なぜクラス メソッドではなくファクトリーなのでしょうか?** どちらも機能します。スタンドアロンのファクトリ関数のほうが簡単です
テストでモックを作成し、ファクトリ ロジックをクラス自体に結合することを回避します。

---

## 5. 構成パターン

データクラス (または Pydantic `BaseModel`) を使用して構成を保持します。これにより無料の検証が可能になります。
役立つエラー メッセージと、すべてのオプションを文書化する 1 つの場所。「」パイソン
# your_package/config.py
__future__ からアノテーションをインポート
データクラスからインポートデータクラス、フィールド


@データクラス
クラス YourSettings:
    「」
    クライアントの構成。

    属性:
        timeout: HTTP タイムアウト (秒単位)。
        retries: 一時的なエラーに対する再試行の回数。
        base_url: ベース API URL。
    「」
    タイムアウト: int = 30
    再試行: int = 3
    Base_url: str = "https://api.example.com"
    extra_headers: dict[str, str] = フィールド(default_factory=dict)

    def __post_init__(self) -> なし:
        self.timeout <= 0 の場合:
            raise ValueError("タイムアウトは正の値でなければなりません")
        self.retries < 0 の場合:
            raise ValueError("再試行は負でない必要があります")
「」環境変数のロードが必要な場合は、**オプション**の依存関係として `pydantic-settings` を使用します。
必須のdepとしてではなく、`[project.optional-dependencies]`で宣言してください。

---

## 6. `__init__.py` — 明示的なパブリック API

明確に定義された `__all__` は単なるスタイルではなく、ユーザー (および IDE) にそのコードの一部を正確に伝えます。
パブリック API を使用し、コントラクトの一部として内部ヘルパーの誤ったインポートを防ぎます。「」パイソン
# your_package/__init__.py
"""あなたのパッケージ: <1 行の説明>。"""

importlib.metadata インポート バージョンから、PackageNotFoundError

試してみてください:
    __version__ = version("あなたのパッケージ")
PackageNotFoundError を除く:
    __version__ = "0.0.0-dev"

your_package.core から YourClient をインポート
your_package.config から YourSettings をインポートする
your_package.Exceptions から YourPackageError をインポートします

__all__ = [
    "あなたのクライアント"、
    "あなたの設定",
    "あなたのパッケージエラー",
    "__バージョン__",
】
「」ルール:
- ユーザーが使用することになっているもののみをエクスポートします。内部ヘルパーは `_utils.py` またはサブモジュールに組み込まれます。
- `__init__.py` の最上位レベルでのインポートを浅く保つ — 重いオプションの dep のインポートを避ける
  (`redis` など) モジュール レベルで。それらを必要とするクラスまたは関数内に遅延インポートします。
- `__version__` は常にパブリック API の一部です。これにより `your_package.__version__` が有効になります。
  デバッグ中。

---

## 7. オプションのバックエンド (プラグイン パターン)

このパターンにより、パッケージはすぐに使用できる (追加の deps なし) インメモリ バックエンドで動作します。
上級ユーザーが Redis、データベース、または任意のカスタム ストレージをプラグインできるようにします。

### 5.1 抽象基本クラス — インターフェイスを定義します「」パイソン
# your_package/backends/__init__.py
from abc import ABC、abstractmethod


クラスBaseBackend(ABC):
    """抽象ストレージ バックエンド インターフェイス。

    これを実装してカスタム バックエンド (データベース、キャッシュなど) を追加します。
    「」

    @abstractmethod
    async def get(self, key: str) -> str |なし:
        """キーによって値を取得します。見つからない場合は None を返します。"""
        ...

    @abstractmethod
    async def set(self, key: str, value: str, ttl: int | None = None) -> なし:
        """値を保存します。秒単位のオプションの TTL。"""
        ...

    @abstractmethod
    async def delete(self, key: str) -> なし:
        """キーを削除します。"""
        ...
「」### 5.2 メモリ バックエンド — 追加の DEPS は不要「」パイソン
# your_package/backends/memory.py
__future__ からアノテーションをインポート

非同期をインポートする
インポート時間
your_package.backends から BaseBackend をインポート


クラスMemoryBackend(BaseBackend):
    """スレッドセーフなメモリ内バックエンド。追加の依存関係なしですぐに動作します。"""

    def __init__(self) -> なし:
        self._store: dict[str, tuple[str, float |なし]] = {}
        self._lock = asyncio.Lock()

    async def get(self, key: str) -> str |なし:
        self._lock と非同期:
            エントリ = self._store.get(キー)
            エントリが「なし」の場合:
                なしを返す
            値、expires_at = エントリ
            expires_at が None ではなく、time.time() >expires_at の場合:
                del self._store[キー]
                なしを返す
            戻り値

    async def set(self, key: str, value: str, ttl: int | None = None) -> なし:
        self._lock と非同期:
            expires_at = time.time() + ttl でない場合は ttl それ以外はなし
            self._store[key] = (値、expires_at)

    async def delete(self, key: str) -> なし:
        self._lock と非同期:
            self._store.pop(キー、なし)
「」### 5.3 Redis バックエンド — インストールされていない場合は明らかな ImportError が発生します

重要な設計: モジュール レベルではなく、`__init__` 内で `redis` を遅延的にインポートします。このようにして、
`redis` がインストールされていない場合でも、`import your_package` は失敗しません。「」パイソン
# your_package/backends/redis.py
__future__ からアノテーションをインポート
your_package.backends から BaseBackend をインポート

試してみてください:
    redis.asyncio を aioredis としてインポート
ImportError を exc として除く:
    インポートエラーを発生させる(
        「Redis バックエンドには追加の Redis が必要です:\n」
        「 pip install your-package[redis]」
    ）excから


クラスRedisBackend(BaseBackend):
    """分散/マルチプロセス展開用の Redis ベースのストレージ。"""

    def __init__(self, url: str = "redis://localhost:6379") -> なし:
        self._client = aioredis.from_url(url, decode_responses=True)

    async def get(self, key: str) -> str |なし:
        return await self._client.get(key)

    async def set(self, key: str, value: str, ttl: int | None = None) -> なし:
        await self._client.set(key, value, ex=ttl)

    async def delete(self, key: str) -> なし:
        self._client.delete(key) を待ちます
「」### 5.4 ユーザーがバックエンドを選択する方法「」パイソン
# デフォルト: メモリ内、追加の deps は必要ありません
your_package から YourClient をインポート
client = YourClient(api_key="sk-...")

# Redis: pip install your-package[redis]
your_package.backends.redis から RedisBackend をインポート
client = YourClient(api_key="sk-...", backend=RedisBackend(url="redis://localhost:6379"))
「」
