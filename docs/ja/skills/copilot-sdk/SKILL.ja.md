---
name: copilot-sdk
description: GitHub Copilot SDK を使ってエージェント型アプリケーションを構築します。アプリに AI エージェントを組み込むとき、カスタムツールを作成するとき、ストリーミング応答を実装するとき、セッションを管理するとき、MCP サーバーに接続するとき、またはカスタムエージェントを作成するときに使用します。Copilot SDK、GitHub SDK、agentic app、embed Copilot、programmable agent、MCP server、custom agent でトリガーされます。
---

# GitHub Copilot SDK

Python、TypeScript、Go、または .NET を使って、Copilot のエージェント型ワークフローをあらゆるアプリケーションに組み込めます。

## 概要

GitHub Copilot SDK は、Copilot CLI の背後にあるものと同じエンジンを公開します。つまり、本番環境で実証済みのエージェントランタイムをプログラムから呼び出せます。独自にオーケストレーションを構築する必要はありません。エージェントの振る舞いを定義すれば、計画、ツール呼び出し、ファイル編集などは Copilot が処理します。

## 前提条件

1. **GitHub Copilot CLI** がインストール済みかつ認証済みであること（[インストールガイド](https://docs.github.com/en/copilot/how-tos/set-up/install-copilot-cli)）
2. **言語ランタイム**: Node.js 18+、Python 3.8+、Go 1.21+、または .NET 8.0+

CLI の確認: `copilot --version`

## インストール

### Node.js/TypeScript
```bash
mkdir copilot-demo && cd copilot-demo
npm init -y --init-type module
npm install @github/copilot-sdk tsx
```

### Python
```bash
pip install github-copilot-sdk
```

### Go
```bash
mkdir copilot-demo && cd copilot-demo
go mod init copilot-demo
go get github.com/github/copilot-sdk/go
```

### .NET
```bash
dotnet new console -n CopilotDemo && cd CopilotDemo
dotnet add package GitHub.Copilot.SDK
```

## クイックスタート

### TypeScript
```typescript
import { CopilotClient, approveAll } from "@github/copilot-sdk";

const client = new CopilotClient();
const session = await client.createSession({
    onPermissionRequest: approveAll,
    model: "gpt-4.1",
});

const response = await session.sendAndWait({ prompt: "What is 2 + 2?" });
console.log(response?.data.content);

await client.stop();
process.exit(0);
```

実行: `npx tsx index.ts`

### Python
```python
import asyncio
from copilot import CopilotClient, PermissionHandler

async def main():
    client = CopilotClient()
    await client.start()

    session = await client.create_session({
        "on_permission_request": PermissionHandler.approve_all,
        "model": "gpt-4.1",
    })
    response = await session.send_and_wait({"prompt": "What is 2 + 2?"})

    print(response.data.content)
    await client.stop()

asyncio.run(main())
```

### Go
```go
package main

import (
    "fmt"
    "log"
    "os"
    copilot "github.com/github/copilot-sdk/go"
)

func main() {
    client := copilot.NewClient(nil)
    if err := client.Start(); err != nil {
        log.Fatal(err)
    }
    defer client.Stop()

    session, err := client.CreateSession(&copilot.SessionConfig{
        OnPermissionRequest: copilot.PermissionHandler.ApproveAll,
        Model:               "gpt-4.1",
    })
    if err != nil {
        log.Fatal(err)
    }

    response, err := session.SendAndWait(copilot.MessageOptions{Prompt: "What is 2 + 2?"}, 0)
    if err != nil {
        log.Fatal(err)
    }

    fmt.Println(*response.Data.Content)
    os.Exit(0)
}
```

### .NET (C#)
```csharp
using GitHub.Copilot.SDK;

await using var client = new CopilotClient();
await using var session = await client.CreateSessionAsync(new SessionConfig
{
    OnPermissionRequest = PermissionHandler.ApproveAll,
    Model = "gpt-4.1",
});

var response = await session.SendAndWaitAsync(new MessageOptions { Prompt = "What is 2 + 2?" });
Console.WriteLine(response?.Data.Content);
```

実行: `dotnet run`

## ストリーミング応答

より良い UX のためにリアルタイム出力を有効化します:

### TypeScript
```typescript
import { CopilotClient, approveAll, SessionEvent } from "@github/copilot-sdk";

const client = new CopilotClient();
const session = await client.createSession({
    onPermissionRequest: approveAll,
    model: "gpt-4.1",
    streaming: true,
});

session.on((event: SessionEvent) => {
    if (event.type === "assistant.message_delta") {
        process.stdout.write(event.data.deltaContent);
    }
    if (event.type === "session.idle") {
        console.log(); // 完了時に改行
    }
});

await session.sendAndWait({ prompt: "Tell me a short joke" });

await client.stop();
process.exit(0);
```

### Python
```python
import asyncio
import sys
from copilot import CopilotClient, PermissionHandler
from copilot.generated.session_events import SessionEventType

async def main():
    client = CopilotClient()
    await client.start()

    session = await client.create_session({
        "on_permission_request": PermissionHandler.approve_all,
        "model": "gpt-4.1",
        "streaming": True,
    })

    def handle_event(event):
        if event.type == SessionEventType.ASSISTANT_MESSAGE_DELTA:
            sys.stdout.write(event.data.delta_content)
            sys.stdout.flush()
        if event.type == SessionEventType.SESSION_IDLE:
            print()

    session.on(handle_event)
    await session.send_and_wait({"prompt": "Tell me a short joke"})
    await client.stop()

asyncio.run(main())
```

### Go
```go
session, err := client.CreateSession(&copilot.SessionConfig{
	OnPermissionRequest: copilot.PermissionHandler.ApproveAll,
    Model:     "gpt-4.1",
    Streaming: true,
})

session.On(func(event copilot.SessionEvent) {
    if event.Type == "assistant.message_delta" {
        fmt.Print(*event.Data.DeltaContent)
    }
    if event.Type == "session.idle" {
        fmt.Println()
    }
})

_, err = session.SendAndWait(copilot.MessageOptions{Prompt: "Tell me a short joke"}, 0)
```

### .NET
```csharp
await using var session = await client.CreateSessionAsync(new SessionConfig
{
    OnPermissionRequest = PermissionHandler.ApproveAll,
    Model = "gpt-4.1",
    Streaming = true,
});

session.On(ev =>
{
    if (ev is AssistantMessageDeltaEvent deltaEvent)
        Console.Write(deltaEvent.Data.DeltaContent);
    if (ev is SessionIdleEvent)
        Console.WriteLine();
});

await session.SendAndWaitAsync(new MessageOptions { Prompt = "Tell me a short joke" });
```

## カスタムツール

推論中に Copilot が呼び出せるツールを定義します。ツールを定義すると、Copilot に次の内容を伝えることになります:
1. **ツールが何をするか**（description）
2. **必要なパラメータ**（schema）
3. **実行するコード**（handler）

### TypeScript (JSON Schema)
```typescript
import { CopilotClient, approveAll, defineTool, SessionEvent } from "@github/copilot-sdk";

const getWeather = defineTool("get_weather", {
    description: "Get the current weather for a city",
    parameters: {
        type: "object",
        properties: {
            city: { type: "string", description: "The city name" },
        },
        required: ["city"],
    },
    handler: async (args: { city: string }) => {
        const { city } = args;
        // 実際のアプリではここで天気 API を呼び出します
        const conditions = ["sunny", "cloudy", "rainy", "partly cloudy"];
        const temp = Math.floor(Math.random() * 30) + 50;
        const condition = conditions[Math.floor(Math.random() * conditions.length)];
        return { city, temperature: `${temp}°F`, condition };
    },
});

const client = new CopilotClient();
const session = await client.createSession({
    onPermissionRequest: approveAll,
    model: "gpt-4.1",
    streaming: true,
    tools: [getWeather],
});

session.on((event: SessionEvent) => {
    if (event.type === "assistant.message_delta") {
        process.stdout.write(event.data.deltaContent);
    }
});

await session.sendAndWait({
    prompt: "What's the weather like in Seattle and Tokyo?",
});

await client.stop();
process.exit(0);
```

### Python (Pydantic)
```python
import asyncio
import random
import sys
from copilot import CopilotClient, PermissionHandler
from copilot.tools import define_tool
from copilot.generated.session_events import SessionEventType
from pydantic import BaseModel, Field

class GetWeatherParams(BaseModel):
    city: str = Field(description="The name of the city to get weather for")

@define_tool(description="Get the current weather for a city")
async def get_weather(params: GetWeatherParams) -> dict:
    city = params.city
    conditions = ["sunny", "cloudy", "rainy", "partly cloudy"]
    temp = random.randint(50, 80)
    condition = random.choice(conditions)
    return {"city": city, "temperature": f"{temp}°F", "condition": condition}

async def main():
    client = CopilotClient()
    await client.start()

    session = await client.create_session({
        "on_permission_request": PermissionHandler.approve_all,
        "model": "gpt-4.1",
        "streaming": True,
        "tools": [get_weather],
    })

    def handle_event(event):
        if event.type == SessionEventType.ASSISTANT_MESSAGE_DELTA:
            sys.stdout.write(event.data.delta_content)
            sys.stdout.flush()

    session.on(handle_event)

    await session.send_and_wait({
        "prompt": "What's the weather like in Seattle and Tokyo?"
    })

    await client.stop()

asyncio.run(main())
```

### Go
```go
type WeatherParams struct {
    City string `json:"city" jsonschema:"The city name"`
}

type WeatherResult struct {
    City        string `json:"city"`
    Temperature string `json:"temperature"`
    Condition   string `json:"condition"`
}

getWeather := copilot.DefineTool(
    "get_weather",
    "Get the current weather for a city",
    func(params WeatherParams, inv copilot.ToolInvocation) (WeatherResult, error) {
        conditions := []string{"sunny", "cloudy", "rainy", "partly cloudy"}
        temp := rand.Intn(30) + 50
        condition := conditions[rand.Intn(len(conditions))]
        return WeatherResult{
            City:        params.City,
            Temperature: fmt.Sprintf("%d°F", temp),
            Condition:   condition,
        }, nil
    },
)

session, _ := client.CreateSession(&copilot.SessionConfig{
	OnPermissionRequest: copilot.PermissionHandler.ApproveAll,
    Model:     "gpt-4.1",
    Streaming: true,
    Tools:     []copilot.Tool{getWeather},
})
```

### .NET (Microsoft.Extensions.AI)
```csharp
using GitHub.Copilot.SDK;
using Microsoft.Extensions.AI;
using System.ComponentModel;

var getWeather = AIFunctionFactory.Create(
    ([Description("The city name")] string city) =>
    {
        var conditions = new[] { "sunny", "cloudy", "rainy", "partly cloudy" };
        var temp = Random.Shared.Next(50, 80);
        var condition = conditions[Random.Shared.Next(conditions.Length)];
        return new { city, temperature = $"{temp}°F", condition };
    },
    "get_weather",
    "Get the current weather for a city"
);

await using var session = await client.CreateSessionAsync(new SessionConfig
{
    OnPermissionRequest = PermissionHandler.ApproveAll,
    Model = "gpt-4.1",
    Streaming = true,
    Tools = [getWeather],
});
```

## ツールの仕組み

Copilot があなたのツールを呼び出すと判断したとき:
1. Copilot がパラメータ付きでツール呼び出しリクエストを送信
2. SDK があなたの handler 関数を実行
3. 結果が Copilot に返送される
4. Copilot がその結果を応答に反映する

Copilot は、ユーザーの質問とツールの description に基づいて、ツールを呼び出すタイミングを判断します。

## 対話型 CLI アシスタント

完全な対話型アシスタントを構築します:

### TypeScript
```typescript
import { CopilotClient, approveAll, defineTool, SessionEvent } from "@github/copilot-sdk";
import * as readline from "readline";

const getWeather = defineTool("get_weather", {
    description: "Get the current weather for a city",
    parameters: {
        type: "object",
        properties: {
            city: { type: "string", description: "The city name" },
        },
        required: ["city"],
    },
    handler: async ({ city }) => {
        const conditions = ["sunny", "cloudy", "rainy", "partly cloudy"];
        const temp = Math.floor(Math.random() * 30) + 50;
        const condition = conditions[Math.floor(Math.random() * conditions.length)];
        return { city, temperature: `${temp}°F`, condition };
    },
});

const client = new CopilotClient();
const session = await client.createSession({
    onPermissionRequest: approveAll,
    model: "gpt-4.1",
    streaming: true,
    tools: [getWeather],
});

session.on((event: SessionEvent) => {
    if (event.type === "assistant.message_delta") {
        process.stdout.write(event.data.deltaContent);
    }
});

const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout,
});

console.log("Weather Assistant (type 'exit' to quit)");
console.log("Try: 'What's the weather in Paris?'\n");

const prompt = () => {
    rl.question("You: ", async (input) => {
        if (input.toLowerCase() === "exit") {
            await client.stop();
            rl.close();
            return;
        }

        process.stdout.write("Assistant: ");
        await session.sendAndWait({ prompt: input });
        console.log("\n");
        prompt();
    });
};

prompt();
```

### Python
```python
import asyncio
import random
import sys
from copilot import CopilotClient, PermissionHandler
from copilot.tools import define_tool
from copilot.generated.session_events import SessionEventType
from pydantic import BaseModel, Field

class GetWeatherParams(BaseModel):
    city: str = Field(description="The name of the city to get weather for")

@define_tool(description="Get the current weather for a city")
async def get_weather(params: GetWeatherParams) -> dict:
    conditions = ["sunny", "cloudy", "rainy", "partly cloudy"]
    temp = random.randint(50, 80)
    condition = random.choice(conditions)
    return {"city": params.city, "temperature": f"{temp}°F", "condition": condition}

async def main():
    client = CopilotClient()
    await client.start()

    session = await client.create_session({
        "on_permission_request": PermissionHandler.approve_all,
        "model": "gpt-4.1",
        "streaming": True,
        "tools": [get_weather],
    })

    def handle_event(event):
        if event.type == SessionEventType.ASSISTANT_MESSAGE_DELTA:
            sys.stdout.write(event.data.delta_content)
            sys.stdout.flush()

    session.on(handle_event)

    print("Weather Assistant (type 'exit' to quit)")
    print("Try: 'What's the weather in Paris?'\n")

    while True:
        try:
            user_input = input("You: ")
        except EOFError:
            break

        if user_input.lower() == "exit":
            break

        sys.stdout.write("Assistant: ")
        await session.send_and_wait({"prompt": user_input})
        print("\n")

    await client.stop()

asyncio.run(main())
```

## MCP サーバー連携

事前構築済みツールのために MCP（Model Context Protocol）サーバーに接続します。リポジトリ、Issue、PR へアクセスするために GitHub の MCP サーバーへ接続できます:

### TypeScript
```typescript
const session = await client.createSession({
    onPermissionRequest: approveAll,
    model: "gpt-4.1",
    mcpServers: {
        github: {
            type: "http",
            url: "https://api.githubcopilot.com/mcp/",
        },
    },
});
```

### Python
```python
session = await client.create_session({
    "on_permission_request": PermissionHandler.approve_all,
    "model": "gpt-4.1",
    "mcp_servers": {
        "github": {
            "type": "http",
            "url": "https://api.githubcopilot.com/mcp/",
        },
    },
})
```

### Go
```go
session, _ := client.CreateSession(&copilot.SessionConfig{
	OnPermissionRequest: copilot.PermissionHandler.ApproveAll,
    Model: "gpt-4.1",
    MCPServers: map[string]copilot.MCPServerConfig{
        "github": {
            "type": "http",
            "url": "https://api.githubcopilot.com/mcp/",
        },
    },
})
```

### .NET
```csharp
await using var session = await client.CreateSessionAsync(new SessionConfig
{
    OnPermissionRequest = PermissionHandler.ApproveAll,
    Model = "gpt-4.1",
    McpServers = new Dictionary<string, McpServerConfig>
    {
        ["github"] = new McpServerConfig
        {
            Type = "http",
            Url = "https://api.githubcopilot.com/mcp/",
        },
    },
});
```

## カスタムエージェント

特定タスク向けの専門的な AI ペルソナを定義します:

### TypeScript
```typescript
const session = await client.createSession({
    onPermissionRequest: approveAll,
    model: "gpt-4.1",
    customAgents: [{
        name: "pr-reviewer",
        displayName: "PR Reviewer",
        description: "Reviews pull requests for best practices",
        prompt: "You are an expert code reviewer. Focus on security, performance, and maintainability.",
    }],
});
```

### Python
```python
session = await client.create_session({
    "on_permission_request": PermissionHandler.approve_all,
    "model": "gpt-4.1",
    "custom_agents": [{
        "name": "pr-reviewer",
        "display_name": "PR Reviewer",
        "description": "Reviews pull requests for best practices",
        "prompt": "You are an expert code reviewer. Focus on security, performance, and maintainability.",
    }],
})
```

## システムメッセージ

AI の振る舞いと個性をカスタマイズします:

### TypeScript
```typescript
const session = await client.createSession({
    onPermissionRequest: approveAll,
    model: "gpt-4.1",
    systemMessage: {
        content: "You are a helpful assistant for our engineering team. Always be concise.",
    },
});
```

### Python
```python
session = await client.create_session({
    "on_permission_request": PermissionHandler.approve_all,
    "model": "gpt-4.1",
    "system_message": {
        "content": "You are a helpful assistant for our engineering team. Always be concise.",
    },
})
```

## 外部 CLI サーバー

CLI を別プロセスでサーバーモード実行し、SDK から接続します。デバッグ、リソース共有、カスタム環境で有用です。

### サーバーモードで CLI を起動
```bash
copilot --server --port 4321
```

### SDK を外部サーバーに接続

#### TypeScript
```typescript
const client = new CopilotClient({
    cliUrl: "localhost:4321"
});

const session = await client.createSession({
    onPermissionRequest: approveAll,
    model: "gpt-4.1",
});
```

#### Python
```python
client = CopilotClient({
    "cli_url": "localhost:4321"
})
await client.start()

session = await client.create_session({
    "on_permission_request": PermissionHandler.approve_all,
    "model": "gpt-4.1",
})
```

#### Go
```go
client := copilot.NewClient(&copilot.ClientOptions{
    CLIUrl: "localhost:4321",
})

if err := client.Start(); err != nil {
    log.Fatal(err)
}

session, _ := client.CreateSession(&copilot.SessionConfig{
	OnPermissionRequest: copilot.PermissionHandler.ApproveAll,
	Model:               "gpt-4.1",
})
```

#### .NET
```csharp
using var client = new CopilotClient(new CopilotClientOptions
{
    CliUrl = "localhost:4321"
});

await using var session = await client.CreateSessionAsync(new SessionConfig
{
    OnPermissionRequest = PermissionHandler.ApproveAll,
    Model = "gpt-4.1",
});
```

**注意:** `cliUrl` が指定されている場合、SDK は CLI プロセスを起動・管理しません。既存サーバーへの接続のみを行います。

## イベントタイプ

| Event | 説明 |
|-------|-------------|
| `user.message` | ユーザー入力が追加された |
| `assistant.message` | モデルの完全な応答 |
| `assistant.message_delta` | ストリーミング応答チャンク |
| `assistant.reasoning` | モデルの推論（モデル依存） |
| `assistant.reasoning_delta` | ストリーミング推論チャンク |
| `tool.execution_start` | ツール呼び出し開始 |
| `tool.execution_complete` | ツール実行完了 |
| `session.idle` | アクティブな処理なし |
| `session.error` | エラーが発生 |

## クライアント設定

| Option | 説明 | 既定値 |
|--------|-------------|---------|
| `cliPath` | Copilot CLI 実行ファイルへのパス | System PATH |
| `cliUrl` | 既存サーバーへ接続（例: "localhost:4321"） | None |
| `port` | サーバー通信用ポート | Random |
| `useStdio` | TCP の代わりに stdio トランスポートを使用 | true |
| `logLevel` | ログ出力の詳細度 | "info" |
| `autoStart` | サーバーを自動起動 | true |
| `autoRestart` | クラッシュ時に再起動 | true |
| `cwd` | CLI プロセスの作業ディレクトリ | Inherited |

## セッション設定

| Option | 説明 |
|--------|-------------|
| `model` | 使用する LLM（"gpt-4.1", "claude-sonnet-4.5" など） |
| `sessionId` | カスタムセッション識別子 |
| `tools` | カスタムツール定義 |
| `mcpServers` | MCP サーバー接続 |
| `customAgents` | カスタムエージェントペルソナ |
| `systemMessage` | 既定のシステムプロンプトを上書き |
| `streaming` | 応答チャンクの逐次配信を有効化 |
| `availableTools` | 許可ツールのホワイトリスト |
| `excludedTools` | 無効化ツールのブラックリスト |

## セッション永続化

再起動後も会話を保存・再開します:

### カスタム ID で作成
```typescript
const session = await client.createSession({
    onPermissionRequest: approveAll,
    sessionId: "user-123-conversation",
    model: "gpt-4.1"
});
```

### セッション再開
```typescript
const session = await client.resumeSession("user-123-conversation", { onPermissionRequest: approveAll });
await session.send({ prompt: "What did we discuss earlier?" });
```

### セッション一覧取得と削除
```typescript
const sessions = await client.listSessions();
await client.deleteSession("old-session-id");
```

## エラーハンドリング

```typescript
try {
    const client = new CopilotClient();
    const session = await client.createSession({
        onPermissionRequest: approveAll,
        model: "gpt-4.1",
    });
    const response = await session.sendAndWait(
        { prompt: "Hello!" },
        30000 // ミリ秒単位のタイムアウト
    );
} catch (error) {
    if (error.code === "ENOENT") {
        console.error("Copilot CLI not installed");
    } else if (error.code === "ECONNREFUSED") {
        console.error("Cannot connect to Copilot server");
    } else {
        console.error("Error:", error.message);
    }
} finally {
    await client.stop();
}
```

## グレースフルシャットダウン

```typescript
process.on("SIGINT", async () => {
    console.log("Shutting down...");
    await client.stop();
    process.exit(0);
});
```

## よくあるパターン

### マルチターン会話
```typescript
const session = await client.createSession({
    onPermissionRequest: approveAll,
    model: "gpt-4.1",
});

await session.sendAndWait({ prompt: "My name is Alice" });
await session.sendAndWait({ prompt: "What's my name?" });
// Response: "Your name is Alice"
```

### ファイル添付
```typescript
await session.send({
    prompt: "Analyze this file",
    attachments: [{
        type: "file",
        path: "./data.csv",
        displayName: "Sales Data"
    }]
});
```

### 長時間処理の中断
```typescript
const timeoutId = setTimeout(() => {
    session.abort();
}, 60000);

session.on((event) => {
    if (event.type === "session.idle") {
        clearTimeout(timeoutId);
    }
});
```

## 利用可能モデル

実行時に利用可能なモデルを問い合わせます:

```typescript
const models = await client.getModels();
// Returns: ["gpt-4.1", "gpt-4o", "claude-sonnet-4.5", ...]
```

## ベストプラクティス

1. **常にクリーンアップする**: `try-finally` または `defer` を使って `client.stop()` が必ず呼ばれるようにする
2. **タイムアウトを設定する**: 長時間処理ではタイムアウト付き `sendAndWait` を使う
3. **イベントを処理する**: 堅牢なエラーハンドリングのためにエラーイベントを購読する
4. **ストリーミングを使う**: 長い応答でより良い UX のためにストリーミングを有効化する
5. **セッションを永続化する**: マルチターン会話にはカスタムセッション ID を使う
6. **明確なツールを定義する**: 説明的なツール名と説明文を書く

## アーキテクチャ

```
Your Application
       |
  SDK Client
       | JSON-RPC
  Copilot CLI (server mode)
       |
  GitHub (models, auth)
```

SDK は CLI プロセスのライフサイクルを自動管理します。すべての通信は stdio または TCP 上の JSON-RPC で行われます。

## リソース

- **GitHub Repository**: https://github.com/github/copilot-sdk
- **Getting Started Tutorial**: https://github.com/github/copilot-sdk/blob/main/docs/tutorials/first-app.md
- **GitHub MCP Server**: https://github.com/github/github-mcp-server
- **MCP Servers Directory**: https://github.com/modelcontextprotocol/servers
- **Cookbook**: https://github.com/github/copilot-sdk/tree/main/cookbook
- **Samples**: https://github.com/github/copilot-sdk/tree/main/samples

## ステータス

この SDK は **Technical Preview** 段階であり、破壊的変更が入る可能性があります。現時点では本番利用は推奨されません。
