---
description: 'TypeScript SDK を使って Model Context Protocol (MCP) server を構築するための指示'
applyTo: '**/*.ts, **/*.js, **/package.json'
---

# TypeScript MCP Server 開発

## 指示

- **@modelcontextprotocol/sdk** npm package を使う: `npm install @modelcontextprotocol/sdk`
- 特定 path から import する: `@modelcontextprotocol/sdk/server/mcp.js`、`@modelcontextprotocol/sdk/server/stdio.js` など
- protocol handling を自動化した高レベル server 実装には `McpServer` class を使う
- request handler を手動で扱う低レベル制御には `Server` class を使う
- 入出力 schema validation には **zod** を使う: `npm install zod@3`
- UI 表示向上のため、tool、resource、prompt には常に `title` field を提供する
- `registerTool()`、`registerResource()`、`registerPrompt()` method を使う (古い API より推奨)
- schema は zod で定義する: `{ inputSchema: { param: z.string() }, outputSchema: { result: z.string() } }`
- tool からは `content` (表示用) と `structuredContent` (構造化 data 用) の両方を返す
- HTTP server では、Express などの framework とともに `StreamableHTTPServerTransport` を使う
- local integration では、stdio ベース通信に `StdioServerTransport` を使う
- request ID collision を防ぐため、request ごとに新しい transport instance を作る (stateless mode)
- stateful server では `sessionIdGenerator` を使った session 管理を行う
- local server では DNS rebinding protection を有効にする: `enableDnsRebindingProtection: true`
- browser ベース client 向けには CORS header を設定し、`Mcp-Session-Id` を expose する
- URI parameter を持つ dynamic resource には `ResourceTemplate` を使う: `new ResourceTemplate('resource://{param}', { list: undefined })`
- より良い UX のため completions をサポートするには `@modelcontextprotocol/sdk/server/completable.js` の `completable()` wrapper を使う
- client に LLM completion を要求する sampling には `server.server.createMessage()` を実装する
- tool 実行中に追加 user input を要求するには `server.server.elicitInput()` を使う
- 一括更新には notification debouncing を有効にする: `debouncedNotificationMethods: ['notifications/tools/list_changed']`
- dynamic update: 登録済み項目に対して `.enable()`、`.disable()`、`.update()`、`.remove()` を呼び、`listChanged` notification を発行する
- UI 表示名には `@modelcontextprotocol/sdk/shared/metadataUtils.js` の `getDisplayName()` を使う
- MCP Inspector で server をテストする: `npx @modelcontextprotocol/inspector`

## ベストプラクティス

- tool 実装は単一責務に集中させる
- LLM が理解しやすいよう、明確で説明的な title と description を付ける
- すべての parameter と return value に適切な TypeScript 型を使う
- try-catch block による包括的な error handling を実装する
- error 条件では tool result に `isError: true` を返す
- すべての非同期処理に async / await を使う
- database connection を閉じ、resource を適切に cleanup する
- 処理前に input parameter を検証する
- stdout / stderr を汚さない structured logging を debugging に使う
- file system や network access を公開する場合は security 影響を考慮する
- transport の close event で適切な resource cleanup を実装する
- 設定 (port、API key など) には environment variable を使う
- tool の capability と limitation を明確に document する
- 複数 client でテストして互換性を確認する

## よくあるパターン

### 基本 server 設定 (HTTP)
```typescript
import { McpServer } from '@modelcontextprotocol/sdk/server/mcp.js';
import { StreamableHTTPServerTransport } from '@modelcontextprotocol/sdk/server/streamableHttp.js';
import express from 'express';

const server = new McpServer({
    name: 'my-server',
    version: '1.0.0'
});

const app = express();
app.use(express.json());

app.post('/mcp', async (req, res) => {
    const transport = new StreamableHTTPServerTransport({
        sessionIdGenerator: undefined,
        enableJsonResponse: true
    });

    res.on('close', () => transport.close());

    await server.connect(transport);
    await transport.handleRequest(req, res, req.body);
});

app.listen(3000);
```

### 基本 server 設定 (stdio)
```typescript
import { McpServer } from '@modelcontextprotocol/sdk/server/mcp.js';
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js';

const server = new McpServer({
    name: 'my-server',
    version: '1.0.0'
});

// ... tool、resource、prompt を登録 ...

const transport = new StdioServerTransport();
await server.connect(transport);
```

### 単純な tool
```typescript
import { z } from 'zod';

server.registerTool(
    'calculate',
    {
        title: 'Calculator',
        description: '基本的な計算を行う',
        inputSchema: { a: z.number(), b: z.number(), op: z.enum(['+', '-', '*', '/']) },
        outputSchema: { result: z.number() }
    },
    async ({ a, b, op }) => {
        const result = op === '+' ? a + b : op === '-' ? a - b :
                      op === '*' ? a * b : a / b;
        const output = { result };
        return {
            content: [{ type: 'text', text: JSON.stringify(output) }],
            structuredContent: output
        };
    }
);
```

### Dynamic Resource
```typescript
import { ResourceTemplate } from '@modelcontextprotocol/sdk/server/mcp.js';

server.registerResource(
    'user',
    new ResourceTemplate('users://{userId}', { list: undefined }),
    {
        title: 'User Profile',
        description: 'ユーザープロファイル data を取得する'
    },
    async (uri, { userId }) => ({
        contents: [{
            uri: uri.href,
            text: `User ${userId} data here`
        }]
    })
);
```

### Sampling を伴う tool
```typescript
server.registerTool(
    'summarize',
    {
        title: 'Text Summarizer',
        description: 'LLM を使って text を要約する',
        inputSchema: { text: z.string() },
        outputSchema: { summary: z.string() }
    },
    async ({ text }) => {
        const response = await server.server.createMessage({
            messages: [{
                role: 'user',
                content: { type: 'text', text: `Summarize: ${text}` }
            }],
            maxTokens: 500
        });

        const summary = response.content.type === 'text' ?
            response.content.text : '要約できませんでした';
        const output = { summary };
        return {
            content: [{ type: 'text', text: JSON.stringify(output) }],
            structuredContent: output
        };
    }
);
```

### Completion 付き prompt
```typescript
import { completable } from '@modelcontextprotocol/sdk/server/completable.js';

server.registerPrompt(
    'review',
    {
        title: 'Code Review',
        description: '特定の観点で code を review する',
        argsSchema: {
            language: completable(z.string(), value =>
                ['typescript', 'python', 'javascript', 'java']
                    .filter(l => l.startsWith(value))
            ),
            code: z.string()
        }
    },
    ({ language, code }) => ({
        messages: [{
            role: 'user',
            content: {
                type: 'text',
                text: `Review this ${language} code:\n\n${code}`
            }
        }]
    })
);
```

### Error Handling
```typescript
server.registerTool(
    'risky-operation',
    {
        title: 'Risky Operation',
        description: '失敗する可能性がある処理',
        inputSchema: { input: z.string() },
        outputSchema: { result: z.string() }
    },
    async ({ input }) => {
        try {
            const result = await performRiskyOperation(input);
            const output = { result };
            return {
                content: [{ type: 'text', text: JSON.stringify(output) }],
                structuredContent: output
            };
        } catch (err: unknown) {
            const error = err as Error;
            return {
                content: [{ type: 'text', text: `Error: ${error.message}` }],
                isError: true
            };
        }
    }
);
```
