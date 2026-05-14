---
name: java-mcp-server-generator
description: '公式MCP Java SDKを使用し、リアクティブストリームおよびオプションのSpring Boot統合を備えたJavaで完全なModel Context Protocolサーバープロジェクトを生成します。'
---

# Java MCPサーバージェネレーター

公式Java SDKを使用して、MavenまたはGradleで完全な本番対応のMCPサーバーをJavaで生成します。

## プロジェクト生成

Java MCPサーバーの作成を求められた場合、以下の構造で完全なプロジェクトを生成します：

```
my-mcp-server/
├── pom.xml (または build.gradle.kts)
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/example/mcp/
│   │   │       ├── McpServerApplication.java
│   │   │       ├── config/
│   │   │       │   └── ServerConfiguration.java
│   │   │       ├── tools/
│   │   │       │   ├── ToolDefinitions.java
│   │   │       │   └── ToolHandlers.java
│   │   │       ├── resources/
│   │   │       │   ├── ResourceDefinitions.java
│   │   │       │   └── ResourceHandlers.java
│   │   │       └── prompts/
│   │   │           ├── PromptDefinitions.java
│   │   │           └── PromptHandlers.java
│   │   └── resources/
│   │       └── application.properties (Spring使用時)
│   └── test/
│       └── java/
│           └── com/example/mcp/
│               └── McpServerTest.java
└── README.md
```

## Maven pom.xml テンプレート

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>my-mcp-server</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>

    <name>My MCP Server</name>
    <description>Model Context Protocolサーバー実装</description>

    <properties>
        <java.version>17</java.version>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <mcp.version>0.14.1</mcp.version>
        <slf4j.version>2.0.9</slf4j.version>
        <logback.version>1.4.11</logback.version>
        <junit.version>5.10.0</junit.version>
    </properties>

    <dependencies>
        <!-- MCP Java SDK -->
        <dependency>
            <groupId>io.modelcontextprotocol.sdk</groupId>
            <artifactId>mcp</artifactId>
            <version>${mcp.version}</version>
        </dependency>

        <!-- ロギング -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>${logback.version}</version>
        </dependency>

        <!-- テスト -->
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>io.projectreactor</groupId>
            <artifactId>reactor-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.11.0</version>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.1.2</version>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>3.5.0</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <transformers>
                                <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <mainClass>com.example.mcp.McpServerApplication</mainClass>
                                </transformer>
                            </transformers>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

## Gradle build.gradle.kts テンプレート

```kotlin
plugins {
    id("java")
    id("application")
}

group = "com.example"
version = "1.0.0"

java {
    sourceCompatibility = JavaVersion.VERSION_17
    targetCompatibility = JavaVersion.VERSION_17
}

repositories {
    mavenCentral()
}

dependencies {
    // MCP Java SDK
    implementation("io.modelcontextprotocol.sdk:mcp:0.14.1")
    
    // ロギング
    implementation("org.slf4j:slf4j-api:2.0.9")
    implementation("ch.qos.logback:logback-classic:1.4.11")
    
    // テスト
    testImplementation("org.junit.jupiter:junit-jupiter:5.10.0")
    testImplementation("io.projectreactor:reactor-test:3.5.0")
}

application {
    mainClass.set("com.example.mcp.McpServerApplication")
}

tasks.test {
    useJUnitPlatform()
}
```

## McpServerApplication.java テンプレート

```java
package com.example.mcp;

import com.example.mcp.tools.ToolHandlers;
import com.example.mcp.resources.ResourceHandlers;
import com.example.mcp.prompts.PromptHandlers;
import io.mcp.server.McpServer;
import io.mcp.server.McpServerBuilder;
import io.mcp.server.transport.StdioServerTransport;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import reactor.core.Disposable;

public class McpServerApplication {
    
    private static final Logger log = LoggerFactory.getLogger(McpServerApplication.class);
    
    public static void main(String[] args) {
        log.info("MCPサーバーを起動しています...");
        
        try {
            McpServer server = createServer();
            StdioServerTransport transport = new StdioServerTransport();
            
            // サーバー起動
            Disposable serverDisposable = server.start(transport).subscribe();
            
            // グレースフルシャットダウン
            Runtime.getRuntime().addShutdownHook(new Thread(() -> {
                log.info("MCPサーバーをシャットダウンしています");
                serverDisposable.dispose();
                server.stop().block();
            }));
            
            log.info("MCPサーバーが正常に起動しました");
            
            // 実行継続
            Thread.currentThread().join();
            
        } catch (Exception e) {
            log.error("MCPサーバーの起動に失敗しました", e);
            System.exit(1);
        }
    }
    
    private static McpServer createServer() {
        McpServer server = McpServerBuilder.builder()
            .serverInfo("my-mcp-server", "1.0.0")
            .capabilities(capabilities -> capabilities
                .tools(true)
                .resources(true)
                .prompts(true))
            .build();
        
        // ハンドラー登録
        ToolHandlers.register(server);
        ResourceHandlers.register(server);
        PromptHandlers.register(server);
        
        return server;
    }
}
```

## ToolDefinitions.java テンプレート

```java
package com.example.mcp.tools;

import io.mcp.json.JsonSchema;
import io.mcp.server.tool.Tool;

import java.util.List;

public class ToolDefinitions {
    
    public static List<Tool> getTools() {
        return List.of(
            createGreetTool(),
            createCalculateTool()
        );
    }
    
    private static Tool createGreetTool() {
        return Tool.builder()
            .name("greet")
            .description("挨拶メッセージを生成します")
            .inputSchema(JsonSchema.object()
                .property("name", JsonSchema.string()
                    .description("挨拶する名前")
                    .required(true)))
            .build();
    }
    
    private static Tool createCalculateTool() {
        return Tool.builder()
            .name("calculate")
            .description("数学的計算を実行します")
            .inputSchema(JsonSchema.object()
                .property("operation", JsonSchema.string()
                    .description("実行する演算")
                    .enumValues(List.of("add", "subtract", "multiply", "divide"))
                    .required(true))
                .property("a", JsonSchema.number()
                    .description("第1オペランド")
                    .required(true))
                .property("b", JsonSchema.number()
                    .description("第2オペランド")
                    .required(true)))
            .build();
    }
}
```

## ToolHandlers.java テンプレート

```java
package com.example.mcp.tools;

import com.fasterxml.jackson.databind.JsonNode;
import io.mcp.server.McpServer;
import io.mcp.server.tool.ToolResponse;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import reactor.core.publisher.Mono;

public class ToolHandlers {
    
    private static final Logger log = LoggerFactory.getLogger(ToolHandlers.class);
    
    public static void register(McpServer server) {
        // ツール一覧ハンドラー登録
        server.addToolListHandler(() -> {
            log.debug("利用可能なツールを一覧表示しています");
            return Mono.just(ToolDefinitions.getTools());
        });
        
        // greetハンドラー登録
        server.addToolHandler("greet", ToolHandlers::handleGreet);
        
        // calculateハンドラー登録
        server.addToolHandler("calculate", ToolHandlers::handleCalculate);
    }
    
    private static Mono<ToolResponse> handleGreet(JsonNode arguments) {
        log.info("greetツールが呼び出されました");
        
        if (!arguments.has("name")) {
            return Mono.just(ToolResponse.error()
                .message("'name'パラメータがありません")
                .build());
        }
        
        String name = arguments.get("name").asText();
        String greeting = "こんにちは、" + name + "さん！MCPへようこそ。";
        
        log.debug("挨拶メッセージを生成しました: {}", name);
        
        return Mono.just(ToolResponse.success()
            .addTextContent(greeting)
            .build());
    }
    
    private static Mono<ToolResponse> handleCalculate(JsonNode arguments) {
        log.info("calculateツールが呼び出されました");
        
        if (!arguments.has("operation") || !arguments.has("a") || !arguments.has("b")) {
            return Mono.just(ToolResponse.error()
                .message("必須パラメータが不足しています")
                .build());
        }
        
        String operation = arguments.get("operation").asText();
        double a = arguments.get("a").asDouble();
        double b = arguments.get("b").asDouble();
        
        double result;
        switch (operation) {
            case "add":
                result = a + b;
                break;
            case "subtract":
                result = a - b;
                break;
            case "multiply":
                result = a * b;
                break;
            case "divide":
                if (b == 0) {
                    return Mono.just(ToolResponse.error()
                        .message("ゼロによる除算はできません")
                        .build());
                }
                result = a / b;
                break;
            default:
                return Mono.just(ToolResponse.error()
                    .message("不明な演算: " + operation)
                    .build());
        }
        
        log.debug("計算結果: {} {} {} = {}", a, operation, b, result);
        
        return Mono.just(ToolResponse.success()
            .addTextContent("結果: " + result)
            .build());
    }
}
```

## ResourceDefinitions.java テンプレート

```java
package com.example.mcp.resources;

import io.mcp.server.resource.Resource;

import java.util.List;

public class ResourceDefinitions {
    
    public static List<Resource> getResources() {
        return List.of(
            Resource.builder()
                .name("Example Data")
                .uri("resource://data/example")
                .description("例のリソースデータ")
                .mimeType("application/json")
                .build(),
            Resource.builder()
                .name("Configuration")
                .uri("resource://config")
                .description("サーバー設定")
                .mimeType("application/json")
                .build()
        );
    }
}
```

## ResourceHandlers.java テンプレート

```java
package com.example.mcp.resources;

import io.mcp.server.McpServer;
import io.mcp.server.resource.ResourceContent;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import reactor.core.publisher.Mono;

import java.time.Instant;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

public class ResourceHandlers {
    
    private static final Logger log = LoggerFactory.getLogger(ResourceHandlers.class);
    private static final Map<String, Boolean> subscriptions = new ConcurrentHashMap<>();
    
    public static void register(McpServer server) {
        // リソース一覧ハンドラー登録
        server.addResourceListHandler(() -> {
            log.debug("利用可能なリソースを一覧表示しています");
            return Mono.just(ResourceDefinitions.getResources());
        });
        
        // リソース読み取りハンドラー登録
        server.addResourceReadHandler(ResourceHandlers::handleRead);
        
        // リソース購読ハンドラー登録
        server.addResourceSubscribeHandler(ResourceHandlers::handleSubscribe);
        
        // リソース購読解除ハンドラー登録
        server.addResourceUnsubscribeHandler(ResourceHandlers::handleUnsubscribe);
    }
    
    private static Mono<ResourceContent> handleRead(String uri) {
        log.info("リソースを読み取っています: {}", uri);
        
        switch (uri) {
            case "resource://data/example":
                String jsonData = String.format(
                    "{\"message\":\"例のリソースデータ\",\"timestamp\":\"%s\"}",
                    Instant.now()
                );
                return Mono.just(ResourceContent.text(jsonData, uri, "application/json"));
                
            case "resource://config":
                String config = "{\"serverName\":\"my-mcp-server\",\"version\":\"1.0.0\"}";
                return Mono.just(ResourceContent.text(config, uri, "application/json"));
                
            default:
                log.warn("不明なリソースが要求されました: {}", uri);
                return Mono.error(new IllegalArgumentException("不明なリソースURI: " + uri));
        }
    }
    
    private static Mono<Void> handleSubscribe(String uri) {
        log.info("クライアントがリソースを購読しました: {}", uri);
        subscriptions.put(uri, true);
        return Mono.empty();
    }
    
    private static Mono<Void> handleUnsubscribe(String uri) {
        log.info("クライアントがリソースの購読を解除しました: {}", uri);
        subscriptions.remove(uri);
        return Mono.empty();
    }
}
```

## PromptDefinitions.java テンプレート

```java
package com.example.mcp.prompts;

import io.mcp.server.prompt.Prompt;
import io.mcp.server.prompt.PromptArgument;

import java.util.List;

public class PromptDefinitions {
    
    public static List<Prompt> getPrompts() {
        return List.of(
            Prompt.builder()
                .name("code-review")
                .description("コードレビュー用プロンプトを生成します")
                .argument(PromptArgument.builder()
                    .name("language")
                    .description("プログラミング言語")
                    .required(true)
                    .build())
                .argument(PromptArgument.builder()
                    .name("focus")
                    .description("レビューの焦点")
                    .required(false)
                    .build())
                .build()
        );
    }
}
```

## PromptHandlers.java テンプレート

```java
package com.example.mcp.prompts;

import io.mcp.server.McpServer;
import io.mcp.server.prompt.PromptMessage;
import io.mcp.server.prompt.PromptResult;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import reactor.core.publisher.Mono;

import java.util.List;
import java.util.Map;

public class PromptHandlers {
    
    private static final Logger log = LoggerFactory.getLogger(PromptHandlers.class);
    
    public static void register(McpServer server) {
        // プロンプト一覧ハンドラー登録
        server.addPromptListHandler(() -> {
            log.debug("利用可能なプロンプトを一覧表示しています");
            return Mono.just(PromptDefinitions.getPrompts());
        });
        
        // プロンプト取得ハンドラー登録
        server.addPromptGetHandler(PromptHandlers::handleCodeReview);
    }
    
    private static Mono<PromptResult> handleCodeReview(String name, Map<String, String> arguments) {
        log.info("プロンプトを取得しています: {}", name);
        
        if (!name.equals("code-review")) {
            return Mono.error(new IllegalArgumentException("不明なプロンプト: " + name));
        }
        
        String language = arguments.getOrDefault("language", "Java");
        String focus = arguments.getOrDefault("focus", "一般的な品質");
        
        String description = language + "のコードレビュー（焦点: " + focus + "）";
        
        List<PromptMessage> messages = List.of(
            PromptMessage.user("この" + language + "コードを" + focus + "に焦点を当ててレビューしてください。"),
            PromptMessage.assistant(focus + "に焦点を当ててコードをレビューします。コードを共有してください。"),
            PromptMessage.user("レビューするコードはこちらです: [ここにコードを貼り付けてください]")
        );
        
        log.debug("{}（{}）用のコードレビュープロンプトを生成しました", language, focus);
        
        return Mono.just(PromptResult.builder()
            .description(description)
            .messages(messages)
            .build());
    }
}
```

## McpServerTest.java テンプレート

```java
package com.example.mcp;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.node.ObjectNode;
import io.mcp.server.McpServer;
import io.mcp.server.McpSyncServer;
import io.mcp.server.tool.ToolResponse;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

class McpServerTest {
    
    private McpSyncServer syncServer;
    private ObjectMapper objectMapper;
    
    @BeforeEach
    void setUp() {
        McpServer server = createTestServer();
        syncServer = server.toSyncServer();
        objectMapper = new ObjectMapper();
    }
    
    private McpServer createTestServer() {
        // メインアプリケーションと同じセットアップ
        McpServer server = McpServerBuilder.builder()
            .serverInfo("test-server", "1.0.0")
            .capabilities(cap -> cap.tools(true))
            .build();
        
        // ハンドラー登録
        ToolHandlers.register(server);
        
        return server;
    }
    
    @Test
    void testGreetTool() {
        ObjectNode args = objectMapper.createObjectNode();
        args.put("name", "Java");
        
        ToolResponse response = syncServer.callTool("greet", args);
        
        assertFalse(response.isError());
        assertEquals(1, response.getContent().size());
        assertTrue(response.getContent().get(0).getText().contains("Java"));
    }
    
    @Test
    void testCalculateTool() {
        ObjectNode args = objectMapper.createObjectNode();
        args.put("operation", "add");
        args.put("a", 5);
        args.put("b", 3);
        
        ToolResponse response = syncServer.callTool("calculate", args);
        
        assertFalse(response.isError());
        assertTrue(response.getContent().get(0).getText().contains("8"));
    }
    
    @Test
    void testDivideByZero() {
        ObjectNode args = objectMapper.createObjectNode();
        args.put("operation", "divide");
        args.put("a", 10);
        args.put("b", 0);
        
        ToolResponse response = syncServer.callTool("calculate", args);
        
        assertTrue(response.isError());
    }
}
```

## README.md テンプレート

```markdown
# My MCP Server

Javaと公式MCP Java SDKで構築されたModel Context Protocolサーバー。

## 特徴

- ✅ ツール: greet, calculate
- ✅ リソース: 例のデータ、設定
- ✅ プロンプト: code-review
- ✅ Project Reactorによるリアクティブストリーム
- ✅ SLF4Jによる構造化ロギング
- ✅ 完全なテストカバレッジ

## 要件

- Java 17以上
- Maven 3.6以上または Gradle 7以上

## ビルド

### Maven
```bash
mvn clean package
```

### Gradle
```bash
./gradlew build
```

## 実行

### Maven
```bash
java -jar target/my-mcp-server-1.0.0.jar
```

### Gradle
```bash
./gradlew run
```

## テスト

### Maven
```bash
mvn test
```

### Gradle
```bash
./gradlew test
```

## Claude Desktopとの統合

`claude_desktop_config.json`に以下を追加：

```json
{
  "mcpServers": {
    "my-mcp-server": {
      "command": "java",
      "args": ["-jar", "/path/to/my-mcp-server-1.0.0.jar"]
    }
  }
}
```

## ライセンス

MIT
```

## 生成手順

1. **プロジェクト名とパッケージを尋ねる**
2. **ビルドツールを選択する**（MavenまたはGradle）
3. **適切なパッケージ構造で全ファイルを生成する**
4. **非同期ハンドラーにリアクティブストリームを使用する**
5. **SLF4Jによる包括的なロギングを含める**
6. **すべてのハンドラーに対するテストを追加する**
7. **Javaの命名規則（camelCase、PascalCase）に従う**
8. **適切なレスポンスを伴うエラーハンドリングを含める**
9. **公開APIにJavadocでドキュメントを付与する**
10. **同期・非同期の両方の例を提供する**
