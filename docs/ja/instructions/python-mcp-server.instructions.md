---
description: 'Python SDK を使って Model Context Protocol (MCP) サーバーを構築するための instruction'
applyTo: '**/*.py, **/pyproject.toml, **/requirements.txt'
---

# Python MCP Server 開発

## 指示

- project 管理には **uv** を使う: `uv init mcp-server-demo` と `uv add "mcp[cli]"`
- `mcp.server.fastmcp` から FastMCP を import する: `from mcp.server.fastmcp import FastMCP`
- 登録には `@mcp.tool()`、`@mcp.resource()`、`@mcp.prompt()` decorator を使う
- type hint は必須。schema 生成と validation に使われる
- structured output には Pydantic model、TypedDict、dataclass を使う
- return type が互換であれば、tool は自動的に structured output を返す
- stdio transport には `mcp.run()` または `mcp.run(transport="stdio")` を使う
- HTTP server には `mcp.run(transport="streamable-http")` を使うか、Starlette / FastAPI に mount する
- MCP 機能へアクセスするには、tool / resource で `Context` parameter を使う: `ctx: Context`
- log は `await ctx.debug()`、`await ctx.info()`、`await ctx.warning()`、`await ctx.error()` で送る
- 進捗報告には `await ctx.report_progress(progress, total, message)` を使う
- ユーザー入力の要求には `await ctx.elicit(message, schema)` を使う
- LLM sampling には `await ctx.session.create_message(messages, max_tokens)` を使う
- server、tool、resource、prompt の icon 設定には `Icon(src="path", mimeType="image/png")` を使う
- 画像の自動処理には `Image` class を使う: `return Image(data=bytes, format="png")`
- URI pattern で resource template を定義する: `@mcp.resource("greeting://{name}")`
- completion support は partial value を受け取って suggestion を返す形で実装する
- 共有 resource を持つ startup / shutdown には lifespan context manager を使う
- tool 内では `ctx.request_context.lifespan_context` から lifespan context にアクセスする
- stateless HTTP server には、FastMCP 初期化で `stateless_http=True` を設定する
- 現代的な client 向け JSON response には `json_response=True` を有効にする
- server のテストには次を使う: `uv run mcp dev server.py` (Inspector) または `uv run mcp install server.py` (Claude Desktop)
- Starlette では異なる path で複数 server を mount できる: `Mount("/path", mcp.streamable_http_app())`
- browser client 向けには CORS を設定し、`Mcp-Session-Id` header を公開する
- FastMCP で不十分な場合は、最大限の制御のため low-level Server class を使う

## ベスト プラクティス

- 常に type hint を使う。schema 生成と validation を駆動するため
- structured tool output には Pydantic model または TypedDict を返す
- tool 関数は単一責任に集中させる
- docstring は明確に書く。tool description になるため
- type hint 付きの説明的な parameter 名を使う
- Pydantic の Field description で入力を検証する
- try-except block による適切なエラー処理を実装する
- I/O-bound 操作には async 関数を使う
- lifespan context manager で resource を cleanup する
- stdio transport と干渉しないよう、stdio 使用時の log は stderr に出す
- 設定には environment variable を使う
- LLM 統合前に tool を個別にテストする
- file system や network access を公開する場合は security を考慮する
- machine-readable data には structured output を使う
- 後方互換性のため、content と structured data の両方を提供する

## よくあるパターン

### 基本的な Server Setup (stdio)
```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("My Server")

@mcp.tool()
def calculate(a: int, b: int, op: str) -> int:
    """Perform calculation"""
    if op == "add":
        return a + b
    return a - b

if __name__ == "__main__":
    mcp.run()  # stdio by default
```

### HTTP Server
```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("My HTTP Server")

@mcp.tool()
def hello(name: str = "World") -> str:
    """Greet someone"""
    return f"Hello, {name}!"

if __name__ == "__main__":
    mcp.run(transport="streamable-http")
```

### Structured Output を返す Tool
```python
from pydantic import BaseModel, Field

class WeatherData(BaseModel):
    temperature: float = Field(description="Temperature in Celsius")
    condition: str
    humidity: float

@mcp.tool()
def get_weather(city: str) -> WeatherData:
    """Get weather for a city"""
    return WeatherData(
        temperature=22.5,
        condition="sunny",
        humidity=65.0
    )
```

### Dynamic Resource
```python
@mcp.resource("users://{user_id}")
def get_user(user_id: str) -> str:
    """Get user profile data"""
    return f"User {user_id} profile data"
```

### Context を使う Tool
```python
from mcp.server.fastmcp import Context
from mcp.server.session import ServerSession

@mcp.tool()
async def process_data(
    data: str,
    ctx: Context[ServerSession, None]
) -> str:
    """Process data with logging"""
    await ctx.info(f"Processing: {data}")
    await ctx.report_progress(0.5, 1.0, "Halfway done")
    return f"Processed: {data}"
```

### Sampling を使う Tool
```python
from mcp.server.fastmcp import Context
from mcp.server.session import ServerSession
from mcp.types import SamplingMessage, TextContent

@mcp.tool()
async def summarize(
    text: str,
    ctx: Context[ServerSession, None]
) -> str:
    """Summarize text using LLM"""
    result = await ctx.session.create_message(
        messages=[SamplingMessage(
            role="user",
            content=TextContent(type="text", text=f"Summarize: {text}")
        )],
        max_tokens=100
    )
    return result.content.text if result.content.type == "text" else ""
```

### Lifespan Management
```python
from contextlib import asynccontextmanager
from dataclasses import dataclass
from mcp.server.fastmcp import FastMCP, Context

@dataclass
class AppContext:
    db: Database

@asynccontextmanager
async def app_lifespan(server: FastMCP):
    db = await Database.connect()
    try:
        yield AppContext(db=db)
    finally:
        await db.disconnect()

mcp = FastMCP("My App", lifespan=app_lifespan)

@mcp.tool()
def query(sql: str, ctx: Context) -> str:
    """Query database"""
    db = ctx.request_context.lifespan_context.db
    return db.execute(sql)
```

### Message を返す Prompt
```python
from mcp.server.fastmcp.prompts import base

@mcp.prompt(title="Code Review")
def review_code(code: str) -> list[base.Message]:
    """Create code review prompt"""
    return [
        base.UserMessage("Review this code:"),
        base.UserMessage(code),
        base.AssistantMessage("I'll review the code for you.")
    ]
```

### エラー処理
```python
@mcp.tool()
async def risky_operation(input: str) -> str:
    """Operation that might fail"""
    try:
        result = await perform_operation(input)
        return f"Success: {result}"
    except Exception as e:
        return f"Error: {str(e)}"
```
