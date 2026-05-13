---
name: go-mcp-server-generator
description: '公式 github.com/modelcontextprotocol/go-sdk を使用して、適切な構成・依存関係・実装を備えた完全な Go MCP サーバープロジェクトを生成します。'
---

# Go MCP サーバープロジェクトジェネレーター

Go で、本番運用可能な完全な Model Context Protocol (MCP) サーバープロジェクトを生成します。

## プロジェクト要件

以下を備えた Go MCP サーバーを作成します:

1. **プロジェクト構成**: 適切な Go モジュールレイアウト
2. **依存関係**: 公式 MCP SDK と必要なパッケージ
3. **サーバー設定**: トランスポートを含む MCP サーバー設定
4. **ツール**: 型付きの入力/出力を持つ、実用的なツールを少なくとも 2〜3 個
5. **エラーハンドリング**: 適切なエラー処理と context の利用
6. **ドキュメント**: セットアップと使用方法を記載した README
7. **テスト**: 基本的なテスト構成

## テンプレート構成

```
myserver/
├── go.mod
├── go.sum
├── main.go
├── tools/
│   ├── tool1.go
│   └── tool2.go
├── resources/
│   └── resource1.go
├── config/
│   └── config.go
├── README.md
└── main_test.go
```

## go.mod テンプレート

```go
module github.com/yourusername/{{PROJECT_NAME}}

go 1.23

require (
    github.com/modelcontextprotocol/go-sdk v1.0.0
)
```

## main.go テンプレート

```go
package main

import (
    "context"
    "log"
    "os"
    "os/signal"
    "syscall"

    "github.com/modelcontextprotocol/go-sdk/mcp"
    "github.com/yourusername/{{PROJECT_NAME}}/config"
    "github.com/yourusername/{{PROJECT_NAME}}/tools"
)

func main() {
    cfg := config.Load()
    
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()

    // 正常終了シーケンスを処理
    sigCh := make(chan os.Signal, 1)
    signal.Notify(sigCh, os.Interrupt, syscall.SIGTERM)
    go func() {
        <-sigCh
        log.Println("シャットダウン中...")
        cancel()
    }()

    // サーバーを作成
    server := mcp.NewServer(
        &mcp.Implementation{
            Name:    cfg.ServerName,
            Version: cfg.Version,
        },
        &mcp.Options{
            Capabilities: &mcp.ServerCapabilities{
                Tools:     &mcp.ToolsCapability{},
                Resources: &mcp.ResourcesCapability{},
                Prompts:   &mcp.PromptsCapability{},
            },
        },
    )

    // ツールを登録
    tools.RegisterTools(server)

    // サーバーを実行
    transport := &mcp.StdioTransport{}
    if err := server.Run(ctx, transport); err != nil {
        log.Fatalf("サーバーエラー: %v", err)
    }
}
```

## tools/tool1.go テンプレート

```go
package tools

import (
    "context"
    "fmt"

    "github.com/modelcontextprotocol/go-sdk/mcp"
)

type Tool1Input struct {
    Param1 string `json:"param1" jsonschema:"required,description=最初のパラメータ"`
    Param2 int    `json:"param2,omitempty" jsonschema:"description=任意の2番目のパラメータ"`
}

type Tool1Output struct {
    Result string `json:"result" jsonschema:"description=処理結果"`
    Status string `json:"status" jsonschema:"description=処理ステータス"`
}

func Tool1Handler(ctx context.Context, req *mcp.CallToolRequest, input Tool1Input) (
    *mcp.CallToolResult,
    Tool1Output,
    error,
) {
    // 入力を検証
    if input.Param1 == "" {
        return nil, Tool1Output{}, fmt.Errorf("param1 は必須です")
    }

    // context を確認
    if ctx.Err() != nil {
        return nil, Tool1Output{}, ctx.Err()
    }

    // 処理を実行
    result := fmt.Sprintf("処理済み: %s", input.Param1)

    return nil, Tool1Output{
        Result: result,
        Status: "success",
    }, nil
}

func RegisterTool1(server *mcp.Server) {
    mcp.AddTool(server,
        &mcp.Tool{
            Name:        "tool1",
            Description: "tool1 が何をするかの説明",
        },
        Tool1Handler,
    )
}
```

## tools/registry.go テンプレート

```go
package tools

import "github.com/modelcontextprotocol/go-sdk/mcp"

func RegisterTools(server *mcp.Server) {
    RegisterTool1(server)
    RegisterTool2(server)
    // 追加のツールをここで登録
}
```

## config/config.go テンプレート

```go
package config

import "os"

type Config struct {
    ServerName string
    Version    string
    LogLevel   string
}

func Load() *Config {
    return &Config{
        ServerName: getEnv("SERVER_NAME", "{{PROJECT_NAME}}"),
        Version:    getEnv("VERSION", "v1.0.0"),
        LogLevel:   getEnv("LOG_LEVEL", "info"),
    }
}

func getEnv(key, defaultValue string) string {
    if value := os.Getenv(key); value != "" {
        return value
    }
    return defaultValue
}
```

## main_test.go テンプレート

```go
package main

import (
    "context"
    "testing"

    "github.com/yourusername/{{PROJECT_NAME}}/tools"
)

func TestTool1Handler(t *testing.T) {
    ctx := context.Background()
    input := tools.Tool1Input{
        Param1: "test",
        Param2: 42,
    }

    result, output, err := tools.Tool1Handler(ctx, nil, input)
    if err != nil {
        t.Fatalf("Tool1Handler が失敗しました: %v", err)
    }

    if output.Status != "success" {
        t.Errorf("ステータスは 'success' の想定ですが、実際は '%s' でした", output.Status)
    }

    if result != nil {
        t.Error("result は nil の想定です")
    }
}
```

## README.md テンプレート

```markdown
# {{PROJECT_NAME}}

Go で構築された Model Context Protocol (MCP) サーバーです。

## 説明

{{PROJECT_DESCRIPTION}}

## インストール

\`\`\`bash
go mod download
go build -o {{PROJECT_NAME}}
\`\`\`

## 使い方

stdio トランスポートでサーバーを実行します:

\`\`\`bash
./{{PROJECT_NAME}}
\`\`\`

## 設定

環境変数で設定します:

- `SERVER_NAME`: サーバー名（デフォルト: "{{PROJECT_NAME}}")
- `VERSION`: サーバーバージョン（デフォルト: "v1.0.0")
- `LOG_LEVEL`: ログレベル（デフォルト: "info")

## 利用可能なツール

### tool1
{{TOOL1_DESCRIPTION}}

**入力:**
- `param1` (string, 必須): 最初のパラメータ
- `param2` (int, 任意): 2番目のパラメータ

**出力:**
- `result` (string): 処理結果
- `status` (string): 処理のステータス

## 開発

テスト実行:

\`\`\`bash
go test ./...
\`\`\`

ビルド:

\`\`\`bash
go build -o {{PROJECT_NAME}}
\`\`\`

## ライセンス

MIT
```

## 生成手順

Go MCP サーバーを生成する際は、以下に従ってください:

1. **モジュール初期化**: 適切なモジュールパスで `go.mod` を作成
2. **構成**: テンプレートのディレクトリ構成に従う
3. **型安全性**: すべての入出力で JSON schema タグ付き struct を使う
4. **エラーハンドリング**: 入力検証、context 確認、エラーのラップを行う
5. **ドキュメント**: 明確な説明と例を追加する
6. **テスト**: ツールごとに少なくとも 1 つのテストを含める
7. **設定**: config には環境変数を使う
8. **ロギング**: 構造化ログ（log/slog）を使う
9. **正常終了**: シグナルを適切に処理する
10. **トランスポート**: デフォルトは stdio とし、代替手段を文書化する

## ベストプラクティス

- ツールは焦点を絞り、単一責務にする
- 型名と関数名は説明的にする
- struct タグに JSON schema のドキュメントを含める
- context のキャンセルは常に尊重する
- 説明的なエラーを返す
- main.go は最小限にし、ロジックはパッケージに分離する
- ツールハンドラのテストを書く
- すべての公開関数をドキュメント化する

