---
description: '公式 github.com/modelcontextprotocol/go-sdk パッケージを使って Go で Model Context Protocol (MCP) サーバーを構築するためのベストプラクティスとパターン。'
applyTo: "**/*.go, **/go.mod, **/go.sum"
---

# Go MCP サーバー開発ガイドライン

Go で MCP サーバーを構築する際は、公式 Go SDK を使って以下のベストプラクティスとパターンに従ってください。

## サーバー セットアップ

`mcp.NewServer` を使って MCP サーバーを作成します。

```go
import "github.com/modelcontextprotocol/go-sdk/mcp"

server := mcp.NewServer(
    &mcp.Implementation{
        Name:    "my-server",
        Version: "v1.0.0",
    },
    nil, // または mcp.Options を指定
)
```

## Tool の追加

型安全性のために、構造体ベースの入力と出力を使って `mcp.AddTool` を使用します。

```go
type ToolInput struct {
    Query string `json:"query" jsonschema:"the search query"`
    Limit int    `json:"limit,omitempty" jsonschema:"maximum results to return"`
}

type ToolOutput struct {
    Results []string `json:"results" jsonschema:"list of search results"`
    Count   int      `json:"count" jsonschema:"number of results found"`
}

func SearchTool(ctx context.Context, req *mcp.CallToolRequest, input ToolInput) (
    *mcp.CallToolResult,
    ToolOutput,
    error,
) {
    // Tool ロジックを実装
    results := performSearch(ctx, input.Query, input.Limit)

    return nil, ToolOutput{
        Results: results,
        Count:   len(results),
    }, nil
}

// Tool を登録
mcp.AddTool(server,
    &mcp.Tool{
        Name:        "search",
        Description: "Search for information",
    },
    SearchTool,
)
```

## Resource の追加

アクセス可能なデータを提供するには `mcp.AddResource` を使います。

```go
func GetResource(ctx context.Context, req *mcp.ReadResourceRequest) (*mcp.ReadResourceResult, error) {
    content, err := loadResourceContent(ctx, req.URI)
    if err != nil {
        return nil, err
    }

    return &mcp.ReadResourceResult{
        Contents: []any{
            &mcp.TextResourceContents{
                ResourceContents: mcp.ResourceContents{
                    URI:      req.URI,
                    MIMEType: "text/plain",
                },
                Text: content,
            },
        },
    }, nil
}

mcp.AddResource(server,
    &mcp.Resource{
        URI:         "file:///data/example.txt",
        Name:        "Example Data",
        Description: "Example resource data",
        MIMEType:    "text/plain",
    },
    GetResource,
)
```

## Prompt の追加

再利用可能な prompt template には `mcp.AddPrompt` を使います。

```go
type PromptInput struct {
    Topic string `json:"topic" jsonschema:"the topic to analyze"`
}

func AnalyzePrompt(ctx context.Context, req *mcp.GetPromptRequest, input PromptInput) (
    *mcp.GetPromptResult,
    error,
) {
    return &mcp.GetPromptResult{
        Description: "Analyze the given topic",
        Messages: []mcp.PromptMessage{
            {
                Role: mcp.RoleUser,
                Content: mcp.TextContent{
                    Text: fmt.Sprintf("Analyze this topic: %s", input.Topic),
                },
            },
        },
    }, nil
}

mcp.AddPrompt(server,
    &mcp.Prompt{
        Name:        "analyze",
        Description: "Analyze a topic",
        Arguments: []mcp.PromptArgument{
            {
                Name:        "topic",
                Description: "The topic to analyze",
                Required:    true,
            },
        },
    },
    AnalyzePrompt,
)
```

## Transport の設定

### Stdio Transport

stdin/stdout 経由で通信する場合 (desktop integration で最も一般的):

```go
if err := server.Run(ctx, &mcp.StdioTransport{}); err != nil {
    log.Fatal(err)
}
```

### HTTP Transport

HTTP ベースで通信する場合:

```go
import "github.com/modelcontextprotocol/go-sdk/mcp"

transport := &mcp.HTTPTransport{
    Addr: ":8080",
    // 任意: TLS、timeout などを設定
}

if err := server.Run(ctx, transport); err != nil {
    log.Fatal(err)
}
```

## エラー処理

常に適切な error を返し、キャンセルには context を使います。

```go
func MyTool(ctx context.Context, req *mcp.CallToolRequest, input MyInput) (
    *mcp.CallToolResult,
    MyOutput,
    error,
) {
    // context cancellation を確認
    if ctx.Err() != nil {
        return nil, MyOutput{}, ctx.Err()
    }

    // 無効な入力に対して error を返す
    if input.Query == "" {
        return nil, MyOutput{}, fmt.Errorf("query cannot be empty")
    }

    // 処理を実行
    result, err := performOperation(ctx, input)
    if err != nil {
        return nil, MyOutput{}, fmt.Errorf("operation failed: %w", err)
    }

    return nil, result, nil
}
```

## JSON Schema タグ

より良い client integration のために、構造体の説明には `jsonschema` タグを使います。

```go
type Input struct {
    Name     string   `json:"name" jsonschema:"required,description=User's name"`
    Age      int      `json:"age" jsonschema:"minimum=0,maximum=150"`
    Email    string   `json:"email,omitempty" jsonschema:"format=email"`
    Tags     []string `json:"tags,omitempty" jsonschema:"uniqueItems=true"`
    Active   bool     `json:"active" jsonschema:"default=true"`
}
```

## Context の使用

常に context の cancellation と deadline を尊重します。

```go
func LongRunningTool(ctx context.Context, req *mcp.CallToolRequest, input Input) (
    *mcp.CallToolResult,
    Output,
    error,
) {
    select {
    case <-ctx.Done():
        return nil, Output{}, ctx.Err()
    case result := <-performWork(ctx, input):
        return nil, result, nil
    }
}
```

## Server Options

option でサーバー挙動を構成します。

```go
options := &mcp.Options{
    Capabilities: &mcp.ServerCapabilities{
        Tools:     &mcp.ToolsCapability{},
        Resources: &mcp.ResourcesCapability{
            Subscribe: true, // resource subscription を有効化
        },
        Prompts: &mcp.PromptsCapability{},
    },
}

server := mcp.NewServer(
    &mcp.Implementation{Name: "my-server", Version: "v1.0.0"},
    options,
)
```

## テスト

標準的な Go の testing パターンで MCP tool をテストします。

```go
func TestSearchTool(t *testing.T) {
    ctx := context.Background()
    input := ToolInput{Query: "test", Limit: 10}

    result, output, err := SearchTool(ctx, nil, input)
    if err != nil {
        t.Fatalf("SearchTool failed: %v", err)
    }

    if len(output.Results) == 0 {
        t.Error("Expected results, got none")
    }
}
```

## Module Setup

Go module を適切に初期化します。

```bash
go mod init github.com/yourusername/yourserver
go get github.com/modelcontextprotocol/go-sdk@latest
```

`go.mod` には次が含まれているべきです。

```go
module github.com/yourusername/yourserver

go 1.23

require github.com/modelcontextprotocol/go-sdk v1.0.0
```

## よくあるパターン

### Logging

構造化ログを使います。

```go
import "log/slog"

logger := slog.Default()
logger.Info("tool called", "name", req.Params.Name, "args", req.Params.Arguments)
```

### Configuration

環境変数または設定ファイルを使います。

```go
type Config struct {
    ServerName string
    Version    string
    Port       int
}

func LoadConfig() *Config {
    return &Config{
        ServerName: getEnv("SERVER_NAME", "my-server"),
        Version:    getEnv("VERSION", "v1.0.0"),
        Port:       getEnvInt("PORT", 8080),
    }
}
```

### Graceful Shutdown

shutdown signal を適切に処理します。

```go
ctx, cancel := context.WithCancel(context.Background())
defer cancel()

sigCh := make(chan os.Signal, 1)
signal.Notify(sigCh, os.Interrupt, syscall.SIGTERM)

go func() {
    <-sigCh
    cancel()
}()

if err := server.Run(ctx, transport); err != nil {
    log.Fatal(err)
}
```
