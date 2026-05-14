---
name: kotlin-mcp-server-generator
description: '公式の io.modelcontextprotocol:kotlin-sdk ライブラリを使用して、適切な構造、依存関係、実装を備えた完全な Kotlin MCP サーバープロジェクトを生成します。'
---

# Kotlin MCP サーバープロジェクトジェネレーター

Kotlinで完全な本番対応のModel Context Protocol（MCP）サーバープロジェクトを生成します。

## プロジェクト要件

以下を備えたKotlin MCPサーバーを作成します：

1. **プロジェクト構造**：GradleベースのKotlinプロジェクトレイアウト
2. **依存関係**：公式MCP SDK、Ktor、kotlinxライブラリ
3. **サーバー設定**：トランスポートを設定したMCPサーバー
4. **ツール**：型付き入出力を持つ便利なツールを2〜3個以上
5. **エラーハンドリング**：適切な例外処理とバリデーション
6. **ドキュメント**：セットアップと使用方法を記載したREADME
7. **テスト**：コルーチンを使った基本的なテスト構造

## テンプレート構造

```
myserver/
├── build.gradle.kts
├── settings.gradle.kts
├── gradle.properties
├── src/
│   ├── main/
│   │   └── kotlin/
│   │       └── com/example/myserver/
│   │           ├── Main.kt
│   │           ├── Server.kt
│   │           ├── config/
│   │           │   └── Config.kt
│   │           └── tools/
│   │               ├── Tool1.kt
│   │               └── Tool2.kt
│   └── test/
│       └── kotlin/
│           └── com/example/myserver/
│               └── ServerTest.kt
└── README.md
```

## build.gradle.kts テンプレート

```kotlin
plugins {
    kotlin("jvm") version "2.1.0"
    kotlin("plugin.serialization") version "2.1.0"
    application
}

group = "com.example"
version = "1.0.0"

repositories {
    mavenCentral()
}

dependencies {
    implementation("io.modelcontextprotocol:kotlin-sdk:0.7.2")
    
    // トランスポート用Ktor
    implementation("io.ktor:ktor-server-netty:3.0.0")
    implementation("io.ktor:ktor-client-cio:3.0.0")
    
    // シリアライゼーション
    implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.7.3")
    
    // コルーチン
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.9.0")
    
    // ロギング
    implementation("io.github.oshai:kotlin-logging-jvm:7.0.0")
    implementation("ch.qos.logback:logback-classic:1.5.12")
    
    // テスト
    testImplementation(kotlin("test"))
    testImplementation("org.jetbrains.kotlinx:kotlinx-coroutines-test:1.9.0")
}

application {
    mainClass.set("com.example.myserver.MainKt")
}

tasks.test {
    useJUnitPlatform()
}

kotlin {
    jvmToolchain(17)
}
```

## settings.gradle.kts テンプレート

```kotlin
rootProject.name = "{{PROJECT_NAME}}"
```

## Main.kt テンプレート

```kotlin
package com.example.myserver

import io.modelcontextprotocol.kotlin.sdk.server.StdioServerTransport
import kotlinx.coroutines.runBlocking
import io.github.oshai.kotlinlogging.KotlinLogging

private val logger = KotlinLogging.logger {}

fun main() = runBlocking {
    logger.info { "MCPサーバーを起動しています..." }
    
    val config = loadConfig()
    val server = createServer(config)
    
    // stdioトランスポートを使用
    val transport = StdioServerTransport()
    
    logger.info { "サーバー '${config.name}' バージョン${config.version} が準備完了しました" }
    server.connect(transport)
}
```

## Server.kt テンプレート

```kotlin
package com.example.myserver

import io.modelcontextprotocol.kotlin.sdk.server.Server
import io.modelcontextprotocol.kotlin.sdk.server.ServerOptions
import io.modelcontextprotocol.kotlin.sdk.Implementation
import io.modelcontextprotocol.kotlin.sdk.ServerCapabilities
import com.example.myserver.tools.registerTools

fun createServer(config: Config): Server {
    val server = Server(
        serverInfo = Implementation(
            name = config.name,
            version = config.version
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
        )
    ) {
        config.description
    }
    
    // すべてのツールを登録
    server.registerTools()
    
    return server
}
```

## Config.kt テンプレート

```kotlin
package com.example.myserver.config

import kotlinx.serialization.Serializable

@Serializable
data class Config(
    val name: String = "{{PROJECT_NAME}}",
    val version: String = "1.0.0",
    val description: String = "{{PROJECT_DESCRIPTION}}"
)

fun loadConfig(): Config {
    return Config(
        name = System.getenv("SERVER_NAME") ?: "{{PROJECT_NAME}}",
        version = System.getenv("VERSION") ?: "1.0.0",
        description = System.getenv("DESCRIPTION") ?: "{{PROJECT_DESCRIPTION}}"
    )
}
```

## Tool1.kt テンプレート

```kotlin
package com.example.myserver.tools

import io.modelcontextprotocol.kotlin.sdk.server.Server
import io.modelcontextprotocol.kotlin.sdk.CallToolRequest
import io.modelcontextprotocol.kotlin.sdk.CallToolResult
import io.modelcontextprotocol.kotlin.sdk.TextContent
import kotlinx.serialization.json.buildJsonObject
import kotlinx.serialization.json.put
import kotlinx.serialization.json.putJsonObject
import kotlinx.serialization.json.putJsonArray

fun Server.registerTool1() {
    addTool(
        name = "tool1",
        description = "tool1が行う処理の説明",
        inputSchema = buildJsonObject {
            put("type", "object")
            putJsonObject("properties") {
                putJsonObject("param1") {
                    put("type", "string")
                    put("description", "最初のパラメーター")
                }
                putJsonObject("param2") {
                    put("type", "integer")
                    put("description", "オプションの2番目のパラメーター")
                }
            }
            putJsonArray("required") {
                add("param1")
            }
        }
    ) { request: CallToolRequest ->
        // パラメーターの抽出と検証
        val param1 = request.params.arguments["param1"] as? String
            ?: throw IllegalArgumentException("param1は必須です")
        val param2 = (request.params.arguments["param2"] as? Number)?.toInt() ?: 0
        
        // ツールのロジックを実行
        val result = performTool1Logic(param1, param2)
        
        CallToolResult(
            content = listOf(
                TextContent(text = result)
            )
        )
    }
}

private fun performTool1Logic(param1: String, param2: Int): String {
    // ここにツールのロジックを実装
    return "処理結果: $param1 と値 $param2"
}
```

## tools/ToolRegistry.kt テンプレート

```kotlin
package com.example.myserver.tools

import io.modelcontextprotocol.kotlin.sdk.server.Server

fun Server.registerTools() {
    registerTool1()
    registerTool2()
    // 追加のツールはここに登録
}
```

## ServerTest.kt テンプレート

```kotlin
package com.example.myserver

import kotlinx.coroutines.test.runTest
import kotlin.test.Test
import kotlin.test.assertEquals
import kotlin.test.assertFalse

class ServerTest {
    
    @Test
    fun `サーバー作成のテスト`() = runTest {
        val config = Config(
            name = "test-server",
            version = "1.0.0",
            description = "テストサーバー"
        )
        
        val server = createServer(config)
        
        assertEquals("test-server", server.serverInfo.name)
        assertEquals("1.0.0", server.serverInfo.version)
    }
    
    @Test
    fun `tool1実行のテスト`() = runTest {
        val config = Config()
        val server = createServer(config)
        
        // ツール実行のテスト
        // 注意: サーバー内のツール呼び出し用の適切なテストユーティリティを実装する必要があります
    }
}
```

## README.md テンプレート

```markdown
# {{PROJECT_NAME}}

Kotlinで構築されたModel Context Protocol（MCP）サーバー。

## 説明

{{PROJECT_DESCRIPTION}}

## 要件

- Java 17以上
- Kotlin 2.1.0

## インストール

プロジェクトをビルド：

\`\`\`bash
./gradlew build
\`\`\`

## 使い方

stdioトランスポートでサーバーを起動：

\`\`\`bash
./gradlew run
\`\`\`

またはjarをビルドして実行：

\`\`\`bash
./gradlew installDist
./build/install/{{PROJECT_NAME}}/bin/{{PROJECT_NAME}}
\`\`\`

## 設定

環境変数で設定：

- `SERVER_NAME`: サーバー名（デフォルト: "{{PROJECT_NAME}}"）
- `VERSION`: サーバーバージョン（デフォルト: "1.0.0"）
- `DESCRIPTION`: サーバー説明

## 利用可能なツール

### tool1
{{TOOL1_DESCRIPTION}}

**入力:**
- `param1` (文字列、必須): 最初のパラメーター
- `param2` (整数、任意): 2番目のパラメーター

**出力:**
- 操作のテキスト結果

## 開発

テストを実行：

\`\`\`bash
./gradlew test
\`\`\`

ビルド：

\`\`\`bash
./gradlew build
\`\`\`

自動リロード付きで実行（開発用）：

\`\`\`bash
./gradlew run --continuous
\`\`\`

## マルチプラットフォーム

このプロジェクトはKotlinマルチプラットフォームを使用し、JVM、Wasm、iOSをターゲットにできます。  
プラットフォーム設定は `build.gradle.kts` を参照してください。

## ライセンス

MIT
```

## 生成手順

Kotlin MCPサーバーを生成する際は：

1. **Gradle設定**：すべての依存関係を含む適切な `build.gradle.kts` を作成
2. **パッケージ構造**：Kotlinのパッケージ規約に従う
3. **型安全**：データクラスとkotlinx.serializationを使用
4. **コルーチン**：すべての操作はsuspend関数にする
5. **エラーハンドリング**：Kotlin例外とバリデーションを使用
6. **JSONスキーマ**：ツールスキーマに `buildJsonObject` を使用
7. **テスト**：コルーチンテストユーティリティを含める
8. **ロギング**：kotlin-loggingで構造化ログを使用
9. **設定**：データクラスと環境変数を使用
10. **ドキュメント**：公開APIにKDocコメントを付与

## ベストプラクティス

- すべての非同期操作にsuspend関数を使う
- Kotlinのnull安全性と型システムを活用
- 構造化データにはデータクラスを使う
- JSON処理にkotlinx.serializationを使う
- 結果型にはsealedクラスを使う
- Result/Eitherパターンで適切なエラーハンドリングを実装
- kotlinx-coroutines-testでテストを書く
- テスト可能性のため依存性注入を使う
- Kotlinのコーディング規約に従う
- 意味のある名前とKDocコメントを使う

## トランスポートオプション

### Stdioトランスポート
```kotlin
val transport = StdioServerTransport()
server.connect(transport)
```

### SSEトランスポート（Ktor）
```kotlin
embeddedServer(Netty, port = 8080) {
    mcp {
        Server(/*...*/) { "説明" }
    }
}.start(wait = true)
```

## マルチプラットフォーム設定

マルチプラットフォームプロジェクトの場合、`build.gradle.kts` に以下を追加：

```kotlin
kotlin {
    jvm()
    js(IR) { nodejs() }
    wasmJs()
    
    sourceSets {
        commonMain.dependencies {
            implementation("io.modelcontextprotocol:kotlin-sdk:0.7.2")
        }
    }
}
```
