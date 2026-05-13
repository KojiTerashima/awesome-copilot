# Step 2: Instrument with `wrap` and capture a reference trace

> 完全な `wrap()` API、`Runnable` class、CLI command については `wrap-api.md` を参照してください。

**Why this step**: 何かを作る前に、まずアプリ内を流れる実データを見る必要があります。このステップでは、データ境界を示す `wrap()` を追加し、`Runnable` class を実装し、`pixie trace` で reference trace を取得し、すべての eval criterion が評価可能であることを確認します。

このステップは 3 つをまとめたものです: (1) data-flow analysis、(2) instrumentation、(3) runnable の作成です。

---

## 2a. Data-flow analysis and `wrap` instrumentation

LLM call site を起点に、コードを前後へたどって次を見つけてください。

- **Entry input**: user が entry point 経由で送るもの
- **Dependency input**: 外部 system (database、API、cache) から来る data
- **App output**: user や外部 system へ出ていく data
- **Intermediate state**: 評価に関連する内部 decision (routing、tool call)

見つけた各 data point について、**すぐに application code に `wrap()` call を追加**します。

```python
import pixie

# External dependency data — value form (DB/API 呼び出し結果)
profile = pixie.wrap(db.get_profile(user_id), purpose="input", name="customer_profile",
    description="Customer profile fetched from database")

# External dependency data — function form (遅延評価 / 呼び出し回避用)
history = pixie.wrap(redis.get_history, purpose="input", name="conversation_history",
    description="Conversation history from Redis")(session_id)

# App output — user が受け取るもの
response = pixie.wrap(response_text, purpose="output", name="response",
    description="The assistant's response to the user")

# Intermediate state — 評価に関係する内部 decision
selected_agent = pixie.wrap(selected_agent, purpose="state", name="routing_decision",
    description="Which agent was selected to handle this request")
```

### Rules for wrapping

1. **Wrap at the data boundary** — utility function の奥深くではなく、data がアプリへ入る/出る境界で wrap する
2. **Names must be unique** — application 全体で name は一意でなければならない (registry key と dataset field name に使われる)
3. **Use `lower_snake_case`** for names
4. **Don't wrap LLM call arguments or responses** — それらは OpenInference の auto-instrumentation で既に捕捉される
5. **Don't change the function's interface** — `wrap()` は純粋な追加であり、同じ型を返す

### Value vs. function wrapping

```python
# Value form: data value を wrap (結果はすでに計算済み)
profile = pixie.wrap(db.get_profile(user_id), purpose="input", name="customer_profile")

# Function form: callable 自体を wrap — eval mode では元の関数は
# 呼ばれず、代わりに registry value が返る
profile = pixie.wrap(db.get_profile, purpose="input", name="customer_profile")(user_id)
```

function form は、eval mode で外部呼び出し自体を防ぎたいときに使います (高コスト、side effect がある、きれいな injection point が欲しいなど)。tracing mode では関数は通常どおり呼ばれ、結果が log されます。

### Coverage check

`wrap()` call を追加したら、`pixie_qa/02-eval-criteria.md` の各 eval criterion を見直し、必要なすべての data point に対応する wrap call があることを確認してください。criterion に必要な data が捕捉されていなければ、その場で wrap を追加します。先送りしてはいけません。

## 2b. Implement the Runnable class

`Runnable` class は、この skill の旧バージョンで使っていた plain function を置き換えるものです。次の 3 つの lifecycle method を公開します。

- **`setup()`** — async。`run()` が一度も呼ばれる前に 1 回だけ呼ばれる。共有 resource (例: async HTTP client、DB connection、事前読み込み済み config) の初期化をここで行う。省略可で、既定は no-op。
- **`run(args)`** — async。各 dataset entry について **並列に** 呼ばれる (最大 4 並列)。`entry_kwargs` から作られた検証済み Pydantic model を `args` として受け取り、アプリの **本物の entry point** を呼ぶ。**並行実行に安全であることが必須**。
- **`teardown()`** — async。すべての `run()` の後に 1 回だけ呼ばれる。resource を解放する。省略可で、既定は no-op。

**Import resolution**: runnable 読み込み時には project root が自動で `sys.path` に追加されるため、通常の `import` 文を使えます (例: `from app import service`)。`sys.path` を自分で操作する必要はありません。

class は `pixie_qa/scripts/run_app.py` に置きます。

```python
# pixie_qa/scripts/run_app.py
from __future__ import annotations
from pydantic import BaseModel
import pixie


class AppArgs(BaseModel):
    user_message: str


class AppRunnable(pixie.Runnable[AppArgs]):
    """Runnable that drives the application for tracing and evaluation.

    wrap(purpose="input") calls in the app inject dependency data from the
    test registry automatically.  wrap(purpose="output"/"state") calls
    capture data for evaluation.  No manual mocking needed.
    """

    @classmethod
    def create(cls) -> AppRunnable:
        return cls()

    async def run(self, args: AppArgs) -> None:
        from myapp import handle_request
        await handle_request(args.user_message)
```

**For web servers**: async HTTP client を `setup()` で初期化し、`run()` で使います。

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

**For FastAPI/Starlette apps** (server を起動せずに in-process でテストする場合) は、ASGI app を直接実行するために `httpx.ASGITransport` を使います。これは高速で、port 管理も不要です。

```python
import asyncio
import httpx
from pydantic import BaseModel
import pixie


class AppArgs(BaseModel):
    user_message: str


class AppRunnable(pixie.Runnable[AppArgs]):
    _client: httpx.AsyncClient
    _sem: asyncio.Semaphore

    @classmethod
    def create(cls) -> AppRunnable:
        inst = cls()
        inst._sem = asyncio.Semaphore(1)  # app が共有 mutable state を使うなら直列化
        return inst

    async def setup(self) -> None:
        from myapp.main import app  # your FastAPI/Starlette app instance

        # ASGITransport runs the app in-process — no server needed
        transport = httpx.ASGITransport(app=app)
        self._client = httpx.AsyncClient(transport=transport, base_url="http://test")

    async def run(self, args: AppArgs) -> None:
        async with self._sem:
            await self._client.post("/chat", json={"message": args.user_message})

    async def teardown(self) -> None:
        await self._client.aclose()
```

適切な pattern を選んでください。

- **Direct function call**: app が単純な async function を公開している場合 (web framework なし)
- **`httpx.AsyncClient` with `base_url`**: 起動済み HTTP server に対してテストする必要がある場合
- **`httpx.ASGITransport`**: app が FastAPI/Starlette の場合。最速で server 不要、eval 向けに最も安定

**Rules**:

- `run()` method は、dataset の `entry_kwargs` からフィールドが埋められた Pydantic model を受け取ります。アプリが必要とする field を持つ `BaseModel` subclass を定義してください。
- すべての lifecycle method (`setup`, `run`, `teardown`) は **async** です。
- `run()` は app の本物の entry point を通じて呼び出さなければなりません。request handling を bypass してはいけません。
- file は `pixie_qa/scripts/run_app.py` に置き、class 名は `AppRunnable` (または分かりやすい名前) にしてください。
- dataset の `"runnable"` field は class を参照します: `"pixie_qa/scripts/run_app.py:AppRunnable"`.

**Concurrency**: `run()` は複数の dataset entry に対して並列に呼ばれます (最大 4 並列)。SQLite、file-based DB、global cache など共有 mutable state を app が使うなら、アクセスを同期しなければなりません。

```python
import asyncio

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

- **SQLite**: `sqlite3` connection は async write の並列実行に安全ではありません。`Semaphore(1)` で直列化するか、WAL mode の `aiosqlite` に切り替えてください。
- **Global mutable state**: `run()` 中に変更される module-level dict/list には lock が必要です。
- **Rate-limited external APIs**: 429 error を避けるため semaphore を追加してください。

## 2c. Capture the reference trace with `pixie trace`

`pixie trace` CLI command を使って `Runnable` を実行し、trace file を取得します。entry input は JSON file として渡します。

```bash
# entry kwargs を持つ JSON ファイルを作成
echo '{"user_message": "a realistic sample input"}' > pixie_qa/sample-input.json

pixie trace --runnable pixie_qa/scripts/run_app.py:AppRunnable \
  --input pixie_qa/sample-input.json \
  --output pixie_qa/reference-trace.jsonl
```

`--input` flag は **file path** として JSON file を受け取ります (inline JSON ではありません)。JSON object の key が、Pydantic model に渡される kwargs になります。

この command は `AppRunnable.create()` を呼び、その後に `setup()`、与えられた input で 1 回 `run(args)`、最後に `teardown()` を呼びます。生成された trace は output file に書き出されます。

JSONL trace file には、各 `wrap()` event と各 LLM span が 1 行ずつ入ります。

```jsonl
{"type": "kwargs", "value": {"user_message": "What are your hours?"}}
{"type": "wrap", "name": "customer_profile", "purpose": "input", "data": {...}, ...}
{"type": "llm_span", "request_model": "gpt-4o", "input_messages": [...], ...}
{"type": "wrap", "name": "response", "purpose": "output", "data": "Our hours are...", ...}
```

## 2d. Verify wrap coverage with `pixie format`

`pixie format` を trace file に対して実行し、dataset-entry 形式で data を確認します。これにより data shape と、実際の app output がどう見えるかの両方を把握できます。

```bash
pixie format --input reference-trace.jsonl --output dataset-sample.json
```

出力は整形済み dataset entry template で、次を含みます。

- `entry_kwargs`: runnable argument の正確な key/value
- `eval_input`: すべての dependency data (`wrap(purpose="input")` call 由来)
- `eval_output`: trace から捕捉した **実際の app output** (これは実出力であり、dataset の `eval_output` field として固定するものではなく、アプリが何を返すかを理解するために使う)

`pixie_qa/02-eval-criteria.md` の各 eval criterion について、format 出力に評価に必要な data が含まれていることを確認してください。足りない data point があれば、戻って `wrap()` call を追加します。

---

## Output

- `pixie_qa/scripts/run_app.py` — `Runnable` class
- `pixie_qa/reference-trace.jsonl` — 期待するすべての wrap event を含む reference trace
