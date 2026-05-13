---
description: 'async/await パターンを使う公式 rmcp SDK で Rust の Model Context Protocol サーバーを構築するためのベスト プラクティス'
applyTo: '**/*.rs'
---

# Rust MCP Server 開発のベスト プラクティス

このガイドは、公式 Rust SDK (`rmcp`) を使って Model Context Protocol (MCP) server を構築するための best practice を提供します。

## インストールとセットアップ

### Dependency を追加する

`Cargo.toml` に `rmcp` crate を追加します。

```toml
[dependencies]
rmcp = { version = "0.8.1", features = ["server"] }
tokio = { version = "1", features = ["full"] }
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
anyhow = "1.0"
tracing = "0.1"
tracing-subscriber = "0.3"
```

macro support 用:

```toml
[dependencies]
rmcp-macros = "0.8"
schemars = { version = "0.8", features = ["derive"] }
```

### プロジェクト構造

Rust MCP server project は次のように整理します。

```
my-mcp-server/
├── Cargo.toml
├── src/
│   ├── main.rs           # Server entry point
│   ├── handler.rs        # ServerHandler implementation
│   ├── tools/
│   │   ├── mod.rs
│   │   ├── calculator.rs
│   │   └── greeter.rs
│   ├── prompts/
│   │   ├── mod.rs
│   │   └── code_review.rs
│   └── resources/
│       ├── mod.rs
│       └── data.rs
└── tests/
    └── integration_tests.rs
```

## Server 実装

### 基本的な Server Setup

stdio transport を使って server を作成します。

```rust
use rmcp::{
    protocol::ServerCapabilities,
    server::{Server, ServerHandler},
    transport::StdioTransport,
};
use tokio::signal;

#[tokio::main]
async fn main() -> anyhow::Result<()> {
    tracing_subscriber::fmt::init();

    let handler = MyServerHandler::new();
    let transport = StdioTransport::new();

    let server = Server::builder()
        .with_handler(handler)
        .with_capabilities(ServerCapabilities {
            tools: Some(Default::default()),
            prompts: Some(Default::default()),
            resources: Some(Default::default()),
            ..Default::default()
        })
        .build(transport)?;

    server.run(signal::ctrl_c()).await?;

    Ok(())
}
```

### ServerHandler 実装

`ServerHandler` trait を実装します。

```rust
use rmcp::{
    model::*,
    protocol::*,
    server::{RequestContext, ServerHandler, RoleServer},
    ErrorData,
};

pub struct MyServerHandler {
    tool_router: ToolRouter,
}

impl MyServerHandler {
    pub fn new() -> Self {
        Self {
            tool_router: Self::create_tool_router(),
        }
    }

    fn create_tool_router() -> ToolRouter {
        // Initialize and return tool router
        ToolRouter::new()
    }
}

#[async_trait::async_trait]
impl ServerHandler for MyServerHandler {
    async fn list_tools(
        &self,
        _request: Option<PaginatedRequestParam>,
        _context: RequestContext<RoleServer>,
    ) -> Result<ListToolsResult, ErrorData> {
        let items = self.tool_router.list_all();
        Ok(ListToolsResult::with_all_items(items))
    }

    async fn call_tool(
        &self,
        request: CallToolRequestParam,
        context: RequestContext<RoleServer>,
    ) -> Result<CallToolResult, ErrorData> {
        let tcc = ToolCallContext::new(self, request, context);
        self.tool_router.call(tcc).await
    }
}
```

## Tool 開発

### Tool に Macro を使う

宣言的な tool 定義には `#[tool]` macro を使います。

```rust
use rmcp::tool;
use rmcp::model::Parameters;
use serde::{Deserialize, Serialize};
use schemars::JsonSchema;

#[derive(Debug, Deserialize, JsonSchema)]
pub struct CalculateParams {
    pub a: f64,
    pub b: f64,
    pub operation: String,
}

/// Performs mathematical calculations
#[tool(
    name = "calculate",
    description = "Performs basic arithmetic operations",
    annotations(read_only_hint = true)
)]
pub async fn calculate(params: Parameters<CalculateParams>) -> Result<f64, String> {
    let p = params.inner();
    match p.operation.as_str() {
        "add" => Ok(p.a + p.b),
        "subtract" => Ok(p.a - p.b),
        "multiply" => Ok(p.a * p.b),
        "divide" => {
            if p.b == 0.0 {
                Err("Division by zero".to_string())
            } else {
                Ok(p.a / p.b)
            }
        }
        _ => Err(format!("Unknown operation: {}", p.operation)),
    }
}
```

### Tool Router に Macro を使う

`#[tool_router]` と `#[tool_handler]` macro を使います。

```rust
use rmcp::{tool_router, tool_handler};

pub struct ToolsHandler {
    tool_router: ToolRouter,
}

#[tool_router]
impl ToolsHandler {
    #[tool]
    async fn greet(params: Parameters<GreetParams>) -> String {
        format!("Hello, {}!", params.inner().name)
    }

    #[tool(annotations(destructive_hint = true))]
    async fn reset_counter() -> String {
        "Counter reset".to_string()
    }

    pub fn new() -> Self {
        Self {
            tool_router: Self::tool_router(),
        }
    }
}

#[tool_handler]
impl ServerHandler for ToolsHandler {
    // Other handler methods...
}
```

### Tool Annotation

tool の振る舞いに関する hint を提供するため annotation を使います。

```rust
#[tool(
    name = "delete_file",
    annotations(
        destructive_hint = true,
        read_only_hint = false,
        idempotent_hint = false
    )
)]
pub async fn delete_file(params: Parameters<DeleteParams>) -> Result<(), String> {
    // Delete file logic
}

#[tool(
    name = "search_data",
    annotations(
        read_only_hint = true,
        idempotent_hint = true,
        open_world_hint = true
    )
)]
pub async fn search_data(params: Parameters<SearchParams>) -> Vec<String> {
    // Search logic
}
```

### Rich Content を返す

tool から structured content を返します。

```rust
use rmcp::model::{ToolResponseContent, TextContent, ImageContent};

#[tool]
async fn analyze_code(params: Parameters<CodeParams>) -> ToolResponseContent {
    ToolResponseContent::from(vec![
        TextContent::text(format!("Analysis of {}:", params.inner().filename)),
        TextContent::text("No issues found."),
    ])
}
```

## Prompt 実装

### Prompt Handler

prompt handler を実装します。

```rust
use rmcp::model::{Prompt, PromptArgument, PromptMessage, GetPromptResult};

async fn list_prompts(
    &self,
    _request: Option<PaginatedRequestParam>,
    _context: RequestContext<RoleServer>,
) -> Result<ListPromptsResult, ErrorData> {
    let prompts = vec![
        Prompt {
            name: "code-review".to_string(),
            description: Some("Review code for best practices".to_string()),
            arguments: Some(vec![
                PromptArgument {
                    name: "language".to_string(),
                    description: Some("Programming language".to_string()),
                    required: Some(true),
                },
            ]),
        },
    ];

    Ok(ListPromptsResult { prompts })
}

async fn get_prompt(
    &self,
    request: GetPromptRequestParam,
    _context: RequestContext<RoleServer>,
) -> Result<GetPromptResult, ErrorData> {
    match request.name.as_str() {
        "code-review" => {
            let language = request.arguments
                .as_ref()
                .and_then(|args| args.get("language"))
                .ok_or_else(|| ErrorData::invalid_params("language required"))?;

            Ok(GetPromptResult {
                description: Some("Code review prompt".to_string()),
                messages: vec![
                    PromptMessage::user(format!(
                        "Review this {} code for best practices and suggest improvements",
                        language
                    )),
                ],
            })
        }
        _ => Err(ErrorData::invalid_params("Unknown prompt")),
    }
}
```

## Resource 実装

### Resource Handler

resource handler を実装します。

```rust
use rmcp::model::{Resource, ResourceContents, ReadResourceResult};

async fn list_resources(
    &self,
    _request: Option<PaginatedRequestParam>,
    _context: RequestContext<RoleServer>,
) -> Result<ListResourcesResult, ErrorData> {
    let resources = vec![
        Resource {
            uri: "file:///data/config.json".to_string(),
            name: "Configuration".to_string(),
            description: Some("Server configuration".to_string()),
            mime_type: Some("application/json".to_string()),
        },
    ];

    Ok(ListResourcesResult { resources })
}

async fn read_resource(
    &self,
    request: ReadResourceRequestParam,
    _context: RequestContext<RoleServer>,
) -> Result<ReadResourceResult, ErrorData> {
    match request.uri.as_str() {
        "file:///data/config.json" => {
            let content = r#"{"version": "1.0", "enabled": true}"#;
            Ok(ReadResourceResult {
                contents: vec![
                    ResourceContents::text(content.to_string())
                        .with_uri(request.uri)
                        .with_mime_type("application/json"),
                ],
            })
        }
        _ => Err(ErrorData::invalid_params("Unknown resource")),
    }
}
```

## Transport Option

### Stdio Transport

CLI integration 向けの標準 input / output transport:

```rust
use rmcp::transport::StdioTransport;

let transport = StdioTransport::new();
let server = Server::builder()
    .with_handler(handler)
    .build(transport)?;
```

### SSE (Server-Sent Events) Transport

HTTP ベースの SSE transport:

```rust
use rmcp::transport::SseServerTransport;
use std::net::SocketAddr;

let addr: SocketAddr = "127.0.0.1:8000".parse()?;
let transport = SseServerTransport::new(addr);

let server = Server::builder()
    .with_handler(handler)
    .build(transport)?;

server.run(signal::ctrl_c()).await?;
```

### Streamable HTTP Transport

Axum を使う HTTP streaming transport:

```rust
use rmcp::transport::StreamableHttpTransport;
use axum::{Router, routing::post};

let transport = StreamableHttpTransport::new();
let app = Router::new()
    .route("/mcp", post(transport.handler()));

let listener = tokio::net::TcpListener::bind("127.0.0.1:3000").await?;
axum::serve(listener, app).await?;
```

### Custom Transport

custom transport (TCP、Unix Socket、WebSocket) を実装します。

```rust
use rmcp::transport::Transport;
use tokio::net::TcpListener;

// See examples/transport/ for TCP, Unix Socket, WebSocket implementations
```

## Error Handling

### ErrorData の使用

適切な MCP error を返します。

```rust
use rmcp::ErrorData;

fn validate_params(value: &str) -> Result<(), ErrorData> {
    if value.is_empty() {
        return Err(ErrorData::invalid_params("Value cannot be empty"));
    }
    Ok(())
}

async fn call_tool(
    &self,
    request: CallToolRequestParam,
    context: RequestContext<RoleServer>,
) -> Result<CallToolResult, ErrorData> {
    validate_params(&request.name)?;

    // Tool execution...

    Ok(CallToolResult {
        content: vec![TextContent::text("Success")],
        is_error: Some(false),
    })
}
```

### Anyhow Integration

application level の error には `anyhow` を使います。

```rust
use anyhow::{Context, Result};

async fn load_config() -> Result<Config> {
    let content = tokio::fs::read_to_string("config.json")
        .await
        .context("Failed to read config file")?;

    let config: Config = serde_json::from_str(&content)
        .context("Failed to parse config")?;

    Ok(config)
}
```

## Testing

### Unit Test

tool と handler の unit test を書きます。

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[tokio::test]
    async fn test_calculate_add() {
        let params = Parameters::new(CalculateParams {
            a: 5.0,
            b: 3.0,
            operation: "add".to_string(),
        });

        let result = calculate(params).await.unwrap();
        assert_eq!(result, 8.0);
    }

    #[tokio::test]
    async fn test_divide_by_zero() {
        let params = Parameters::new(CalculateParams {
            a: 5.0,
            b: 0.0,
            operation: "divide".to_string(),
        });

        let result = calculate(params).await;
        assert!(result.is_err());
    }
}
```

### Integration Test

server 全体の interaction をテストします。

```rust
#[tokio::test]
async fn test_server_list_tools() {
    let handler = MyServerHandler::new();
    let context = RequestContext::default();

    let result = handler.list_tools(None, context).await.unwrap();

    assert!(!result.tools.is_empty());
    assert!(result.tools.iter().any(|t| t.name == "calculate"));
}
```

## Progress Notification

### 進捗報告

長時間実行される操作の間に progress notification を送ります。

```rust
use rmcp::model::ProgressNotification;

#[tool]
async fn process_large_file(
    params: Parameters<ProcessParams>,
    context: RequestContext<RoleServer>,
) -> Result<String, String> {
    let total = 100;

    for i in 0..=total {
        // Do work...

        if i % 10 == 0 {
            context.notify_progress(ProgressNotification {
                progress: i,
                total: Some(total),
            }).await.ok();
        }
    }

    Ok("Processing complete".to_string())
}
```

## OAuth 認証

### OAuth Integration

安全な access のため OAuth を実装します。

```rust
use rmcp::oauth::{OAuthConfig, OAuthProvider};

let oauth_config = OAuthConfig {
    authorization_endpoint: "https://auth.example.com/authorize".to_string(),
    token_endpoint: "https://auth.example.com/token".to_string(),
    client_id: env::var("CLIENT_ID")?,
    client_secret: env::var("CLIENT_SECRET")?,
    scopes: vec!["read".to_string(), "write".to_string()],
};

let oauth_provider = OAuthProvider::new(oauth_config);
// See examples/servers/complex_auth_sse.rs for complete implementation
```

## パフォーマンスのベスト プラクティス

### Async Operation

non-blocking operation には async / await を使います。

```rust
#[tool]
async fn fetch_data(params: Parameters<FetchParams>) -> Result<String, String> {
    let client = reqwest::Client::new();
    let response = client
        .get(&params.inner().url)
        .send()
        .await
        .map_err(|e| e.to_string())?;

    let text = response.text().await.map_err(|e| e.to_string())?;
    Ok(text)
}
```

### State 管理

共有 state には `Arc` と `RwLock` を使います。

```rust
use std::sync::Arc;
use tokio::sync::RwLock;

pub struct ServerState {
    counter: Arc<RwLock<i32>>,
}

impl ServerState {
    pub fn new() -> Self {
        Self {
            counter: Arc::new(RwLock::new(0)),
        }
    }

    pub async fn increment(&self) -> i32 {
        let mut counter = self.counter.write().await;
        *counter += 1;
        *counter
    }
}
```

## Logging と Tracing

### Tracing を設定する

observability のために tracing を構成します。

```rust
use tracing::{info, warn, error, debug};
use tracing_subscriber;

fn init_logging() {
    tracing_subscriber::fmt()
        .with_max_level(tracing::Level::DEBUG)
        .with_target(false)
        .with_thread_ids(true)
        .init();
}

#[tool]
async fn my_tool(params: Parameters<MyParams>) -> String {
    debug!("Tool called with params: {:?}", params);
    info!("Processing request");

    // Tool logic...

    info!("Request completed");
    "Done".to_string()
}
```

## Deployment

### Binary Distribution

最適化された release binary を build します。

```bash
cargo build --release --target x86_64-unknown-linux-gnu
cargo build --release --target x86_64-pc-windows-msvc
cargo build --release --target x86_64-apple-darwin
```

### Cross-Compilation

cross-platform build には cross を使います。

```bash
cargo install cross
cross build --release --target aarch64-unknown-linux-gnu
```

### Docker Deployment

Dockerfile を作成します。

```dockerfile
FROM rust:1.75 as builder
WORKDIR /app
COPY . .
RUN cargo build --release

FROM debian:bookworm-slim
RUN apt-get update && apt-get install -y ca-certificates
COPY --from=builder /app/target/release/my-mcp-server /usr/local/bin/
CMD ["my-mcp-server"]
```

## 追加リソース

- [rmcp Documentation](https://docs.rs/rmcp)
- [rmcp-macros Documentation](https://docs.rs/rmcp-macros)
- [Examples Repository](https://github.com/modelcontextprotocol/rust-sdk/tree/main/examples)
- [MCP Specification](https://spec.modelcontextprotocol.io/)
- [Rust Async Book](https://rust-lang.github.io/async-book/)
