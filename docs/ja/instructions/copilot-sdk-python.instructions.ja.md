---
applyTo: "**.py, pyproject.toml, setup.py"
description: "このファイルは、GitHub Copilot SDK を使用して Python アプリケーションを構築するためのガイダンスを提供します。"
name: "GitHub Copilot SDK Python Instructions"
---

## 基本原則

- SDK はテクニカルプレビュー段階にあり、重大な変更が含まれる可能性があります
- Python 3.9以降が必要です
- GitHub Copilot CLI がインストールされ、PATH に含まれている必要があります
- 全体で async/await パターンを使用します (asyncio)
- 非同期コンテキストマネージャーと手動ライフサイクル管理の両方をサポート
- IDE サポートを向上させるために提供される型ヒント

## インストール

常に pip 経由でインストールします。

```bash
pip install github-copilot-sdk
# or with poetry
poetry add github-copilot-sdk
# or with uv
uv add github-copilot-sdk
```

## クライアントの初期化

### 基本的なクライアントのセットアップ

```python
from copilot import CopilotClient, PermissionHandler
import asyncio

async def main():
    async with CopilotClient() as client:
        # Use client...
        pass

asyncio.run(main())
```

### クライアント構成オプション

CopilotClient を作成するときは、次のキーを含む辞書を使用します。

- `cli_path` - CLI 実行可能ファイルへのパス (デフォルト: PATH または COPILOT_CLI_PATH 環境変数からの「copilot」)
- `cli_url` - 既存の CLI サーバーの URL (例: "localhost:8080")。提供された場合、クライアントはプロセスを生成しません
- `port` - サーバーポート (デフォルト: ランダムの場合は 0)
- `use_stdio` - TCP の代わりに stdio トランスポートを使用します (デフォルト: True)
- `log_level` - ログレベル (デフォルト: "info")
- `auto_start` - サーバーの自動起動 (デフォルト: True)
- `auto_restart` - クラッシュ時の自動再起動 (デフォルト: True)
- `cwd` - CLI プロセスの作業ディレクトリ (デフォルト: os.getcwd())
- `env` - CLI プロセスの環境変数 (dict)

### 手動サーバー制御

明示的な制御の場合:

```python
from copilot import CopilotClient
import asyncio

async def main():
    client = CopilotClient({"auto_start": False})
    await client.start()
    # Use client...
    await client.stop()

asyncio.run(main())
```

`stop()` に時間がかかりすぎる場合は、`force_stop()` を使用してください。

## セッション管理

### セッションの作成

SessionConfig には辞書を使用します。

```python
session = await client.create_session({
    "on_permission_request": PermissionHandler.approve_all,
    "model": "gpt-5",
    "streaming": True,
    "tools": [...],
    "system_message": { ... },
    "available_tools": ["tool1", "tool2"],
    "excluded_tools": ["tool3"],
    "provider": { ... }
})
```

### セッション構成オプション

- `session_id` - カスタムセッションID (str)
- `model` - モデル名 (「gpt-5」、「claude-sonnet-4.5」など)
- `tools` - CLI に公開されるカスタムツール (list[Tool])
- `system_message` - システムメッセージのカスタマイズ (dict)
- `available_tools` - ツール名の許可リスト (list[str])
- `excluded_tools` - ツール名のブロックリスト (list[str])
- `provider` - カスタム API プロバイダー構成 (BYOK) (ProviderConfig)
- `streaming` - ストリーミング応答チャンクを有効にする (ブール値)
- `mcp_servers` - MCP サーバー構成 (リスト)
- `custom_agents` - カスタムエージェント構成 (リスト)
- `config_dir` - 構成ディレクトリの上書き (str)
- `skill_directories` - スキルディレクトリ (list[str])
- `disabled_skills` - 無効化されたスキル (list[str])
- `on_permission_request` - 許可要求ハンドラー (呼び出し可能)

### セッションの再開

```python
session = await client.resume_session("session-id", {
    "on_permission_request": PermissionHandler.approve_all,
    "tools": [my_new_tool]
})
```

### セッション操作

- `session.session_id` - セッション識別子 (str) を取得します。
- `await session.send({"prompt": "...", "attachments": [...]})` - メッセージを送信し、str (メッセージ ID) を返します。
- `await session.send_and_wait({"prompt": "..."}, timeout=60.0)` - 送信してアイドル状態になるまで待機し、SessionEvent を返します。なし
- `await session.abort()` - 現在の処理を中止します
- `await session.get_messages()` - すべてのイベント/メッセージを取得し、list[SessionEvent] を返します。
- `await session.destroy()` - クリーンアップセッション

## イベント処理

### イベントサブスクリプションパターン

セッションイベントを待機するには、常に asyncio イベントまたは Future を使用してください。

```python
import asyncio

done = asyncio.Event()

def handler(event):
    if event.type == "assistant.message":
        print(event.data.content)
    elif event.type == "session.idle":
        done.set()

session.on(handler)
await session.send({"prompt": "..."})
await done.wait()
```

### イベントの登録解除

`on()` メソッドは、サブスクライブを解除する関数を返します。

```python
unsubscribe = session.on(lambda event: print(event.type))
# Later...
unsubscribe()
```

### イベントの種類

イベントタイプのチェックには属性アクセスを使用します。

```python
def handler(event):
    if event.type == "user.message":
        # Handle user message
        pass
    elif event.type == "assistant.message":
        print(event.data.content)
    elif event.type == "tool.executionStart":
        # Tool execution started
        pass
    elif event.type == "tool.executionComplete":
        # Tool execution completed
        pass
    elif event.type == "session.start":
        # Session started
        pass
    elif event.type == "session.idle":
        # Session is idle (processing complete)
        pass
    elif event.type == "session.error":
        print(f"Error: {event.data.message}")

session.on(handler)
```

## ストリーミング応答

### ストリーミングを有効にする

SessionConfig で `streaming: True` を設定します。

```python
session = await client.create_session({
    "on_permission_request": PermissionHandler.approve_all,
    "model": "gpt-5",
    "streaming": True
})
```

### ストリーミングイベントの処理

デルタイベント (増分) と最終イベントの両方を処理します。

```python
import asyncio

done = asyncio.Event()

def handler(event):
    if event.type == "assistant.message.delta":
        # Incremental text chunk
        print(event.data.delta_content, end="", flush=True)
    elif event.type == "assistant.reasoning.delta":
        # Incremental reasoning chunk (model-dependent)
        print(event.data.delta_content, end="", flush=True)
    elif event.type == "assistant.message":
        # Final complete message
        print("\n--- Final ---")
        print(event.data.content)
    elif event.type == "assistant.reasoning":
        # Final reasoning content
        print("--- Reasoning ---")
        print(event.data.content)
    elif event.type == "session.idle":
        done.set()

session.on(handler)
await session.send({"prompt": "Tell me a story"})
await done.wait()
```

注: 最終イベント (`assistant.message`、`assistant.reasoning`) は、ストリーミング設定に関係なく常に送信されます。

## カスタムツール

### define_tool を使用したツールの定義

ツール定義には `define_tool` を使用します。

```python
from copilot import define_tool

async def fetch_issue(issue_id: str):
    # Fetch issue from tracker
    return {"id": issue_id, "status": "open"}

session = await client.create_session({
    "on_permission_request": PermissionHandler.approve_all,
    "model": "gpt-5",
    "tools": [
        define_tool(
            name="lookup_issue",
            description="Fetch issue details from tracker",
            parameters={
                "type": "object",
                "properties": {
                    "id": {"type": "string", "description": "Issue ID"}
                },
                "required": ["id"]
            },
            handler=lambda args, inv: fetch_issue(args["id"])
        )
    ]
})
```

### パラメータに Pydantic を使用する

SDK は Pydantic モデルとうまく連携します。

```python
from pydantic import BaseModel, Field

class WeatherArgs(BaseModel):
    location: str = Field(description="City name")
    units: str = Field(default="fahrenheit", description="Temperature units")

async def get_weather(args: WeatherArgs, inv):
    return {"temperature": 72, "units": args.units}

session = await client.create_session({
    "on_permission_request": PermissionHandler.approve_all,
    "tools": [
        define_tool(
            name="get_weather",
            description="Get weather for a location",
            parameters=WeatherArgs.model_json_schema(),
            handler=lambda args, inv: get_weather(WeatherArgs(**args), inv)
        )
    ]
})
```

### ツールの戻り値の型

- 任意の JSON シリアル化可能な値を返します (自動的にラップされます)。
- または、フルコントロールの ToolResult dict を返します。

```python
{
    "text_result_for_llm": str,  # Result shown to LLM
    "result_type": "success" | "failure",
    "error": str,  # Optional: Internal error (not shown to LLM)
    "tool_telemetry": dict  # Optional: Telemetry data
}
```

### ツールハンドラーの署名

ツールハンドラーは 2 つの引数を受け取ります。

- `args` (dict) - LLM によって渡されるツール引数
- `invocation` (ToolInvocation) - 呼び出しに関するメタデータ
  - `invocation.session_id` - セッションID
  - `invocation.tool_call_id` - ツール呼び出し ID
  - `invocation.tool_name` - ツール名
  - `invocation.arguments` - args パラメータと同じ

### ツールの実行フロー

Copilot がツールを呼び出すと、クライアントは自動的に次のことを行います。

1. ハンドラー関数を実行します
2. 戻り値をシリアル化します
3. CLIに応答します

## システムメッセージのカスタマイズ

### 追加モード (デフォルト - ガードレールを保持)

```python
session = await client.create_session({
    "on_permission_request": PermissionHandler.approve_all,
    "model": "gpt-5",
    "system_message": {
        "mode": "append",
        "content": """
<workflow_rules>
- Always check for security vulnerabilities
- Suggest performance improvements when applicable
</workflow_rules>
"""
    }
})
```

### 置換モード (フルコントロール - ガードレールを削除)

```python
session = await client.create_session({
    "on_permission_request": PermissionHandler.approve_all,
    "model": "gpt-5",
    "system_message": {
        "mode": "replace",
        "content": "You are a helpful assistant."
    }
})
```

## 添付ファイル

メッセージにファイルを添付します。

```python
await session.send({
    "prompt": "Analyze this file",
    "attachments": [
        {
            "type": "file",
            "path": "/path/to/file.py",
            "display_name": "My File"
        }
    ]
})
```

## メッセージ配信モード

メッセージオプションで `mode` キーを使用します。

- `"enqueue"` - メッセージを処理のためにキューに入れます
- `"immediate"` - メッセージを直ちに処理します

```python
await session.send({
    "prompt": "...",
    "mode": "enqueue"
})
```

## 複数のセッション

セッションは独立しており、同時に実行できます。

```python
session1 = await client.create_session({
    "on_permission_request": PermissionHandler.approve_all,
    "model": "gpt-5",
})
session2 = await client.create_session({
    "on_permission_request": PermissionHandler.approve_all,
    "model": "claude-sonnet-4.5",
})

await asyncio.gather(
    session1.send({"prompt": "Hello from session 1"}),
    session2.send({"prompt": "Hello from session 2"})
)
```

## 自分のキーの持ち込み (BYOK)

`provider` 経由でカスタム API プロバイダーを使用します。

```python
session = await client.create_session({
    "on_permission_request": PermissionHandler.approve_all,
    "provider": {
        "type": "openai",
        "base_url": "https://api.openai.com/v1",
        "api_key": "your-api-key"
    }
})
```

## セッションのライフサイクル管理

### セッションのリスト表示

```python
sessions = await client.list_sessions()
for metadata in sessions:
    print(f"{metadata.session_id}: {metadata.summary}")
```

### セッションの削除

```python
await client.delete_session(session_id)
```

### 最後のセッションIDの取得

```python
last_id = await client.get_last_session_id()
if last_id:
    session = await client.resume_session(last_id, on_permission_request=PermissionHandler.approve_all)
```

### 接続状態の確認

```python
state = client.get_state()
# Returns: "disconnected" | "connecting" | "connected" | "error"
```

## エラー処理

### 標準例外処理

```python
try:
    session = await client.create_session(on_permission_request=PermissionHandler.approve_all)
    await session.send({"prompt": "Hello"})
except Exception as e:
    print(f"Error: {e}")
```

### セッションエラーイベント

`session.error` イベントタイプを監視して実行時エラーを検出します。

```python
def handler(event):
    if event.type == "session.error":
        print(f"Session Error: {event.data.message}")

session.on(handler)
```

## 接続テスト

ping を使用してサーバーの接続を確認します。

```python
response = await client.ping("health check")
print(f"Server responded at {response['timestamp']}")
```

## リソースのクリーンアップ

### コンテキストマネージャーによる自動クリーンアップ

自動クリーンアップには常に非同期コンテキストマネージャーを使用してください。

```python
async with CopilotClient() as client:
    async with await client.create_session(on_permission_request=PermissionHandler.approve_all) as session:
        # Use session...
        await session.send({"prompt": "Hello"})
    # Session automatically destroyed
# Client automatically stopped
```

### Try-Finally による手動クリーンアップ

```python
client = CopilotClient()
try:
    await client.start()
    session = await client.create_session(on_permission_request=PermissionHandler.approve_all)
    try:
        # Use session...
        pass
    finally:
        await session.destroy()
finally:
    await client.stop()
```

## ベストプラクティス

1. **自動クリーンアップには常に非同期コンテキストマネージャーを使用してください** (`async with`)
2. **asyncio.Event または asyncio.Future** を使用して session.idle イベントを待機します
3. **堅牢なエラー処理のために session.error** イベントを処理する
4. **イベントタイプのチェックには if/elif チェーンを使用します**
5. **ストリーミングを有効にする** ことで、インタラクティブなシナリオでの UX を向上させます
6. **ツール定義にはdefine_toolを使用**
7. **タイプセーフなパラメーター検証には Pydantic モデルを使用します**
8. **不要になったらイベントサブスクリプションを破棄**
9. **安全ガードレールを維持するには、モード: "append" で system_message を使用します**
10. **ストリーミングが有効な場合、デルタイベントと最終イベントの両方を処理します**
11. **タイプヒントを使用**して、IDE サポートを改善し、コードを明確にします

## よくあるパターン

### 単純なクエリと応答

```python
from copilot import CopilotClient, PermissionHandler
import asyncio

async def main():
    async with CopilotClient() as client:
        async with await client.create_session({
            "on_permission_request": PermissionHandler.approve_all,
            "model": "gpt-5",
        }) as session:
            done = asyncio.Event()

            def handler(event):
                if event.type == "assistant.message":
                    print(event.data.content)
                elif event.type == "session.idle":
                    done.set()

            session.on(handler)
            await session.send({"prompt": "What is 2+2?"})
            await done.wait()

asyncio.run(main())
```

### マルチターン会話

```python
async def send_and_wait(session, prompt: str):
    done = asyncio.Event()
    result = []

    def handler(event):
        if event.type == "assistant.message":
            result.append(event.data.content)
            print(event.data.content)
        elif event.type == "session.idle":
            done.set()
        elif event.type == "session.error":
            result.append(None)
            done.set()

    unsubscribe = session.on(handler)
    await session.send({"prompt": prompt})
    await done.wait()
    unsubscribe()

    return result[0] if result else None

async with await client.create_session(on_permission_request=PermissionHandler.approve_all) as session:
    await send_and_wait(session, "What is the capital of France?")
    await send_and_wait(session, "What is its population?")
```

### SendAndWait ヘルパー

```python
# Use built-in send_and_wait for simpler synchronous interaction
async with await client.create_session(on_permission_request=PermissionHandler.approve_all) as session:
    response = await session.send_and_wait(
        {"prompt": "What is 2+2?"},
        timeout=60.0
    )

    if response and response.type == "assistant.message":
        print(response.data.content)
```

### データクラスの戻り値の型を持つツール

```python
from dataclasses import dataclass, asdict
from copilot import define_tool

@dataclass
class UserInfo:
    id: str
    name: str
    email: str
    role: str

async def get_user(args, inv) -> dict:
    user = UserInfo(
        id=args["user_id"],
        name="John Doe",
        email="john@example.com",
        role="Developer"
    )
    return asdict(user)

session = await client.create_session({
    "on_permission_request": PermissionHandler.approve_all,
    "tools": [
        define_tool(
            name="get_user",
            description="Retrieve user information",
            parameters={
                "type": "object",
                "properties": {
                    "user_id": {"type": "string", "description": "User ID"}
                },
                "required": ["user_id"]
            },
            handler=get_user
        )
    ]
})
```

### 進行中のストリーミング

```python
import asyncio

current_message = []
done = asyncio.Event()

def handler(event):
    if event.type == "assistant.message.delta":
        current_message.append(event.data.delta_content)
        print(event.data.delta_content, end="", flush=True)
    elif event.type == "assistant.message":
        print(f"\n\n=== Complete ===")
        print(f"Total length: {len(event.data.content)} chars")
    elif event.type == "session.idle":
        done.set()

unsubscribe = session.on(handler)
await session.send({"prompt": "Write a long story"})
await done.wait()
unsubscribe()
```

### エラー回復

```python
def handler(event):
    if event.type == "session.error":
        print(f"Session error: {event.data.message}")
        # Optionally retry or handle error

session.on(handler)

try:
    await session.send({"prompt": "risky operation"})
except Exception as e:
    # Handle send errors
    print(f"Failed to send: {e}")
```

### TypedDict を使用したタイプセーフティ

```python
from typing import TypedDict, List

class MessageOptions(TypedDict, total=False):
    prompt: str
    attachments: List[dict]
    mode: str

class SessionConfig(TypedDict, total=False):
    model: str
    streaming: bool
    tools: List

# Usage with type hints
options: MessageOptions = {
    "prompt": "Hello",
    "mode": "enqueue"
}
await session.send(options)

config: SessionConfig = {
    "on_permission_request": PermissionHandler.approve_all,
    "model": "gpt-5",
    "streaming": True
}
session = await client.create_session(config)
```

### ストリーミング用の非同期ジェネレーター

```python
from typing import AsyncGenerator

async def stream_response(session, prompt: str) -> AsyncGenerator[str, None]:
    """Stream response chunks as an async generator."""
    queue = asyncio.Queue()
    done = asyncio.Event()

    def handler(event):
        if event.type == "assistant.message.delta":
            queue.put_nowait(event.data.delta_content)
        elif event.type == "session.idle":
            done.set()

    unsubscribe = session.on(handler)
    await session.send({"prompt": prompt})

    while not done.is_set():
        try:
            chunk = await asyncio.wait_for(queue.get(), timeout=0.1)
            yield chunk
        except asyncio.TimeoutError:
            continue

    # Drain remaining items
    while not queue.empty():
        yield queue.get_nowait()

    unsubscribe()

# Usage
async for chunk in stream_response(session, "Tell me a story"):
    print(chunk, end="", flush=True)
```

### ツールのデコレータパターン

```python
from typing import Callable, Any
from copilot import define_tool

def copilot_tool(
    name: str,
    description: str,
    parameters: dict
) -> Callable:
    """Decorator to convert a function into a Copilot tool."""
    def decorator(func: Callable) -> Any:
        return define_tool(
            name=name,
            description=description,
            parameters=parameters,
            handler=lambda args, inv: func(**args)
        )
    return decorator

@copilot_tool(
    name="calculate",
    description="Perform a calculation",
    parameters={
        "type": "object",
        "properties": {
            "expression": {"type": "string", "description": "Math expression"}
        },
        "required": ["expression"]
    }
)
def calculate(expression: str) -> float:
    return eval(expression)

session = await client.create_session({
    "on_permission_request": PermissionHandler.approve_all,
    "tools": [calculate]})
```

## Python 固有の機能

### 非同期コンテキストマネージャープロトコル

SDK は `__aenter__` と `__aexit__` を実装します。

```python
class CopilotClient:
    async def __aenter__(self):
        await self.start()
        return self

    async def __aexit__(self, exc_type, exc_val, exc_tb):
        await self.stop()
        return False

class CopilotSession:
    async def __aenter__(self):
        return self

    async def __aexit__(self, exc_type, exc_val, exc_tb):
        await self.destroy()
        return False
```

### データクラスのサポート

イベントデータは属性として利用できます。

```python
def handler(event):
    # Access event attributes directly
    print(event.type)
    print(event.data.content)  # For assistant.message
    print(event.data.delta_content)  # For assistant.message.delta
```
