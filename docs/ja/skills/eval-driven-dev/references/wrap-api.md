# Wrap API Reference

> pixie source code の docstring から自動生成されています。
> 手で編集せず、upstream の [pixie-qa](https://github.com/yiouli/pixie-qa) source repository から再生成してください。

`pixie.wrap` — data-oriented observation API。

`wrap()` は、処理パイプライン内の名前付き地点で data value または callable を観測します。挙動は active mode に依存します。

- **No-op** (tracing 無効、eval registry なし): `data` を変更せずそのまま返す
- **Tracing** (`pixie trace` 実行中): trace file へ書き出し、OTel event を発行する (span が active なら span event、そうでなければ OTel logger) うえで、`data` を変更せず返す (callable の場合は呼び出し時に event を発火する wrapper を返す)
- **Eval** (eval registry active): `purpose="input"` では dependency data を注入し、`purpose="output"` / `purpose="state"` では output/state を捕捉する

---

## CLI Commands

| Command                                                                                   | Description                                                                                                                                   |
| ----------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| `pixie trace --runnable <filepath:ClassName> --input <kwargs.json> --output <file.jsonl>` | Runnable を 1 回実行し、JSON file の kwargs を使って trace file を書き出す。`--input` は **file path** であり inline JSON ではない。      |
| `pixie format <file.jsonl>`                                                               | trace file を整形済み dataset entry template に変換する。`entry_kwargs`、`eval_input`、`eval_output` (本物の捕捉 output) を表示する。      |
| `pixie trace filter <file.jsonl> --purpose input`                                         | 指定 purpose に一致する wrap event だけを表示する。該当 event ごとに 1 行の JSON を出力する。                                               |

---

## Classes

### `pixie.Runnable`

```python
class pixie.Runnable(Protocol[T]):
    @classmethod
    def create(cls) -> Runnable[Any]: ...
    async def setup(self) -> None: ...
    async def run(self, args: T) -> None: ...
    async def teardown(self) -> None: ...
```

dataset runner が使う structured runnable の protocol。`T` は `pydantic.BaseModel` subclass であり、その field は dataset JSON の `entry_kwargs` key と一致していなければなりません。

Lifecycle:

1. `create()` — runnable instance を構築して返す class method。
2. `setup()` — **async**。最初の `run()` 前に **1 回だけ** 呼ばれる。共有 resource (例: `TestClient`、database connection) をここで初期化する。任意実装で、既定は no-op。
3. `run(args)` — **async**。**各 dataset entry に対して並列に** 呼ばれる (最大 4 並列)。`args` は `entry_kwargs` から作られた検証済み Pydantic model。アプリの本物の entry point を呼び出す。
4. `teardown()` — **async**。最後の `run()` の後に **1 回だけ** 呼ばれる。`setup()` で獲得した resource を解放する。任意実装で、既定は no-op。

`setup()` と `teardown()` には既定の no-op 実装があります。共有 resource が必要な場合だけ override すれば十分です。

**Concurrency**: `run()` は `asyncio.gather` により並列実行されます。実装は **concurrency-safe** でなければなりません。共有 mutable state (SQLite connection、in-memory cache、file handle など) を使う場合は、`asyncio.Semaphore` や `asyncio.Lock` で保護してください。

```python
class AppRunnable(pixie.Runnable[AppArgs]):
    _sem: asyncio.Semaphore

    @classmethod
    def create(cls) -> AppRunnable:
        inst = cls()
        inst._sem = asyncio.Semaphore(1)  # DB access を直列化
        return inst

    async def run(self, args: AppArgs) -> None:
        async with self._sem:
            await call_app(args.message)
```

よくある concurrency pitfall:

- **SQLite**: 並列 write に安全ではない。`Semaphore(1)` を使うか、WAL mode の `aiosqlite` を使う。
- **Global mutable state**: `run()` 中に変更する module-level dict/list には保護が必要。
- **Rate-limited APIs**: 429 error 回避のため semaphore を追加する。

**Import resolution**: `pixie test` / `pixie trace` を起動した project root directory は、runnable と evaluator を読み込む前に自動的に `sys.path` に追加されます。したがって、通常の `import` 文で project module を参照できます (例: `from app import service`)。

**Example**:

```python
# pixie_qa/scripts/run_app.py
from __future__ import annotations
from pydantic import BaseModel
import pixie

class AppArgs(BaseModel):
    user_message: str

class AppRunnable(pixie.Runnable[AppArgs]):
    @classmethod
    def create(cls) -> AppRunnable:
        return cls()

    async def run(self, args: AppArgs) -> None:
        from myapp import handle_request
        await handle_request(args.user_message)
```

**Web server example** (async HTTP client を使用):

```python
import httpx
from pydantic import BaseModel
import pixie

class AppArgs(BaseModel):
    user_message: str

class AppRunnable(pixie.Runnable[AppArgs]):
    _client: httpx.AsyncClient

    @classmethod
    def create(cls) -> AppRunnable:
        return cls()

    async def setup(self) -> None:
        self._client = httpx.AsyncClient(base_url="http://localhost:8000")

    async def run(self, args: AppArgs) -> None:
        await self._client.post("/chat", json={"message": args.user_message})

    async def teardown(self) -> None:
        await self._client.aclose()
```

---

## Functions

### `pixie.wrap`

```python
pixie.wrap(data: 'T', *, purpose: "Literal['input', 'output', 'state']", name: 'str', description: 'str | None' = None) -> 'T'
```

処理パイプライン内の地点で data value または data-provider callable を観測します。

`data` は plain value でも value を生成する callable でもかまいません。どちらの場合も戻り値の型は `T` で、no-op または tracing mode では、呼び出し側は渡したものと同じ型を受け取ります。

`purpose="input"` で eval mode の場合、返される value (または callable) は deserialize された registry value へ置き換えられます。`data` が callable の場合、返される wrapper は元関数を無視し、毎回注入値を返します。他の mode では元 callable を包み、tracing や capture 動作だけを追加します。

Args:
data: data value または data-provider callable。
purpose: data point の分類。 - "input": 外部 dependency から来る data (DB record、API response) - "output": user や外部 system に出ていく data - "state": 評価用の中間 state (routing decision など)
name: この data point の一意識別子。eval registry と trace log の key に使われる。
description: その data が何かを説明する任意の human-readable description。

Returns:
元の data をそのまま返す (tracing / no-op mode)、または registry value (`purpose="input"` で eval mode)。`data` が callable の場合、戻り値も callable。

---

## Error Types

### `WrapRegistryMissError`

```python
WrapRegistryMissError(name: 'str') -> 'None'
```

`wrap(purpose="input")` の name が eval registry に存在しない場合に送出されます。

### `WrapTypeMismatchError`

```python
WrapTypeMismatchError(name: 'str', expected_type: 'type', actual_type: 'type') -> 'None'
```

deserialize された registry value の型が期待型と一致しない場合に送出されます。

---

## Trace File Utilities

wrap log entry 用 Pydantic model と JSONL 読み込み utility。

`WrapLogEntry` は、JSONL trace file に記録された 1 件の `wrap()` event を型付きで表現するモデルです。`pixie trace filter` CLI、dataset loader、verification script など複数箇所で使うため、この単一 model を共有しています。

### `pixie.WrapLogEntry`

```python
pixie.WrapLogEntry(*, type: str = 'wrap', name: str, purpose: str, data: Any, description: str | None = None, trace_id: str | None = None, span_id: str | None = None) -> None
```

JSONL trace file に記録される 1 件の wrap() event。

Attributes:
type: wrap event では常に `"wrap"`。
name: wrap point 名 (`wrap(name=...)` と一致)。
purpose: `"input"`, `"output"`, `"state"` のいずれか。
data: serialised された data (`jsonpickle` string)。
description: 任意の human-readable description。
trace_id: OTel trace ID (利用可能な場合)。
span_id: OTel span ID (利用可能な場合)。

### `pixie.load_wrap_log_entries`

```python
pixie.load_wrap_log_entries(jsonl_path: 'str | Path') -> 'list[WrapLogEntry]'
```

JSONL file からすべての wrap log entry を読み込みます。

非 wrap 行 (例: `type=llm_span`) と malformed line はスキップします。

Args:
jsonl_path: JSONL trace file の path。

Returns:
:class:`WrapLogEntry` object の list。

### `pixie.filter_by_purpose`

```python
pixie.filter_by_purpose(entries: 'list[WrapLogEntry]', purposes: 'set[str]') -> 'list[WrapLogEntry]'
```

wrap log entry を purpose で filter します。

Args:
entries: wrap log entry の list。
purposes: 含める purpose value の set。

Returns:
filter 後の list。
