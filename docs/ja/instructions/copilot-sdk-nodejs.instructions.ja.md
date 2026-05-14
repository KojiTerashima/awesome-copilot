---
applyTo: "**.ts, **.js, package.json"
description: "このファイルは、GitHub Copilot SDK を使用して Node.js/TypeScript アプリケーションを構築するためのガイダンスを提供します。"
name: "GitHub Copilot SDK Node.js Instructions"
---

## 基本原則

- SDK はテクニカルプレビュー段階にあり、重大な変更が含まれる可能性があります
- Node.js 18.0以降が必要です
- GitHub Copilot CLI がインストールされ、PATH に含まれている必要があります
- タイプセーフのために TypeScript で構築されています
- 全体的に非同期/待機パターンを使用します
- 完全な TypeScript 型定義を提供します

## インストール

常に npm/pnpm/yarn 経由でインストールします。

```bash
npm install @github/copilot-sdk
# or
pnpm add @github/copilot-sdk
# or
yarn add @github/copilot-sdk
```

## クライアントの初期化

### 基本的なクライアントのセットアップ

```typescript
import { CopilotClient, approveAll } from "@github/copilot-sdk";

const client = new CopilotClient();
await client.start();
// Use client...
await client.stop();
```

### クライアント構成オプション

CopilotClient を作成するときは、`CopilotClientOptions` を使用します。

- `cliPath` - CLI 実行可能ファイルへのパス (デフォルト: PATH からの「copilot」)
- `cliArgs` - SDK 管理フラグの前に付加される追加の引数 (string[])
- `cliUrl` - 既存の CLI サーバーの URL (例: "localhost:8080")。提供された場合、クライアントはプロセスを生成しません
- `port` - サーバーポート (デフォルト: ランダムの場合は 0)
- `useStdio` - TCP の代わりに stdio トランスポートを使用します (デフォルト: true)
- `logLevel` - ログレベル (デフォルト: "デバッグ")
- `autoStart` - サーバーの自動起動 (デフォルト: true)
- `autoRestart` - クラッシュ時の自動再起動 (デフォルト: true)
- `cwd` - CLI プロセスの作業ディレクトリ (デフォルト: process.cwd())
- `env` - CLI プロセスの環境変数 (デフォルト: process.env)

### 手動サーバー制御

明示的な制御の場合:

```typescript
const client = new CopilotClient({ autoStart: false });
await client.start();
// Use client...
await client.stop();
```

`stop()` に時間がかかりすぎる場合は、`forceStop()` を使用してください。

## セッション管理

### セッションの作成

設定には `SessionConfig` を使用します。

```typescript
const session = await client.createSession({
    onPermissionRequest: approveAll,
    model: "gpt-5",
    streaming: true,
    tools: [...],
    systemMessage: { ... },
    availableTools: ["tool1", "tool2"],
    excludedTools: ["tool3"],
    provider: { ... }
});
```

### セッション構成オプション

- `sessionId` - カスタムセッションID（文字列）
- `model` - モデル名 (「gpt-5」、「claude-sonnet-4.5」など)
- `tools` - CLI に公開されるカスタムツール (Tool[])
- `systemMessage` - システムメッセージのカスタマイズ (SystemMessageConfig)
- `availableTools` - ツール名の許可リスト (string[])
- `excludedTools` - ツール名のブロックリスト (string[])
- `provider` - カスタム API プロバイダー構成 (BYOK) (ProviderConfig)
- `streaming` - ストリーミング応答チャンクを有効にする (ブール値)
- `mcpServers` - MCP サーバー構成 (MCPServerConfig[])
- `customAgents` - カスタムエージェント構成 (CustomAgentConfig[])
- `configDir` - 構成ディレクトリのオーバーライド (文字列)
- `skillDirectories` - スキルディレクトリ (string[])
- `disabledSkills` - 無効化されたスキル (string[])
- `onPermissionRequest` - 許可要求ハンドラー (PermissionHandler)

### セッションの再開

```typescript
const session = await client.resumeSession("session-id", {
  tools: [myNewTool],
  onPermissionRequest: approveAll,
});
```

### セッション操作

- `session.sessionId` - セッション識別子（文字列）を取得します
- `await session.send({ prompt: "...", attachments: [...] })` - メッセージを送信し、Promise<string> を返します
- `await session.sendAndWait({ prompt: "..." }, timeout)` - 送信してアイドル状態になるまで待機し、Promise<AssistantMessageEvent | を返します。 null>
- `await session.abort()` - 現在の処理を中止します
- `await session.getMessages()` - すべてのイベント/メッセージを取得し、Promise<SessionEvent[]> を返します。
- `await session.destroy()` - クリーンアップセッション

## イベント処理

### イベントサブスクリプションパターン

セッションイベントの待機には、常に async/await または Promise を使用してください。

```typescript
await new Promise<void>((resolve) => {
  session.on((event) => {
    if (event.type === "assistant.message") {
      console.log(event.data.content);
    } else if (event.type === "session.idle") {
      resolve();
    }
  });

  session.send({ prompt: "..." });
});
```

### イベントの登録解除

`on()` メソッドは、サブスクライブを解除する関数を返します。

```typescript
const unsubscribe = session.on((event) => {
  // handler
});
// Later...
unsubscribe();
```

### イベントの種類

イベント処理にはタイプガードを備えた判別共用体を使用します。

```typescript
session.on((event) => {
  switch (event.type) {
    case "user.message":
      // Handle user message
      break;
    case "assistant.message":
      console.log(event.data.content);
      break;
    case "tool.executionStart":
      // Tool execution started
      break;
    case "tool.executionComplete":
      // Tool execution completed
      break;
    case "session.start":
      // Session started
      break;
    case "session.idle":
      // Session is idle (processing complete)
      break;
    case "session.error":
      console.error(`Error: ${event.data.message}`);
      break;
  }
});
```

## ストリーミング応答

### ストリーミングを有効にする

SessionConfig で `streaming: true` を設定します。

```typescript
const session = await client.createSession({
    onPermissionRequest: approveAll,
    model: "gpt-5",
    streaming: true,
});
```

### ストリーミングイベントの処理

デルタイベント (増分) と最終イベントの両方を処理します。

```typescript
await new Promise<void>((resolve) => {
  session.on((event) => {
    switch (event.type) {
      case "assistant.message_delta":
        // Incremental text chunk
        process.stdout.write(event.data.deltaContent);
        break;
      case "assistant.reasoning_delta":
        // Incremental reasoning chunk (model-dependent)
        process.stdout.write(event.data.deltaContent);
        break;
      case "assistant.message":
        // Final complete message
        console.log("\n--- Final ---");
        console.log(event.data.content);
        break;
      case "assistant.reasoning":
        // Final reasoning content
        console.log("--- Reasoning ---");
        console.log(event.data.content);
        break;
      case "session.idle":
        resolve();
        break;
    }
  });

  session.send({ prompt: "Tell me a story" });
});
```

注: 最終イベント (`assistant.message`、`assistant.reasoning`) は、ストリーミング設定に関係なく常に送信されます。

## カスタムツール

### defineTool を使用したツールの定義

タイプセーフなツール定義には `defineTool` を使用します。

```typescript
import { defineTool } from "@github/copilot-sdk";

const session = await client.createSession({
    onPermissionRequest: approveAll,
    model: "gpt-5",
  tools: [
    defineTool({
      name: "lookup_issue",
      description: "Fetch issue details from tracker",
      parameters: {
        type: "object",
        properties: {
          id: { type: "string", description: "Issue ID" },
        },
        required: ["id"],
      },
      handler: async (args) => {
        const issue = await fetchIssue(args.id);
        return issue;
      },
    }),
  ],
});
```

### パラメータに Zod を使用する

SDK はパラメータの Zod スキーマをサポートしています。

```typescript
import { z } from "zod";

const session = await client.createSession({
    onPermissionRequest: approveAll,
  tools: [
    defineTool({
      name: "get_weather",
      description: "Get weather for a location",
      parameters: z.object({
        location: z.string().describe("City name"),
        units: z.enum(["celsius", "fahrenheit"]).optional(),
      }),
      handler: async (args) => {
        return { temperature: 72, units: args.units || "fahrenheit" };
      },
    }),
  ],
});
```

### ツールの戻り値の型

- 任意の JSON シリアル化可能な値を返します (自動的にラップされます)。
- または、メタデータを完全に制御するには `ToolResultObject` を返します。

```typescript
{
    textResultForLlm: string;  // Result shown to LLM
    resultType: "success" | "failure";
    error?: string;  // Internal error (not shown to LLM)
    toolTelemetry?: Record<string, unknown>;
}
```

### ツールの実行フロー

Copilot がツールを呼び出すと、クライアントは自動的に次のことを行います。

1. ハンドラー関数を実行します
2. 戻り値をシリアル化します
3. CLIに応答します

## システムメッセージのカスタマイズ

### 追加モード (デフォルト - ガードレールを保持)

```typescript
const session = await client.createSession({
    onPermissionRequest: approveAll,
    model: "gpt-5",
  systemMessage: {
    mode: "append",
    content: `
<workflow_rules>
- Always check for security vulnerabilities
- Suggest performance improvements when applicable
</workflow_rules>
`,
  },
});
```

### 置換モード (フルコントロール - ガードレールを削除)

```typescript
const session = await client.createSession({
    onPermissionRequest: approveAll,
    model: "gpt-5",
  systemMessage: {
    mode: "replace",
    content: "You are a helpful assistant.",
  },
});
```

## 添付ファイル

メッセージにファイルを添付します。

```typescript
await session.send({
  prompt: "Analyze this file",
  attachments: [
    {
      type: "file",
      path: "/path/to/file.ts",
      displayName: "My File",
    },
  ],
});
```

## メッセージ配信モード

メッセージオプションで `mode` プロパティを使用します。

- `"enqueue"` - メッセージを処理のためにキューに入れます
- `"immediate"` - メッセージを直ちに処理します

```typescript
await session.send({
  prompt: "...",
  mode: "enqueue",
});
```

## 複数のセッション

セッションは独立しており、同時に実行できます。

```typescript
const session1 = await client.createSession({
    onPermissionRequest: approveAll,
    model: "gpt-5",
});
const session2 = await client.createSession({
    onPermissionRequest: approveAll,
    model: "claude-sonnet-4.5",
});

await Promise.all([
  session1.send({ prompt: "Hello from session 1" }),
  session2.send({ prompt: "Hello from session 2" }),
]);
```

## 自分のキーの持ち込み (BYOK)

`provider` 経由でカスタム API プロバイダーを使用します。

```typescript
const session = await client.createSession({
    onPermissionRequest: approveAll,
  provider: {
    type: "openai",
    baseUrl: "https://api.openai.com/v1",
    apiKey: "your-api-key",
  },
});
```

## セッションのライフサイクル管理

### セッションのリスト表示

```typescript
const sessions = await client.listSessions();
for (const metadata of sessions) {
  console.log(`${metadata.sessionId}: ${metadata.summary}`);
}
```

### セッションの削除

```typescript
await client.deleteSession(sessionId);
```

### 最後のセッションIDの取得

```typescript
const lastId = await client.getLastSessionId();
if (lastId) {
  const session = await client.resumeSession(lastId, { onPermissionRequest: approveAll });
}
```

### 接続状態の確認

```typescript
const state = client.getState();
// Returns: "disconnected" | "connecting" | "connected" | "error"
```

## エラー処理

### 標準例外処理

```typescript
try {
  const session = await client.createSession({ onPermissionRequest: approveAll });
  await session.send({ prompt: "Hello" });
} catch (error) {
  console.error(`Error: ${error.message}`);
}
```

### セッションエラーイベント

`session.error` イベントタイプを監視して実行時エラーを検出します。

```typescript
session.on((event) => {
  if (event.type === "session.error") {
    console.error(`Session Error: ${event.data.message}`);
  }
});
```

## 接続テスト

ping を使用してサーバーの接続を確認します。

```typescript
const response = await client.ping("health check");
console.log(`Server responded at ${new Date(response.timestamp)}`);
```

## リソースのクリーンアップ

### Try-Finally による自動クリーンアップ

常に、finally ブロック内で try-finally または cleanup を使用してください。

```typescript
const client = new CopilotClient();
try {
  await client.start();
  const session = await client.createSession({ onPermissionRequest: approveAll });
  try {
    // Use session...
  } finally {
    await session.destroy();
  }
} finally {
  await client.stop();
}
```

### クリーンアップ関数のパターン

```typescript
async function withClient<T>(
  fn: (client: CopilotClient) => Promise<T>,
): Promise<T> {
  const client = new CopilotClient();
  try {
    await client.start();
    return await fn(client);
  } finally {
    await client.stop();
  }
}

async function withSession<T>(
  client: CopilotClient,
  fn: (session: CopilotSession) => Promise<T>,
): Promise<T> {
  const session = await client.createSession({ onPermissionRequest: approveAll });
  try {
    return await fn(session);
  } finally {
    await session.destroy();
  }
}

// Usage
await withClient(async (client) => {
  await withSession(client, async (session) => {
    await session.send({ prompt: "Hello!" });
  });
});
```

## ベストプラクティス

1. **リソースのクリーンアップには常に try-finally を使用してください**
2. **Promises を使用**して session.idle イベントを待機します
3. **堅牢なエラー処理のために session.error** イベントを処理する
4. **イベント処理にはタイプガードまたは switch ステートメントを使用します**
5. **ストリーミングを有効にする** ことで、インタラクティブなシナリオでの UX を向上させます
6. **タイプセーフなツール定義にはdefineToolを使用してください**
7. **実行時パラメータの検証には Zod スキーマを使用します**
8. **不要になったらイベントサブスクリプションを破棄**
9. **安全ガードレールを維持するには、モード: "append" で systemMessage を使用します**
10. **ストリーミングが有効な場合、デルタイベントと最終イベントの両方を処理します**
11. **TypeScript 型を利用してコンパイル時の安全性を確保**

## よくあるパターン

### 単純なクエリと応答

```typescript
import { CopilotClient, approveAll } from "@github/copilot-sdk";

const client = new CopilotClient();
try {
  await client.start();

  const session = await client.createSession({
    onPermissionRequest: approveAll,
    model: "gpt-5",
  });
  try {
    await new Promise<void>((resolve) => {
      session.on((event) => {
        if (event.type === "assistant.message") {
          console.log(event.data.content);
        } else if (event.type === "session.idle") {
          resolve();
        }
      });

      session.send({ prompt: "What is 2+2?" });
    });
  } finally {
    await session.destroy();
  }
} finally {
  await client.stop();
}
```

### マルチターン会話

```typescript
const session = await client.createSession({ onPermissionRequest: approveAll });

async function sendAndWait(prompt: string): Promise<void> {
  await new Promise<void>((resolve, reject) => {
    const unsubscribe = session.on((event) => {
      if (event.type === "assistant.message") {
        console.log(event.data.content);
      } else if (event.type === "session.idle") {
        unsubscribe();
        resolve();
      } else if (event.type === "session.error") {
        unsubscribe();
        reject(new Error(event.data.message));
      }
    });

    session.send({ prompt });
  });
}

await sendAndWait("What is the capital of France?");
await sendAndWait("What is its population?");
```

### SendAndWait ヘルパー

```typescript
// Use built-in sendAndWait for simpler synchronous interaction
const response = await session.sendAndWait({ prompt: "What is 2+2?" }, 60000);

if (response) {
  console.log(response.data.content);
}
```

### タイプセーフなパラメータを備えたツール

```typescript
import { z } from "zod";
import { defineTool } from "@github/copilot-sdk";

interface UserInfo {
  id: string;
  name: string;
  email: string;
  role: string;
}

const session = await client.createSession({
    onPermissionRequest: approveAll,
  tools: [
    defineTool({
      name: "get_user",
      description: "Retrieve user information",
      parameters: z.object({
        userId: z.string().describe("User ID"),
      }),
      handler: async (args): Promise<UserInfo> => {
        return {
          id: args.userId,
          name: "John Doe",
          email: "john@example.com",
          role: "Developer",
        };
      },
    }),
  ],
});
```

### 進行中のストリーミング

```typescript
let currentMessage = "";

const unsubscribe = session.on((event) => {
  if (event.type === "assistant.message_delta") {
    currentMessage += event.data.deltaContent;
    process.stdout.write(event.data.deltaContent);
  } else if (event.type === "assistant.message") {
    console.log("\n\n=== Complete ===");
    console.log(`Total length: ${event.data.content.length} chars`);
  } else if (event.type === "session.idle") {
    unsubscribe();
  }
});

await session.send({ prompt: "Write a long story" });
```

### エラー回復

```typescript
session.on((event) => {
  if (event.type === "session.error") {
    console.error("Session error:", event.data.message);
    // Optionally retry or handle error
  }
});

try {
  await session.send({ prompt: "risky operation" });
} catch (error) {
  // Handle send errors
  console.error("Failed to send:", error);
}
```

## TypeScript 固有の機能

### 型推論

```typescript
import type { SessionEvent, AssistantMessageEvent } from "@github/copilot-sdk";

session.on((event: SessionEvent) => {
  if (event.type === "assistant.message") {
    // TypeScript knows event is AssistantMessageEvent here
    const content: string = event.data.content;
  }
});
```

### 汎用ヘルパー

```typescript
async function waitForEvent<T extends SessionEvent["type"]>(
  session: CopilotSession,
  eventType: T,
): Promise<Extract<SessionEvent, { type: T }>> {
  return new Promise((resolve) => {
    const unsubscribe = session.on((event) => {
      if (event.type === eventType) {
        unsubscribe();
        resolve(event as Extract<SessionEvent, { type: T }>);
      }
    });
  });
}

// Usage
const message = await waitForEvent(session, "assistant.message");
console.log(message.data.content);
```
