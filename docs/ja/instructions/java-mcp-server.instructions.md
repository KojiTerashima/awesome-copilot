---
description: '公式 MCP Java SDK と reactive stream、Spring integration を用いて Java で Model Context Protocol (MCP) server を構築するためのベストプラクティスとパターン。'
applyTo: "**/*.java, **/pom.xml, **/build.gradle, **/build.gradle.kts"
---

# Java MCP Server 開発ガイドライン

Java で MCP server を構築する際は、公式 Java SDK を使って次のベストプラクティスとパターンに従ってください。

## 依存関係

Maven project に MCP Java SDK を追加してください。

```xml
<dependencies>
    <dependency>
        <groupId>io.modelcontextprotocol.sdk</groupId>
        <artifactId>mcp</artifactId>
        <version>0.14.1</version>
    </dependency>
</dependencies>
```

Gradle の場合:

```kotlin
dependencies {
    implementation("io.modelcontextprotocol.sdk:mcp:0.14.1")
}
```

## Server のセットアップ

builder pattern を使って MCP server を作成してください。

```java
import io.mcp.server.McpServer;
import io.mcp.server.McpServerBuilder;
import io.mcp.server.transport.StdioServerTransport;

McpServer server = McpServerBuilder.builder()
    .serverInfo("my-server", "1.0.0")
    .capabilities(capabilities -> capabilities
        .tools(true)
        .resources(true)
        .prompts(true))
    .build();

// Start with stdio transport
StdioServerTransport transport = new StdioServerTransport();
server.start(transport).subscribe();
```

## Tool の追加

server に tool handler を登録してください。

```java
import io.mcp.server.tool.Tool;
import io.mcp.server.tool.ToolHandler;
import reactor.core.publisher.Mono;

// Define a tool
Tool searchTool = Tool.builder()
    .name("search")
    .description("Search for information")
    .inputSchema(JsonSchema.object()
        .property("query", JsonSchema.string()
            .description("Search query")
            .required(true))
        .property("limit", JsonSchema.integer()
            .description("Maximum results")
            .defaultValue(10)))
    .build();

// Register tool handler
server.addToolHandler("search", (arguments) -> {
    String query = arguments.get("query").asText();
    int limit = arguments.has("limit")
        ? arguments.get("limit").asInt()
        : 10;

    // Perform search
    List<String> results = performSearch(query, limit);

    return Mono.just(ToolResponse.success()
        .addTextContent("Found " + results.size() + " results")
        .build());
});
```

## Resource の追加

data access 用の resource handler を実装してください。

```java
import io.mcp.server.resource.Resource;
import io.mcp.server.resource.ResourceHandler;

// Register resource list handler
server.addResourceListHandler(() -> {
    List<Resource> resources = List.of(
        Resource.builder()
            .name("Data File")
            .uri("resource://data/example.txt")
            .description("Example data file")
            .mimeType("text/plain")
            .build()
    );
    return Mono.just(resources);
});

// Register resource read handler
server.addResourceReadHandler((uri) -> {
    if (uri.equals("resource://data/example.txt")) {
        String content = loadResourceContent(uri);
        return Mono.just(ResourceContent.text(content, uri));
    }
    throw new ResourceNotFoundException(uri);
});

// Register resource subscribe handler
server.addResourceSubscribeHandler((uri) -> {
    subscriptions.add(uri);
    log.info("Client subscribed to {}", uri);
    return Mono.empty();
});
```

## Prompt の追加

template 化された会話用の prompt handler を実装してください。

```java
import io.mcp.server.prompt.Prompt;
import io.mcp.server.prompt.PromptMessage;
import io.mcp.server.prompt.PromptArgument;

// Register prompt list handler
server.addPromptListHandler(() -> {
    List<Prompt> prompts = List.of(
        Prompt.builder()
            .name("analyze")
            .description("Analyze a topic")
            .argument(PromptArgument.builder()
                .name("topic")
                .description("Topic to analyze")
                .required(true)
                .build())
            .argument(PromptArgument.builder()
                .name("depth")
                .description("Analysis depth")
                .required(false)
                .build())
            .build()
    );
    return Mono.just(prompts);
});

// Register prompt get handler
server.addPromptGetHandler((name, arguments) -> {
    if (name.equals("analyze")) {
        String topic = arguments.getOrDefault("topic", "general");
        String depth = arguments.getOrDefault("depth", "basic");

        List<PromptMessage> messages = List.of(
            PromptMessage.user("Please analyze this topic: " + topic),
            PromptMessage.assistant("I'll provide a " + depth + " analysis of " + topic)
        );

        return Mono.just(PromptResult.builder()
            .description("Analysis of " + topic + " at " + depth + " level")
            .messages(messages)
            .build());
    }
    throw new PromptNotFoundException(name);
});
```

## Reactive Streams パターン

Java SDK は、非同期処理に Reactive Streams（Project Reactor）を使います。

```java
// Return Mono for single results
server.addToolHandler("process", (args) -> {
    return Mono.fromCallable(() -> {
        String result = expensiveOperation(args);
        return ToolResponse.success()
            .addTextContent(result)
            .build();
    }).subscribeOn(Schedulers.boundedElastic());
});

// Return Flux for streaming results
server.addResourceListHandler(() -> {
    return Flux.fromIterable(getResources())
        .map(r -> Resource.builder()
            .uri(r.getUri())
            .name(r.getName())
            .build())
        .collectList();
});
```

## Synchronous Facade

blocking な用途では synchronous API を使ってください。

```java
import io.mcp.server.McpSyncServer;

McpSyncServer syncServer = server.toSyncServer();

// Blocking tool handler
syncServer.addToolHandler("greet", (args) -> {
    String name = args.get("name").asText();
    return ToolResponse.success()
        .addTextContent("Hello, " + name + "!")
        .build();
});
```

## Transport 設定

### Stdio Transport

ローカル subprocess 通信用:

```java
import io.mcp.server.transport.StdioServerTransport;

StdioServerTransport transport = new StdioServerTransport();
server.start(transport).block();
```

### HTTP Transport (Servlet)

HTTP ベース server 用:

```java
import io.mcp.server.transport.ServletServerTransport;
import jakarta.servlet.http.HttpServlet;

public class McpServlet extends HttpServlet {
    private final McpServer server;
    private final ServletServerTransport transport;

    public McpServlet() {
        this.server = createMcpServer();
        this.transport = new ServletServerTransport();
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) {
        transport.handleRequest(server, req, resp).block();
    }
}
```

## Spring Boot Integration

シームレスな統合には Spring Boot starter を使ってください。

```xml
<dependency>
    <groupId>io.modelcontextprotocol.sdk</groupId>
    <artifactId>mcp-spring-boot-starter</artifactId>
    <version>0.14.1</version>
</dependency>
```

Spring で server を構成します。

```java
import org.springframework.context.annotation.Configuration;
import io.mcp.spring.McpServerConfigurer;

@Configuration
public class McpConfiguration {

    @Bean
    public McpServerConfigurer mcpServerConfigurer() {
        return server -> server
            .serverInfo("spring-server", "1.0.0")
            .capabilities(cap -> cap
                .tools(true)
                .resources(true)
                .prompts(true));
    }
}
```

handler は Spring bean として登録してください。

```java
import org.springframework.stereotype.Component;
import io.mcp.spring.ToolHandler;

@Component
public class SearchToolHandler implements ToolHandler {

    @Override
    public String getName() {
        return "search";
    }

    @Override
    public Tool getTool() {
        return Tool.builder()
            .name("search")
            .description("Search for information")
            .inputSchema(JsonSchema.object()
                .property("query", JsonSchema.string().required(true)))
            .build();
    }

    @Override
    public Mono<ToolResponse> handle(JsonNode arguments) {
        String query = arguments.get("query").asText();
        return Mono.just(ToolResponse.success()
            .addTextContent("Search results for: " + query)
            .build());
    }
}
```

## Error Handling

MCP exception を使って適切な error handling を行ってください。

```java
server.addToolHandler("risky", (args) -> {
    return Mono.fromCallable(() -> {
        try {
            String result = riskyOperation(args);
            return ToolResponse.success()
                .addTextContent(result)
                .build();
        } catch (ValidationException e) {
            return ToolResponse.error()
                .message("Invalid input: " + e.getMessage())
                .build();
        } catch (Exception e) {
            log.error("Unexpected error", e);
            return ToolResponse.error()
                .message("Internal error occurred")
                .build();
        }
    });
});
```

## JSON Schema の構築

fluent schema builder を使ってください。

```java
import io.mcp.json.JsonSchema;

JsonSchema schema = JsonSchema.object()
    .property("name", JsonSchema.string()
        .description("User's name")
        .minLength(1)
        .maxLength(100)
        .required(true))
    .property("age", JsonSchema.integer()
        .description("User's age")
        .minimum(0)
        .maximum(150))
    .property("email", JsonSchema.string()
        .description("Email address")
        .format("email")
        .required(true))
    .property("tags", JsonSchema.array()
        .items(JsonSchema.string())
        .uniqueItems(true))
    .additionalProperties(false)
    .build();
```

## Logging と Observability

logging には SLF4J を使ってください。

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

private static final Logger log = LoggerFactory.getLogger(MyMcpServer.class);

server.addToolHandler("process", (args) -> {
    log.info("Tool called: process, args: {}", args);

    return Mono.fromCallable(() -> {
        String result = process(args);
        log.debug("Processing completed successfully");
        return ToolResponse.success()
            .addTextContent(result)
            .build();
    }).doOnError(error -> {
        log.error("Processing failed", error);
    });
});
```

Reactor で context を伝播します。

```java
import reactor.util.context.Context;

server.addToolHandler("traced", (args) -> {
    return Mono.deferContextual(ctx -> {
        String traceId = ctx.get("traceId");
        log.info("Processing with traceId: {}", traceId);

        return Mono.just(ToolResponse.success()
            .addTextContent("Processed")
            .build());
    });
});
```

## テスト

synchronous API を使って test を書いてください。

```java
import org.junit.jupiter.api.Test;
import static org.assertj.core.Assertions.assertThat;

class McpServerTest {

    @Test
    void testToolHandler() {
        McpServer server = createTestServer();
        McpSyncServer syncServer = server.toSyncServer();

        JsonNode args = objectMapper.createObjectNode()
            .put("query", "test");

        ToolResponse response = syncServer.callTool("search", args);

        assertThat(response.isError()).isFalse();
        assertThat(response.getContent()).hasSize(1);
    }
}
```

## Jackson Integration

SDK は JSON serialization に Jackson を使います。必要に応じてカスタマイズしてください。

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.datatype.jsr310.JavaTimeModule;

ObjectMapper mapper = new ObjectMapper();
mapper.registerModule(new JavaTimeModule());

// Use custom mapper with server
McpServer server = McpServerBuilder.builder()
    .objectMapper(mapper)
    .build();
```

## Content Type

response では複数の content type をサポートしてください。

```java
import io.mcp.server.content.Content;

server.addToolHandler("multi", (args) -> {
    return Mono.just(ToolResponse.success()
        .addTextContent("Plain text response")
        .addImageContent(imageBytes, "image/png")
        .addResourceContent("resource://data", "application/json", jsonData)
        .build());
});
```

## Server Lifecycle

server lifecycle は適切に管理してください。

```java
import reactor.core.Disposable;

Disposable serverDisposable = server.start(transport).subscribe();

// Graceful shutdown
Runtime.getRuntime().addShutdownHook(new Thread(() -> {
    log.info("Shutting down MCP server");
    serverDisposable.dispose();
    server.stop().block();
}));
```

## よくあるパターン

### Request Validation

```java
server.addToolHandler("validate", (args) -> {
    if (!args.has("required_field")) {
        return Mono.just(ToolResponse.error()
            .message("Missing required_field")
            .build());
    }

    return processRequest(args);
});
```

### Async Operation

```java
server.addToolHandler("async", (args) -> {
    return Mono.fromCallable(() -> callExternalApi(args))
        .timeout(Duration.ofSeconds(30))
        .onErrorResume(TimeoutException.class, e ->
            Mono.just(ToolResponse.error()
                .message("Operation timed out")
                .build()))
        .subscribeOn(Schedulers.boundedElastic());
});
```

### Resource Caching

```java
private final Map<String, String> cache = new ConcurrentHashMap<>();

server.addResourceReadHandler((uri) -> {
    return Mono.fromCallable(() ->
        cache.computeIfAbsent(uri, this::loadResource))
        .map(content -> ResourceContent.text(content, uri));
});
```

## ベストプラクティス

1. 非同期処理と backpressure には **Reactive Streams** を使う
2. enterprise application には **Spring Boot** starter を活用する
3. 明確な error message を含む **適切な error handling** を実装する
4. `System.out` ではなく **SLF4J** を使う
5. tool handler と prompt handler では **input validation** を行う
6. 適切な resource cleanup を伴う **graceful shutdown** をサポートする
7. blocking operation には **bounded elastic scheduler** を使う
8. reactive chain の observability には **context propagation** を使う
9. test の簡潔さのために **synchronous API** を使う
10. **Java の命名規約**（method は camelCase、class は PascalCase）に従う
