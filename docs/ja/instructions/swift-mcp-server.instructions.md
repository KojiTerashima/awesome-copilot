---
description: '公式 MCP Swift SDK package を使って Swift で Model Context Protocol (MCP) server を構築するためのベストプラクティスとパターン。'
applyTo: "**/*.swift, **/Package.swift, **/Package.resolved"
---

# Swift MCP Server 開発ガイドライン

Swift で MCP server を構築する場合は、公式 Swift SDK を使い、以下のベストプラクティスとパターンに従うこと。

## Server 設定

capabilities を持つ `Server` class で MCP server を作成する:

```swift
import MCP

let server = Server(
    name: "MyServer",
    version: "1.0.0",
    capabilities: .init(
        prompts: .init(listChanged: true),
        resources: .init(subscribe: true, listChanged: true),
        tools: .init(listChanged: true)
    )
)
```

## Tool の追加

tool handler の登録には `withMethodHandler` を使う:

```swift
// tool list handler を登録
await server.withMethodHandler(ListTools.self) { _ in
    let tools = [
        Tool(
            name: "search",
            description: "情報を検索する",
            inputSchema: .object([
                "properties": .object([
                    "query": .string("検索クエリ"),
                    "limit": .number("最大件数")
                ]),
                "required": .array([.string("query")])
            ])
        )
    ]
    return .init(tools: tools)
}

// tool call handler を登録
await server.withMethodHandler(CallTool.self) { params in
    switch params.name {
    case "search":
        let query = params.arguments?["query"]?.stringValue ?? ""
        let limit = params.arguments?["limit"]?.intValue ?? 10

        // 検索を実行
        let results = performSearch(query: query, limit: limit)

        return .init(
            content: [.text("\(results.count) 件見つかりました")],
            isError: false
        )

    default:
        return .init(
            content: [.text("不明な tool です")],
            isError: true
        )
    }
}
```

## Resource の追加

data access 用の resource handler を実装する:

```swift
// resource list handler を登録
await server.withMethodHandler(ListResources.self) { params in
    let resources = [
        Resource(
            name: "Data File",
            uri: "resource://data/example.txt",
            description: "サンプル data file",
            mimeType: "text/plain"
        )
    ]
    return .init(resources: resources, nextCursor: nil)
}

// resource read handler を登録
await server.withMethodHandler(ReadResource.self) { params in
    switch params.uri {
    case "resource://data/example.txt":
        let content = loadResourceContent(uri: params.uri)
        return .init(contents: [
            Resource.Content.text(
                content,
                uri: params.uri,
                mimeType: "text/plain"
            )
        ])

    default:
        throw MCPError.invalidParams("Unknown resource URI: \(params.uri)")
    }
}

// resource subscribe handler を登録
await server.withMethodHandler(ResourceSubscribe.self) { params in
    // notification 用に subscription を追跡
    subscriptions.insert(params.uri)
    print("Client subscribed to \(params.uri)")
    return .init()
}
```

## Prompt の追加

template 化された会話のための prompt handler を実装する:

```swift
// prompt list handler を登録
await server.withMethodHandler(ListPrompts.self) { params in
    let prompts = [
        Prompt(
            name: "analyze",
            description: "トピックを分析する",
            arguments: [
                .init(name: "topic", description: "分析対象のトピック", required: true),
                .init(name: "depth", description: "分析の深さ", required: false)
            ]
        )
    ]
    return .init(prompts: prompts, nextCursor: nil)
}

// prompt get handler を登録
await server.withMethodHandler(GetPrompt.self) { params in
    switch params.name {
    case "analyze":
        let topic = params.arguments?["topic"]?.stringValue ?? "general"
        let depth = params.arguments?["depth"]?.stringValue ?? "basic"

        let description = "\(topic) を \(depth) レベルで分析"
        let messages: [Prompt.Message] = [
            .user("Please analyze this topic: \(topic)"),
            .assistant("I'll provide a \(depth) analysis of \(topic)")
        ]

        return .init(description: description, messages: messages)

    default:
        throw MCPError.invalidParams("Unknown prompt: \(params.name)")
    }
}
```

## Transport 設定

### Stdio Transport

local subprocess 通信用:

```swift
import MCP
import Logging

let logger = Logger(label: "com.example.mcp-server")
let transport = StdioTransport(logger: logger)

try await server.start(transport: transport)
```

### HTTP Transport (client 側)

remote server 接続用:

```swift
let transport = HTTPClientTransport(
    endpoint: URL(string: "http://localhost:8080")!,
    streaming: true  // Server-Sent Events を有効化
)

try await client.connect(transport: transport)
```

## Concurrency と Actor

server は actor なので、thread-safe にアクセスできる:

```swift
actor ServerState {
    private var subscriptions: Set<String> = []
    private var cache: [String: Any] = [:]

    func addSubscription(_ uri: String) {
        subscriptions.insert(uri)
    }

    func getSubscriptions() -> Set<String> {
        return subscriptions
    }
}

let state = ServerState()

await server.withMethodHandler(ResourceSubscribe.self) { params in
    await state.addSubscription(params.uri)
    return .init()
}
```

## Error Handling

`MCPError` と Swift の error handling を使う:

```swift
await server.withMethodHandler(CallTool.self) { params in
    do {
        guard let query = params.arguments?["query"]?.stringValue else {
            throw MCPError.invalidParams("query parameter がありません")
        }

        let result = try performOperation(query: query)

        return .init(
            content: [.text(result)],
            isError: false
        )
    } catch let error as MCPError {
        return .init(
            content: [.text(error.localizedDescription)],
            isError: true
        )
    } catch {
        return .init(
            content: [.text("想定外の error: \(error.localizedDescription)")],
            isError: true
        )
    }
}
```

## Value 型による JSON Schema

JSON schema には `Value` 型を使う:

```swift
let schema = Value.object([
    "type": .string("object"),
    "properties": .object([
        "name": .object([
            "type": .string("string"),
            "description": .string("ユーザー名")
        ]),
        "age": .object([
            "type": .string("integer"),
            "minimum": .number(0),
            "maximum": .number(150)
        ]),
        "email": .object([
            "type": .string("string"),
            "format": .string("email")
        ])
    ]),
    "required": .array([.string("name")])
])
```

## Swift Package Manager 設定

`Package.swift` は次のように作成する:

```swift
// swift-tools-version: 6.0
import PackageDescription

let package = Package(
    name: "MyMCPServer",
    platforms: [
        .macOS(.v13),
        .iOS(.v16)
    ],
    dependencies: [
        .package(
            url: "https://github.com/modelcontextprotocol/swift-sdk.git",
            from: "0.10.0"
        ),
        .package(
            url: "https://github.com/apple/swift-log.git",
            from: "1.5.0"
        )
    ],
    targets: [
        .executableTarget(
            name: "MyMCPServer",
            dependencies: [
                .product(name: "MCP", package: "swift-sdk"),
                .product(name: "Logging", package: "swift-log")
            ]
        )
    ]
)
```

## ServiceLifecycle を使った graceful shutdown

適切な shutdown には Swift Service Lifecycle を使う:

```swift
import MCP
import ServiceLifecycle
import Logging

struct MCPService: Service {
    let server: Server
    let transport: Transport

    func run() async throws {
        try await server.start(transport: transport)
        try await Task.sleep(for: .days(365 * 100))
    }

    func shutdown() async throws {
        await server.stop()
    }
}

let logger = Logger(label: "com.example.mcp-server")
let transport = StdioTransport(logger: logger)
let mcpService = MCPService(server: server, transport: transport)

let serviceGroup = ServiceGroup(
    services: [mcpService],
    configuration: .init(
        gracefulShutdownSignals: [.sigterm, .sigint]
    ),
    logger: logger
)

try await serviceGroup.run()
```

## Async/Await パターン

server 操作はすべて Swift concurrency を使う:

```swift
await server.withMethodHandler(CallTool.self) { params in
    async let result1 = fetchData1()
    async let result2 = fetchData2()

    let combined = await "\(result1) and \(result2)"

    return .init(
        content: [.text(combined)],
        isError: false
    )
}
```

## Logging

構造化 logging には swift-log を使う:

```swift
import Logging

let logger = Logger(label: "com.example.mcp-server")

await server.withMethodHandler(CallTool.self) { params in
    logger.info("Tool called", metadata: [
        "name": .string(params.name),
        "args": .string("\(params.arguments ?? [:])")
    ])

    // tool call を処理

    logger.debug("Tool completed successfully")

    return .init(content: [.text("Result")], isError: false)
}
```

## テスト

server は async / await でテストする:

```swift
import XCTest
@testable import MyMCPServer

final class ServerTests: XCTestCase {
    func testToolCall() async throws {
        let server = createTestServer()

        // tool call logic をテスト
        let params = CallTool.Params(
            name: "search",
            arguments: ["query": .string("test")]
        )

        // 振る舞いを検証
        XCTAssertNoThrow(try await processToolCall(params))
    }
}
```

## Initialize Hook

initialize hook で client connection を検証する:

```swift
try await server.start(transport: transport) { clientInfo, clientCapabilities in
    // client を検証
    guard clientInfo.name != "BlockedClient" else {
        throw MCPError.invalidRequest("Client not allowed")
    }

    // capability を確認
    if clientCapabilities.sampling == nil {
        logger.warning("Client doesn't support sampling")
    }

    logger.info("Client connected", metadata: [
        "name": .string(clientInfo.name),
        "version": .string(clientInfo.version)
    ])
}
```

## よくあるパターン

### Content Type

異なる content type を扱う:

```swift
return .init(
    content: [
        .text("Plain text response"),
        .image(imageData, mimeType: "image/png", metadata: [
            "width": 1024,
            "height": 768
        ]),
        .resource(
            uri: "resource://data",
            mimeType: "application/json",
            text: jsonString
        )
    ],
    isError: false
)
```

### Strict 設定

missing capability で即失敗させる strict mode を使う:

```swift
let client = Client(
    name: "StrictClient",
    version: "1.0.0",
    configuration: .strict
)

// capability が利用できなければ即座に throw する
try await client.listTools()
```

### Request Batching

複数 request を効率的に送る:

```swift
var tasks: [Task<CallTool.Result, Error>] = []

try await client.withBatch { batch in
    for i in 0..<10 {
        tasks.append(
            try await batch.addRequest(
                CallTool.request(.init(
                    name: "process",
                    arguments: ["id": .number(Double(i))]
                ))
            )
        )
    }
}

for (index, task) in tasks.enumerated() {
    let result = try await task.value
    print("\(index): \(result.content)")
}
```
