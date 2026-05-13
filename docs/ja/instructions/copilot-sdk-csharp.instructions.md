---
applyTo: '**.cs, **.csproj'
description: 'このファイルは、GitHub Copilot SDK を使用して C# アプリケーションを構築するためのガイダンスを提供します。'
name: 'GitHub Copilot SDK C# Instructions'
---

## 基本原則

- SDK はテクニカルプレビュー段階にあり、重大な変更が含まれる可能性があります
- .NET 10.0以降が必要です
- GitHub Copilot CLI がインストールされ、PATH に含まれている必要があります
- 全体的に非同期/待機パターンを使用します
- リソースのクリーンアップのために IAsyncDisposable を実装します。

## インストール

常に NuGet 経由でインストールします。
```bash
dotnet add package GitHub.Copilot.SDK
```

## クライアントの初期化

### 基本的なクライアントのセットアップ

```csharp
await using var client = new CopilotClient();
await client.StartAsync();
```

### クライアント構成オプション

CopilotClient を作成するときは、`CopilotClientOptions` を使用します。

- `CliPath` - CLI 実行可能ファイルへのパス (デフォルト: PATH からの「copilot」)
- `CliArgs` - SDK 管理フラグの前に付加される追加の引数
- `CliUrl` - 既存の CLI サーバーの URL (例: "localhost:8080")。提供された場合、クライアントはプロセスを生成しません
- `Port` - サーバーポート (デフォルト: ランダムの場合は 0)
- `UseStdio` - TCP の代わりに stdio トランスポートを使用します (デフォルト: true)
- `LogLevel` - ログレベル (デフォルト: "info")
- `AutoStart` - 自動起動サーバー (デフォルト: true)
- `AutoRestart` - クラッシュ時の自動再起動 (デフォルト: true)
- `Cwd` - CLI プロセスの作業ディレクトリ
- `Environment` - CLI プロセスの環境変数
- `Logger` - SDK ログ用の ILogger インスタンス

### 手動サーバー制御

明示的な制御の場合:
```csharp
var client = new CopilotClient(new CopilotClientOptions { AutoStart = false });
await client.StartAsync();
// Use client...
await client.StopAsync();
```

`StopAsync()` に時間がかかりすぎる場合は、`ForceStopAsync()` を使用してください。

## セッション管理

### セッションの作成

構成には `SessionConfig` を使用します。

```csharp
await using var session = await client.CreateSessionAsync(new SessionConfig
{
    OnPermissionRequest = PermissionHandler.ApproveAll,
    Model = "gpt-5",
    Streaming = true,
    Tools = [...],
    SystemMessage = new SystemMessageConfig { ... },
    AvailableTools = ["tool1", "tool2"],
    ExcludedTools = ["tool3"],
    Provider = new ProviderConfig { ... }
});
```

### セッション構成オプション

- `SessionId` - カスタムセッションID
- `Model` - モデル名 (「gpt-5」、「claude-sonnet-4.5」など)
- `Tools` - CLI に公開されるカスタムツール
- `SystemMessage` - システムメッセージのカスタマイズ
- `AvailableTools` - ツール名の許可リスト
- `ExcludedTools` - ツール名のブロックリスト
- `Provider` - カスタム API プロバイダー構成 (BYOK)
- `Streaming` - ストリーミング応答チャンクを有効にする (デフォルト: false)

### セッションの再開

```csharp
var session = await client.ResumeSessionAsync(sessionId, new ResumeSessionConfig
{
    OnPermissionRequest = PermissionHandler.ApproveAll,
    // ...
});
```

### セッション操作

- `session.SessionId` - セッション識別子の取得
- `session.SendAsync(new MessageOptions { Prompt = "...", Attachments = [...] })` - メッセージを送信
- `session.AbortAsync()` - 現在の処理を中止します
- `session.GetMessagesAsync()` - すべてのイベント/メッセージを取得します
- `await session.DisposeAsync()` - リソースをクリーンアップする

## イベント処理

### イベントサブスクリプションパターン

セッションイベントの待機には常に TaskCompletionSource を使用してください。

```csharp
var done = new TaskCompletionSource();

session.On(evt =>
{
    if (evt is AssistantMessageEvent msg)
    {
        Console.WriteLine(msg.Data.Content);
    }
    else if (evt is SessionIdleEvent)
    {
        done.SetResult();
    }
});

await session.SendAsync(new MessageOptions { Prompt = "..." });
await done.Task;
```

### イベントの登録解除

`On()` メソッドは IDisposable を返します。

```csharp
var subscription = session.On(evt => { /* handler */ });
// Later...
subscription.Dispose();
```

### イベントの種類

イベント処理にはパターンマッチングまたはスイッチ式を使用します。

```csharp
session.On(evt =>
{
    switch (evt)
    {
        case UserMessageEvent userMsg:
            // Handle user message
            break;
        case AssistantMessageEvent assistantMsg:
            Console.WriteLine(assistantMsg.Data.Content);
            break;
        case ToolExecutionStartEvent toolStart:
            // Tool execution started
            break;
        case ToolExecutionCompleteEvent toolComplete:
            // Tool execution completed
            break;
        case SessionStartEvent start:
            // Session started
            break;
        case SessionIdleEvent idle:
            // Session is idle (processing complete)
            break;
        case SessionErrorEvent error:
            Console.WriteLine($"Error: {error.Data.Message}");
            break;
    }
});
```

## ストリーミング応答

### ストリーミングを有効にする

SessionConfig で `Streaming = true` を設定します。

```csharp
var session = await client.CreateSessionAsync(new SessionConfig
{
    OnPermissionRequest = PermissionHandler.ApproveAll,
    Model = "gpt-5",
    Streaming = true
});
```

### ストリーミングイベントの処理

デルタイベント (増分) と最終イベントの両方を処理します。

```csharp
var done = new TaskCompletionSource();

session.On(evt =>
{
    switch (evt)
    {
        case AssistantMessageDeltaEvent delta:
            // Incremental text chunk
            Console.Write(delta.Data.DeltaContent);
            break;
        case AssistantReasoningDeltaEvent reasoningDelta:
            // Incremental reasoning chunk (model-dependent)
            Console.Write(reasoningDelta.Data.DeltaContent);
            break;
        case AssistantMessageEvent msg:
            // Final complete message
            Console.WriteLine("\n--- Final ---");
            Console.WriteLine(msg.Data.Content);
            break;
        case AssistantReasoningEvent reasoning:
            // Final reasoning content
            Console.WriteLine("--- Reasoning ---");
            Console.WriteLine(reasoning.Data.Content);
            break;
        case SessionIdleEvent:
            done.SetResult();
            break;
    }
});

await session.SendAsync(new MessageOptions { Prompt = "Tell me a story" });
await done.Task;
```

注: 最終イベント (`AssistantMessageEvent`、`AssistantReasoningEvent`) は、ストリーミング設定に関係なく常に送信されます。

## カスタムツール

### AIFunctionFactory を使用したツールの定義

タイプセーフツールには `Microsoft.Extensions.AI.AIFunctionFactory.Create` を使用します。

```csharp
using Microsoft.Extensions.AI;
using System.ComponentModel;

var session = await client.CreateSessionAsync(new SessionConfig
{
    OnPermissionRequest = PermissionHandler.ApproveAll,
    Model = "gpt-5",
    Tools = [
        AIFunctionFactory.Create(
            async ([Description("Issue ID")] string id) => {
                var issue = await FetchIssueAsync(id);
                return issue;
            },
            "lookup_issue",
            "Fetch issue details from tracker"),
    ]
});
```

### ツールの戻り値の型

- 任意の JSON シリアル化可能な値を返します (自動的にラップされます)。
- または、メタデータを完全に制御するには `ToolResultObject` をラップして `ToolResultAIContent` を返します

### ツールの実行フロー

Copilot がツールを呼び出すと、クライアントは自動的に次のことを行います。
1. ハンドラー関数を実行します
2. 戻り値をシリアル化します
3. CLIに応答します

## システムメッセージのカスタマイズ

### 追加モード (デフォルト - ガードレールを保持)

```csharp
var session = await client.CreateSessionAsync(new SessionConfig
{
    OnPermissionRequest = PermissionHandler.ApproveAll,
    Model = "gpt-5",
    SystemMessage = new SystemMessageConfig
    {
        Mode = SystemMessageMode.Append,
        Content = @"
<workflow_rules>
- Always check for security vulnerabilities
- Suggest performance improvements when applicable
</workflow_rules>
"
    }
});
```

### 置換モード (フルコントロール - ガードレールを削除)

```csharp
var session = await client.CreateSessionAsync(new SessionConfig
{
    OnPermissionRequest = PermissionHandler.ApproveAll,
    Model = "gpt-5",
    SystemMessage = new SystemMessageConfig
    {
        Mode = SystemMessageMode.Replace,
        Content = "You are a helpful assistant."
    }
});
```

## 添付ファイル

`UserMessageDataAttachmentsItem` を使用してメッセージにファイルを添付します。

```csharp
await session.SendAsync(new MessageOptions
{
    Prompt = "Analyze this file",
    Attachments = new List<UserMessageDataAttachmentsItem>
    {
        new UserMessageDataAttachmentsItem
        {
            Type = UserMessageDataAttachmentsItemType.File,
            Path = "/path/to/file.cs",
            DisplayName = "My File"
        }
    }
});
```

## メッセージ配信モード

`MessageOptions` で `Mode` プロパティを使用します。

- `"enqueue"` - メッセージを処理のためにキューに入れます
- `"immediate"` - メッセージを直ちに処理します

```csharp
await session.SendAsync(new MessageOptions
{
    Prompt = "...",
    Mode = "enqueue"
});
```

## 複数のセッション

セッションは独立しており、同時に実行できます。

```csharp
var session1 = await client.CreateSessionAsync(new SessionConfig
{
    OnPermissionRequest = PermissionHandler.ApproveAll,
    Model = "gpt-5",
});
var session2 = await client.CreateSessionAsync(new SessionConfig
{
    OnPermissionRequest = PermissionHandler.ApproveAll,
    Model = "claude-sonnet-4.5",
});

await session1.SendAsync(new MessageOptions { Prompt = "Hello from session 1" });
await session2.SendAsync(new MessageOptions { Prompt = "Hello from session 2" });
```

## 自分のキーの持ち込み (BYOK)

`ProviderConfig` 経由でカスタム API プロバイダーを使用します。

```csharp
var session = await client.CreateSessionAsync(new SessionConfig
{
    OnPermissionRequest = PermissionHandler.ApproveAll,
    Provider = new ProviderConfig
    {
        Type = "openai",
        BaseUrl = "https://api.openai.com/v1",
        ApiKey = "your-api-key"
    }
});
```

## セッションのライフサイクル管理

### セッションのリスト表示

```csharp
var sessions = await client.ListSessionsAsync();
foreach (var metadata in sessions)
{
    Console.WriteLine($"Session: {metadata.SessionId}");
}
```

### セッションの削除

```csharp
await client.DeleteSessionAsync(sessionId);
```

### 接続状態の確認

```csharp
var state = client.State;
```

## エラー処理

### 標準例外処理

```csharp
try
{
    var session = await client.CreateSessionAsync(new SessionConfig { OnPermissionRequest = PermissionHandler.ApproveAll });
    await session.SendAsync(new MessageOptions { Prompt = "Hello" });
}
catch (StreamJsonRpc.RemoteInvocationException ex)
{
    Console.Error.WriteLine($"JSON-RPC Error: {ex.Message}");
}
catch (Exception ex)
{
    Console.Error.WriteLine($"Error: {ex.Message}");
}
```

### セッションエラーイベント

`SessionErrorEvent` の実行時エラーを監視します。

```csharp
session.On(evt =>
{
    if (evt is SessionErrorEvent error)
    {
        Console.Error.WriteLine($"Session Error: {error.Data.Message}");
    }
});
```

## 接続テスト

PingAsync を使用してサーバー接続を確認します。

```csharp
var response = await client.PingAsync("test message");
```

## リソースのクリーンアップ

### を使用した自動クリーンアップ

自動破棄には常に `await using` を使用します。

```csharp
await using var client = new CopilotClient();
await using var session = await client.CreateSessionAsync(new SessionConfig { OnPermissionRequest = PermissionHandler.ApproveAll });
// Resources automatically cleaned up
```

### 手動クリーンアップ

`await using` を使用しない場合:

```csharp
var client = new CopilotClient();
try
{
    await client.StartAsync();
    // Use client...
}
finally
{
    await client.StopAsync();
}
```

## ベストプラクティス

1. **CopilotClient と CopilotSession には常に `await using`** を使用してください
2. **TaskCompletionSource を使用**して SessionIdleEvent を待機します
3. **堅牢なエラー処理のために SessionErrorEvent を処理**
4. **イベント処理にはパターンマッチング** (switch 式) を使用します
5. **ストリーミングを有効にする** ことで、インタラクティブなシナリオでの UX を向上させます
6. **タイプセーフなツール定義には AIFunctionFactory を使用します**
7. **不要になったらイベントサブスクリプションを破棄**
8. **SystemMessageMode.Append** を使用して安全ガードレールを維持します
9. **モデルの理解を深めるために、わかりやすいツール名と説明を提供します**
10. **ストリーミングが有効な場合、デルタイベントと最終イベントの両方を処理します**

## よくあるパターン

### 単純なクエリと応答

```csharp
await using var client = new CopilotClient();
await client.StartAsync();

await using var session = await client.CreateSessionAsync(new SessionConfig
{
    OnPermissionRequest = PermissionHandler.ApproveAll,
    Model = "gpt-5"
});

var done = new TaskCompletionSource();

session.On(evt =>
{
    if (evt is AssistantMessageEvent msg)
    {
        Console.WriteLine(msg.Data.Content);
    }
    else if (evt is SessionIdleEvent)
    {
        done.SetResult();
    }
});

await session.SendAsync(new MessageOptions { Prompt = "What is 2+2?" });
await done.Task;
```

### マルチターン会話

```csharp
await using var session = await client.CreateSessionAsync(new SessionConfig { OnPermissionRequest = PermissionHandler.ApproveAll });

async Task SendAndWait(string prompt)
{
    var done = new TaskCompletionSource();
    var subscription = session.On(evt =>
    {
        if (evt is AssistantMessageEvent msg)
        {
            Console.WriteLine(msg.Data.Content);
        }
        else if (evt is SessionIdleEvent)
        {
            done.SetResult();
        }
    });

    await session.SendAsync(new MessageOptions { Prompt = prompt });
    await done.Task;
    subscription.Dispose();
}

await SendAndWait("What is the capital of France?");
await SendAndWait("What is its population?");
```

### 複雑な戻り値の型を持つツール

```csharp
var session = await client.CreateSessionAsync(new SessionConfig
{
    OnPermissionRequest = PermissionHandler.ApproveAll,
    Tools = [
        AIFunctionFactory.Create(
            ([Description("User ID")] string userId) => {
                return new {
                    Id = userId,
                    Name = "John Doe",
                    Email = "john@example.com",
                    Role = "Developer"
                };
            },
            "get_user",
            "Retrieve user information")
    ]
});
```

