---
applyTo: "**.go, go.mod"
description: "このファイルは、GitHub Copilot SDK を使用して Go アプリケーションを構築するためのガイダンスを提供します。"
name: "GitHub Copilot SDK Go Instructions"
---

## 基本原則

- SDK はテクニカルプレビュー段階にあり、重大な変更が含まれる可能性があります
- Go 1.21 以降が必要
- GitHub Copilot CLI がインストールされ、PATH に含まれている必要があります
- 同時操作にはゴルーチンとチャネルを使用します
- 標準ライブラリ以外の外部依存関係はありません

## インストール

常に Go モジュール経由でインストールします。

```bash
go get github.com/github/copilot-sdk/go
```

## クライアントの初期化

### 基本的なクライアントのセットアップ

```go
import "github.com/github/copilot-sdk/go"

client := copilot.NewClient(nil)
if err := client.Start(); err != nil {
    log.Fatal(err)
}
defer client.Stop()
```

### クライアント構成オプション

CopilotClient を作成するときは、`ClientOptions` を使用します。

- `CLIPath` - CLI 実行可能ファイルへのパス (デフォルト: PATH からの「copilot」)
- `CLIUrl` - 既存の CLI サーバーの URL (例: "localhost:8080")。提供された場合、クライアントはプロセスを生成しません
- `Port` - サーバーポート (デフォルト: ランダムの場合は 0)
- `UseStdio` - TCP の代わりに stdio トランスポートを使用します (デフォルト: true)
- `LogLevel` - ログレベル (デフォルト: "info")
- `AutoStart` - サーバーの自動起動 (デフォルト: true、ポインタを使用: `boolPtr(true)`)
- `AutoRestart` - クラッシュ時の自動再起動 (デフォルト: true、ポインタを使用: `boolPtr(true)`)
- `Cwd` - CLI プロセスの作業ディレクトリ
- `Env` - CLI プロセスの環境変数 ([]string)

### 手動サーバー制御

明示的な制御の場合:

```go
autoStart := false
client := copilot.NewClient(&copilot.ClientOptions{AutoStart: &autoStart})
if err := client.Start(); err != nil {
    log.Fatal(err)
}
// Use client...
client.Stop()
```

`Stop()` に時間がかかりすぎる場合は、`ForceStop()` を使用してください。

## セッション管理

### セッションの作成

設定には `SessionConfig` を使用します。

```go
session, err := client.CreateSession(&copilot.SessionConfig{
	OnPermissionRequest: copilot.PermissionHandler.ApproveAll,
    Model: "gpt-5",
    Streaming: true,
    Tools: []copilot.Tool{...},
    SystemMessage: &copilot.SystemMessageConfig{ ... },
    AvailableTools: []string{"tool1", "tool2"},
    ExcludedTools: []string{"tool3"},
    Provider: &copilot.ProviderConfig{ ... },
})
if err != nil {
    log.Fatal(err)
}
```

### セッション構成オプション

- `SessionID` - カスタムセッションID
- `Model` - モデル名 (「gpt-5」、「claude-sonnet-4.5」など)
- `Tools` - CLI に公開されるカスタムツール ([]Tool)
- `SystemMessage` - システムメッセージのカスタマイズ (\*SystemMessageConfig)
- `AvailableTools` - ツール名の許可リスト ([]文字列)
- `ExcludedTools` - ツール名のブロックリスト ([]文字列)
- `Provider` - カスタム API プロバイダー構成 (BYOK) (\*ProviderConfig)
- `Streaming` - ストリーミング応答チャンクを有効にする (ブール値)
- `MCPServers` - MCP サーバー構成
- `CustomAgents` - カスタムエージェント構成
- `ConfigDir` - 構成ディレクトリの上書き
- `SkillDirectories` - スキルディレクトリ ([]文字列)
- `DisabledSkills` - 無効化されたスキル ([]文字列)

### セッションの再開

```go
session, err := client.ResumeSession("session-id", &copilot.ResumeSessionConfig{OnPermissionRequest: copilot.PermissionHandler.ApproveAll})
// Or with options:
session, err := client.ResumeSessionWithOptions("session-id", &copilot.ResumeSessionConfig{ ... })
```

### セッション操作

- `session.SessionID` - セッション識別子（文字列）を取得します
- `session.Send(copilot.MessageOptions{Prompt: "...", Attachments: []copilot.Attachment{...}})` - メッセージを送信し、(メッセージ ID 文字列、エラー) を返します。
- `session.SendAndWait(options, timeout)` - 送信してアイドル状態になるまで待機し、(\*SessionEvent、エラー) を返します。
- `session.Abort()` - 現在の処理を中止し、エラーを返します
- `session.GetMessages()` - すべてのイベント/メッセージを取得し、([]SessionEvent、エラー) を返します。
- `session.Destroy()` - セッションをクリーンアップし、エラーを返します

## イベント処理

### イベントサブスクリプションパターン

セッションイベントを待機するには、常にチャネルまたは完了シグナルを使用してください。

```go
done := make(chan struct{})

unsubscribe := session.On(func(evt copilot.SessionEvent) {
    switch evt.Type {
    case copilot.AssistantMessage:
        fmt.Println(*evt.Data.Content)
    case copilot.SessionIdle:
        close(done)
    }
})
defer unsubscribe()

session.Send(copilot.MessageOptions{Prompt: "..."})
<-done
```

### イベントの登録解除

`On()` メソッドは、サブスクライブを解除する関数を返します。

```go
unsubscribe := session.On(func(evt copilot.SessionEvent) {
    // handler
})
// Later...
unsubscribe()
```

### イベントの種類

イベント処理にはタイプスイッチを使用します。

```go
session.On(func(evt copilot.SessionEvent) {
    switch evt.Type {
    case copilot.UserMessage:
        // Handle user message
    case copilot.AssistantMessage:
        if evt.Data.Content != nil {
            fmt.Println(*evt.Data.Content)
        }
    case copilot.ToolExecutionStart:
        // Tool execution started
    case copilot.ToolExecutionComplete:
        // Tool execution completed
    case copilot.SessionStart:
        // Session started
    case copilot.SessionIdle:
        // Session is idle (processing complete)
    case copilot.SessionError:
        if evt.Data.Message != nil {
            fmt.Println("Error:", *evt.Data.Message)
        }
    }
})
```

## ストリーミング応答

### ストリーミングを有効にする

SessionConfig で `Streaming: true` を設定します。

```go
session, err := client.CreateSession(&copilot.SessionConfig{
	OnPermissionRequest: copilot.PermissionHandler.ApproveAll,
    Model: "gpt-5",
    Streaming: true,
})
```

### ストリーミングイベントの処理

デルタイベント (増分) と最終イベントの両方を処理します。

```go
done := make(chan struct{})

session.On(func(evt copilot.SessionEvent) {
    switch evt.Type {
    case copilot.AssistantMessageDelta:
        // Incremental text chunk
        if evt.Data.DeltaContent != nil {
            fmt.Print(*evt.Data.DeltaContent)
        }
    case copilot.AssistantReasoningDelta:
        // Incremental reasoning chunk (model-dependent)
        if evt.Data.DeltaContent != nil {
            fmt.Print(*evt.Data.DeltaContent)
        }
    case copilot.AssistantMessage:
        // Final complete message
        fmt.Println("\n--- Final ---")
        if evt.Data.Content != nil {
            fmt.Println(*evt.Data.Content)
        }
    case copilot.AssistantReasoning:
        // Final reasoning content
        fmt.Println("--- Reasoning ---")
        if evt.Data.Content != nil {
            fmt.Println(*evt.Data.Content)
        }
    case copilot.SessionIdle:
        close(done)
    }
})

session.Send(copilot.MessageOptions{Prompt: "Tell me a story"})
<-done
```

注: 最終イベント (`AssistantMessage`、`AssistantReasoning`) は、ストリーミング設定に関係なく常に送信されます。

## カスタムツール

### ツールの定義

```go
session, err := client.CreateSession(&copilot.SessionConfig{
	OnPermissionRequest: copilot.PermissionHandler.ApproveAll,
    Model: "gpt-5",
    Tools: []copilot.Tool{
        {
            Name:        "lookup_issue",
            Description: "Fetch issue details from tracker",
            Parameters: map[string]interface{}{
                "type": "object",
                "properties": map[string]interface{}{
                    "id": map[string]interface{}{
                        "type":        "string",
                        "description": "Issue ID",
                    },
                },
                "required": []string{"id"},
            },
            Handler: func(inv copilot.ToolInvocation) (copilot.ToolResult, error) {
                args := inv.Arguments.(map[string]interface{})
                issueID := args["id"].(string)

                issue, err := fetchIssue(issueID)
                if err != nil {
                    return copilot.ToolResult{}, err
                }

                return copilot.ToolResult{
                    TextResultForLLM: fmt.Sprintf("Issue: %v", issue),
                    ResultType:       "success",
                    ToolTelemetry:    map[string]interface{}{},
                }, nil
            },
        },
    },
})
```

### ツールの戻り値の型

- フィールドを含む `ToolResult` 構造体を返します。
  - `TextResultForLLM` (文字列) - LLM の結果テキスト
  - `ResultType` (文字列) - 「成功」または「失敗」
  - `Error` (文字列、オプション) - 内部エラーメッセージ (LLM には表示されません)
  - `ToolTelemetry` (マップ[文字列]インターフェース{}) - テレメトリデータ

### ツールの実行フロー

Copilot がツールを呼び出すと、クライアントは自動的に次のことを行います。

1. ハンドラー関数を実行します
2. ToolResult を返します
3. CLIに応答します

## システムメッセージのカスタマイズ

### 追加モード (デフォルト - ガードレールを保持)

```go
session, err := client.CreateSession(&copilot.SessionConfig{
	OnPermissionRequest: copilot.PermissionHandler.ApproveAll,
    Model: "gpt-5",
    SystemMessage: &copilot.SystemMessageConfig{
        Mode: "append",
        Content: `
<workflow_rules>
- Always check for security vulnerabilities
- Suggest performance improvements when applicable
</workflow_rules>
`,
    },
})
```

### 置換モード (フルコントロール - ガードレールを削除)

```go
session, err := client.CreateSession(&copilot.SessionConfig{
	OnPermissionRequest: copilot.PermissionHandler.ApproveAll,
    Model: "gpt-5",
    SystemMessage: &copilot.SystemMessageConfig{
        Mode:    "replace",
        Content: "You are a helpful assistant.",
    },
})
```

## 添付ファイル

`Attachment` を使用してメッセージにファイルを添付します。

```go
messageID, err := session.Send(copilot.MessageOptions{
    Prompt: "Analyze this file",
    Attachments: []copilot.Attachment{
        {
            Type:        "file",
            Path:        "/path/to/file.go",
            DisplayName: "My File",
        },
    },
})
```

## メッセージ配信モード

`MessageOptions` の `Mode` フィールドを使用します。

- `"enqueue"` - メッセージを処理のためにキューに入れます
- `"immediate"` - メッセージを直ちに処理します

```go
session.Send(copilot.MessageOptions{
    Prompt: "...",
    Mode:   "enqueue",
})
```

## 複数のセッション

セッションは独立しており、同時に実行できます。

```go
session1, _ := client.CreateSession(&copilot.SessionConfig{
	OnPermissionRequest: copilot.PermissionHandler.ApproveAll,
	Model:               "gpt-5",
})
session2, _ := client.CreateSession(&copilot.SessionConfig{
	OnPermissionRequest: copilot.PermissionHandler.ApproveAll,
	Model:               "claude-sonnet-4.5",
})

session1.Send(copilot.MessageOptions{Prompt: "Hello from session 1"})
session2.Send(copilot.MessageOptions{Prompt: "Hello from session 2"})
```

## 自分のキーの持ち込み (BYOK)

`ProviderConfig` を構成してカスタム API プロバイダーを使用します。

```go
session, err := client.CreateSession(&copilot.SessionConfig{
	OnPermissionRequest: copilot.PermissionHandler.ApproveAll,
    Provider: &copilot.ProviderConfig{
        Type:    "openai",
        BaseURL: "https://api.openai.com/v1",
        APIKey:  "your-api-key",
    },
})
```

## セッションのライフサイクル管理

### 接続状態の確認

```go
state := client.GetState()
// Returns: "disconnected", "connecting", "connected", or "error"
```

## エラー処理

### 標準例外処理

```go
session, err := client.CreateSession(&copilot.SessionConfig{
	OnPermissionRequest: copilot.PermissionHandler.ApproveAll,
})
if err != nil {
    log.Fatalf("Failed to create session: %v", err)
}

_, err = session.Send(copilot.MessageOptions{Prompt: "Hello"})
if err != nil {
    log.Printf("Failed to send: %v", err)
}
```

### セッションエラーイベント

`SessionError` タイプの実行時エラーを監視します。

```go
session.On(func(evt copilot.SessionEvent) {
    if evt.Type == copilot.SessionError {
        if evt.Data.Message != nil {
            fmt.Fprintf(os.Stderr, "Session Error: %s\n", *evt.Data.Message)
        }
    }
})
```

## 接続テスト

`Ping` を使用してサーバーの接続を確認します。

```go
resp, err := client.Ping("test message")
if err != nil {
    log.Printf("Server unreachable: %v", err)
} else {
    log.Printf("Server responded at %d", resp.Timestamp)
}
```

## リソースのクリーンアップ

### 遅延によるクリーンアップ

クリーンアップには常に `defer` を使用してください。

```go
client := copilot.NewClient(nil)
if err := client.Start(); err != nil {
    log.Fatal(err)
}
defer client.Stop()

session, err := client.CreateSession(&copilot.SessionConfig{OnPermissionRequest: copilot.PermissionHandler.ApproveAll})
if err != nil {
    log.Fatal(err)
}
defer session.Destroy()
```

### 手動クリーンアップ

遅延を使用しない場合:

```go
client := copilot.NewClient(nil)
err := client.Start()
if err != nil {
    log.Fatal(err)
}

session, err := client.CreateSession(&copilot.SessionConfig{OnPermissionRequest: copilot.PermissionHandler.ApproveAll})
if err != nil {
    client.Stop()
    log.Fatal(err)
}

// Use session...

session.Destroy()
errors := client.Stop()
for _, err := range errors {
    log.Printf("Cleanup error: %v", err)
}
```

## ベストプラクティス

1. **クライアントとセッションのクリーンアップには常に `defer`** を使用してください
2. **チャンネルを使用**してSessionIdleイベントを待機します
3. **SessionError** イベントを処理して堅牢なエラー処理を行う
4. **イベント処理にはタイプスイッチを使用**
5. **ストリーミングを有効にする** ことで、インタラクティブなシナリオでの UX を向上させます
6. **モデルの理解を深めるために、わかりやすいツール名と説明を提供します**
7. **不要になった場合は、サブスクライブ解除関数を呼び出します**
8. **安全ガードレールを維持するには、SystemMessageConfig をモード「append」で使用します**
9. **ストリーミングが有効な場合、デルタイベントと最終イベントの両方を処理します**
10. イベントデータ内の **nil ポインターをチェックします** (コンテンツ、メッセージなどはポインターです)

## よくあるパターン

### 単純なクエリと応答

```go
client := copilot.NewClient(nil)
if err := client.Start(); err != nil {
    log.Fatal(err)
}
defer client.Stop()

session, err := client.CreateSession(&copilot.SessionConfig{
	OnPermissionRequest: copilot.PermissionHandler.ApproveAll,
	Model:               "gpt-5",
})
if err != nil {
    log.Fatal(err)
}
defer session.Destroy()

done := make(chan struct{})

session.On(func(evt copilot.SessionEvent) {
    if evt.Type == copilot.AssistantMessage && evt.Data.Content != nil {
        fmt.Println(*evt.Data.Content)
    } else if evt.Type == copilot.SessionIdle {
        close(done)
    }
})

session.Send(copilot.MessageOptions{Prompt: "What is 2+2?"})
<-done
```

### マルチターン会話

```go
session, _ := client.CreateSession(&copilot.SessionConfig{OnPermissionRequest: copilot.PermissionHandler.ApproveAll})
defer session.Destroy()

sendAndWait := func(prompt string) error {
    done := make(chan struct{})
    var eventErr error

    unsubscribe := session.On(func(evt copilot.SessionEvent) {
        switch evt.Type {
        case copilot.AssistantMessage:
            if evt.Data.Content != nil {
                fmt.Println(*evt.Data.Content)
            }
        case copilot.SessionIdle:
            close(done)
        case copilot.SessionError:
            if evt.Data.Message != nil {
                eventErr = fmt.Errorf(*evt.Data.Message)
            }
        }
    })
    defer unsubscribe()

    if _, err := session.Send(copilot.MessageOptions{Prompt: prompt}); err != nil {
        return err
    }
    <-done
    return eventErr
}

sendAndWait("What is the capital of France?")
sendAndWait("What is its population?")
```

### SendAndWait ヘルパー

```go
// Use built-in SendAndWait for simpler synchronous interaction
response, err := session.SendAndWait(copilot.MessageOptions{
    Prompt: "What is 2+2?",
}, 0) // 0 uses default 60s timeout

if err != nil {
    log.Printf("Error: %v", err)
}
if response != nil && response.Data.Content != nil {
    fmt.Println(*response.Data.Content)
}
```

### 構造体の戻り型を持つツール

```go
type UserInfo struct {
    ID    string `json:"id"`
    Name  string `json:"name"`
    Email string `json:"email"`
    Role  string `json:"role"`
}

session, _ := client.CreateSession(&copilot.SessionConfig{
	OnPermissionRequest: copilot.PermissionHandler.ApproveAll,
    Tools: []copilot.Tool{
        {
            Name:        "get_user",
            Description: "Retrieve user information",
            Parameters: map[string]interface{}{
                "type": "object",
                "properties": map[string]interface{}{
                    "user_id": map[string]interface{}{
                        "type":        "string",
                        "description": "User ID",
                    },
                },
                "required": []string{"user_id"},
            },
            Handler: func(inv copilot.ToolInvocation) (copilot.ToolResult, error) {
                args := inv.Arguments.(map[string]interface{})
                userID := args["user_id"].(string)

                user := UserInfo{
                    ID:    userID,
                    Name:  "John Doe",
                    Email: "john@example.com",
                    Role:  "Developer",
                }

                jsonBytes, _ := json.Marshal(user)
                return copilot.ToolResult{
                    TextResultForLLM: string(jsonBytes),
                    ResultType:       "success",
                    ToolTelemetry:    map[string]interface{}{},
                }, nil
            },
        },
    },
})
```
