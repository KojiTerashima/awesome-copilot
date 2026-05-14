---
description: '公式 io.modelcontextprotocol:kotlin-sdk library を用いて Kotlin で Model Context Protocol (MCP) server を構築するためのベストプラクティスとパターン。'
applyTo: "**/*.kt, **/*.kts, **/build.gradle.kts, **/settings.gradle.kts"
---

# Kotlin MCP Server 開発ガイドライン

Kotlin で MCP server を構築する際は、公式 Kotlin SDK を使って次のベストプラクティスとパターンに従ってください。

## Server のセットアップ

MCP server の作成には `Server` class を使ってください。

```kotlin
import io.modelcontextprotocol.kotlin.sdk.server.Server
import io.modelcontextprotocol.kotlin.sdk.server.ServerOptions
import io.modelcontextprotocol.kotlin.sdk.Implementation
import io.modelcontextprotocol.kotlin.sdk.ServerCapabilities

val server = Server(
    serverInfo = Implementation(
        name = "my-server",
        version = "1.0.0"
    ),
    options = ServerOptions(
        capabilities = ServerCapabilities(
            tools = ServerCapabilities.Tools(),
            resources = ServerCapabilities.Resources(
                subscribe = true,
                listChanged = true
            ),
            prompts = ServerCapabilities.Prompts(listChanged = true)
        )
    ) {
    "Server description goes here"
}
```

## Tool の追加

型付き request / response 処理を伴う tool 登録には `server.addTool()` を使ってください。

```kotlin
import io.modelcontextprotocol.kotlin.sdk.CallToolRequest
import io.modelcontextprotocol.kotlin.sdk.CallToolResult
import io.modelcontextprotocol.kotlin.sdk.TextContent

server.addTool(
    name = "search",
    description = "Search for information",
    inputSchema = buildJsonObject {
        put("type", "object")
        putJsonObject("properties") {
            putJsonObject("query") {
                put("type", "string")
                put("description", "The search query")
            }
            putJsonObject("limit") {
                put("type", "integer")
                put("description", "Maximum results to return")
            }
        }
        putJsonArray("required") {
            add("query")
        }
    }
) { request: CallToolRequest ->
    val query = request.params.arguments["query"] as? String
        ?: throw IllegalArgumentException("query is required")
    val limit = (request.params.arguments["limit"] as? Number)?.toInt() ?: 10

    // Perform search
    val results = performSearch(query, limit)

    CallToolResult(
        content = listOf(
            TextContent(
                text = results.joinToString("\n")
            )
        )
    )
}
```

## Resource の追加

アクセス可能な data を提供するには `server.addResource()` を使ってください。

```kotlin
import io.modelcontextprotocol.kotlin.sdk.ReadResourceRequest
import io.modelcontextprotocol.kotlin.sdk.ReadResourceResult
import io.modelcontextprotocol.kotlin.sdk.TextResourceContents

server.addResource(
    uri = "file:///data/example.txt",
    name = "Example Data",
    description = "Example resource data",
    mimeType = "text/plain"
) { request: ReadResourceRequest ->
    val content = loadResourceContent(request.uri)

    ReadResourceResult(
        contents = listOf(
            TextResourceContents(
                text = content,
                uri = request.uri,
                mimeType = "text/plain"
            )
        )
    )
}
```

## Prompt の追加

再利用可能な prompt template には `server.addPrompt()` を使ってください。

```kotlin
import io.modelcontextprotocol.kotlin.sdk.GetPromptRequest
import io.modelcontextprotocol.kotlin.sdk.GetPromptResult
import io.modelcontextprotocol.kotlin.sdk.PromptMessage
import io.modelcontextprotocol.kotlin.sdk.Role

server.addPrompt(
    name = "analyze",
    description = "Analyze a topic",
    arguments = listOf(
        PromptArgument(
            name = "topic",
            description = "The topic to analyze",
            required = true
        )
    )
) { request: GetPromptRequest ->
    val topic = request.params.arguments?.get("topic") as? String
        ?: throw IllegalArgumentException("topic is required")

    GetPromptResult(
        description = "Analyze the given topic",
        messages = listOf(
            PromptMessage(
                role = Role.User,
                content = TextContent(
                    text = "Analyze this topic: $topic"
                )
            )
        )
    )
}
```

## Transport 設定

### Stdio Transport

stdin / stdout 経由の通信向け:

```kotlin
import io.modelcontextprotocol.kotlin.sdk.server.StdioServerTransport

suspend fun main() {
    val transport = StdioServerTransport()
    server.connect(transport)
}
```

### SSE Transport with Ktor

Server-Sent Events を使う HTTP ベース通信向け:

```kotlin
import io.ktor.server.application.*
import io.ktor.server.engine.*
import io.ktor.server.netty.*
import io.modelcontextprotocol.kotlin.sdk.server.mcp

fun main() {
    embeddedServer(Netty, port = 8080) {
        mcp {
            Server(
                serverInfo = Implementation(
                    name = "sse-server",
                    version = "1.0.0"
                ),
                options = ServerOptions(
                    capabilities = ServerCapabilities(
                        tools = ServerCapabilities.Tools()
                    )
                )
            ) {
                "SSE-based MCP server"
            }
        }
    }.start(wait = true)
}
```

## Coroutine の使い方

すべての MCP operation は suspending function です。Kotlin coroutine を正しく使ってください。

```kotlin
import kotlinx.coroutines.coroutineScope
import kotlinx.coroutines.async

server.addTool(
    name = "parallel-search",
    description = "Search multiple sources in parallel"
) { request ->
    coroutineScope {
        val source1 = async { searchSource1(query) }
        val source2 = async { searchSource2(query) }

        val results = source1.await() + source2.await()

        CallToolResult(
            content = listOf(TextContent(text = results.joinToString("\n")))
        )
    }
}
```

## Error Handling

Kotlin の例外処理を使い、意味のある error message を提供してください。

```kotlin
server.addTool(
    name = "validate-input",
    description = "Process validated input"
) { request ->
    try {
        val input = request.params.arguments["input"] as? String
            ?: throw IllegalArgumentException("input is required")

        require(input.isNotBlank()) { "input cannot be blank" }

        val result = processInput(input)

        CallToolResult(
            content = listOf(TextContent(text = result))
        )
    } catch (e: IllegalArgumentException) {
        CallToolResult(
            isError = true,
            content = listOf(TextContent(text = "Validation error: ${e.message}"))
        )
    }
}
```

## kotlinx.serialization を使った JSON Schema

型安全な JSON schema には kotlinx.serialization を使ってください。

```kotlin
import kotlinx.serialization.Serializable
import kotlinx.serialization.json.*

@Serializable
data class SearchInput(
    val query: String,
    val limit: Int = 10,
    val filters: List<String> = emptyList()
)

fun createToolSchema(): JsonObject = buildJsonObject {
    put("type", "object")
    putJsonObject("properties") {
        putJsonObject("query") {
            put("type", "string")
            put("description", "Search query")
        }
        putJsonObject("limit") {
            put("type", "integer")
            put("default", 10)
        }
        putJsonObject("filters") {
            put("type", "array")
            putJsonObject("items") {
                put("type", "string")
            }
        }
    }
    putJsonArray("required") {
        add("query")
    }
}
```

## Gradle 設定

`build.gradle.kts` を適切に設定してください。

```kotlin
plugins {
    kotlin("jvm") version "2.1.0"
    kotlin("plugin.serialization") version "2.1.0"
}

repositories {
    mavenCentral()
}

dependencies {
    implementation("io.modelcontextprotocol:kotlin-sdk:0.7.2")

    // For client transport
    implementation("io.ktor:ktor-client-cio:3.0.0")

    // For server transport
    implementation("io.ktor:ktor-server-netty:3.0.0")

    // For JSON serialization
    implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.7.3")

    // For coroutines
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.9.0")
}
```

## Multiplatform Support

Kotlin SDK は Kotlin Multiplatform（JVM、Wasm、iOS）をサポートしています。

```kotlin
kotlin {
    jvm()
    js(IR) {
        browser()
        nodejs()
    }
    wasmJs()

    sourceSets {
        commonMain.dependencies {
            implementation("io.modelcontextprotocol:kotlin-sdk:0.7.2")
            implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.9.0")
        }
    }
}
```

## Resource Lifecycle

resource の更新と subscription を扱ってください。

```kotlin
server.addResource(
    uri = "file:///dynamic/data",
    name = "Dynamic Data",
    description = "Frequently updated data",
    mimeType = "application/json"
) { request ->
    // Provide current state
    ReadResourceResult(
        contents = listOf(
            TextResourceContents(
                text = getCurrentData(),
                uri = request.uri,
                mimeType = "application/json"
            )
        )
    )
}

// Notify clients when resource changes
server.notifyResourceListChanged()
```

## テスト

MCP tool のテストには Kotlin coroutine test utility を使ってください。

```kotlin
import kotlinx.coroutines.test.runTest
import kotlin.test.Test
import kotlin.test.assertEquals

class ServerTest {
    @Test
    fun testSearchTool() = runTest {
        val server = createTestServer()

        val request = CallToolRequest(
            params = CallToolParams(
                name = "search",
                arguments = mapOf("query" to "test", "limit" to 5)
            )
        )

        val result = server.callTool(request)

        assertEquals(false, result.isError)
        assert(result.content.isNotEmpty())
    }
}
```

## よくあるパターン

### Logging

構造化 logging には Kotlin logging library を使ってください。

```kotlin
import io.github.oshai.kotlinlogging.KotlinLogging

private val logger = KotlinLogging.logger {}

server.addTool(
    name = "logged-operation",
    description = "Operation with logging"
) { request ->
    logger.info { "Tool called with args: ${request.params.arguments}" }

    try {
        val result = performOperation(request)
        logger.info { "Operation succeeded" }
        result
    } catch (e: Exception) {
        logger.error(e) { "Operation failed" }
        throw e
    }
}
```

### Configuration

設定には data class を使ってください。

```kotlin
import kotlinx.serialization.Serializable

@Serializable
data class ServerConfig(
    val name: String = "my-server",
    val version: String = "1.0.0",
    val port: Int = 8080,
    val enableTools: Boolean = true
)

fun loadConfig(): ServerConfig {
    // Load from environment or config file
    return ServerConfig(
        name = System.getenv("SERVER_NAME") ?: "my-server",
        version = System.getenv("VERSION") ?: "1.0.0"
    )
}
```

### Dependency Injection

testability のために constructor injection を使ってください。

```kotlin
class MyServer(
    private val dataService: DataService,
    private val config: ServerConfig
) {
    fun createServer() = Server(
        serverInfo = Implementation(
            name = config.name,
            version = config.version
        )
    ) {
        "MCP Server with DI"
    }.apply {
        addTool(
            name = "fetch-data",
            description = "Fetch data using injected service"
        ) { request ->
            val data = dataService.fetchData()
            CallToolResult(
                content = listOf(TextContent(text = data))
            )
        }
    }
}
```
