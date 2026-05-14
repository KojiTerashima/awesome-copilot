---
description: 'C# SDK を使用してモデルコンテキスト プロトコル (MCP) サーバーを構築する手順'
applyTo: '**/*.cs, **/*.csproj'
---

# C# MCP サーバー開発

## 説明書

- ほとんどのプロジェクトには **ModelContextProtocol** NuGet パッケージ (プレリリース) を使用してください: `dotnet add package ModelContextProtocol --prerelease`
- HTTP ベースの MCP サーバーには **ModelContextProtocol.AspNetCore** を使用します
- 依存関係を最小限に抑えるには **ModelContextProtocol.Core** を使用します (クライアント専用または低レベルのサーバー API)
- stdio トランスポートの干渉を避けるために、常に `LogToStandardErrorThreshold = LogLevel.Trace` を使用して stderr へのロギングを設定してください。
- MCP ツールを含むクラスで `[McpServerToolType]` 属性を使用する
- メソッドの `[McpServerTool]` 属性を使用してメソッドをツールとして公開します
- `System.ComponentModel` の `[Description]` 属性を使用してツールとパラメータを文書化します
- ツールメソッドで依存関係の注入をサポート - `McpServer`、`HttpClient`、またはその他のサービスをパラメータとして注入
- `McpServer.AsSamplingChatClient()` を使用して、ツール内からクライアントにサンプリングリクエストを返します。
- クラスでは `[McpServerPromptType]`、メソッドでは `[McpServerPrompt]` を使用してプロンプトを公開する
- 標準入出力トランスポートの場合は、サーバーを構築するときに `WithStdioServerTransport()` を使用します
- `WithToolsFromAssembly()` を使用して、現在のアセンブリからすべてのツールを自動検出して登録します
- ツールのメソッドは同期または非同期にすることができます (`Task` または `Task<T>` を返す)
- LLM がその目的を理解できるように、ツールとパラメータの包括的な説明を常に含めてください。
- 適切なキャンセルをサポートするには、非同期ツールで `CancellationToken` パラメーターを使用します
- JSON にシリアル化できる単純な型 (string、int など) または複雑なオブジェクトを返します。
- きめ細かい制御を行うには、`ListToolsHandler` や `CallToolHandler` などのカスタムハンドラーとともに `McpServerOptions` を使用します。
- プロトコルレベルのエラーには、適切な `McpErrorCode` 値を使用して `McpProtocolException` を使用します
- 同じ SDK または準拠する MCP クライアントの `McpClient` を使用して MCP サーバーをテストします
- Microsoft.Extensions.Hosting を使用してプロジェクトを構築し、適切な DI およびライフサイクル管理を実現する

## ベストプラクティス

- ツールのメソッドを焦点を絞って単一目的に保つ
- 機能を明確に示す意味のあるツール名を使用してください
- ツールの機能、期待されるパラメーター、およびツールが何を返すかを説明する詳細な説明を提供します。
- 入力パラメータを検証し、無効な入力に対して `McpProtocolException` を `McpErrorCode.InvalidParams` とともにスローします
- 構造化ログを使用して、標準出力を汚染することなくデバッグを支援します
- `[McpServerToolType]` を使用して関連ツールを論理クラスに整理します
- 外部リソースにアクセスするツールを公開する場合は、セキュリティへの影響を考慮する
- 組み込みの DI コンテナを使用してサービスの有効期間と依存関係を管理する
- 適切なエラー処理を実装し、意味のあるエラーメッセージを返す
- LLM と統合する前にツールを個別にテストする

## よくあるパターン

### 基本的なサーバーのセットアップ
```csharp
var builder = Host.CreateApplicationBuilder(args);
builder.Logging.AddConsole(options => 
    options.LogToStandardErrorThreshold = LogLevel.Trace);
builder.Services
    .AddMcpServer()
    .WithStdioServerTransport()
    .WithToolsFromAssembly();
await builder.Build().RunAsync();
```

### シンプルなツール
```csharp
[McpServerToolType]
public static class MyTools
{
    [McpServerTool, Description("Description of what the tool does")]
    public static string ToolName(
        [Description("Parameter description")] string param) => 
        $"Result: {param}";
}
```

### 依存性注入を備えたツール
```csharp
[McpServerTool, Description("Fetches data from a URL")]
public static async Task<string> FetchData(
    HttpClient httpClient,
    [Description("The URL to fetch")] string url,
    CancellationToken cancellationToken) =>
    await httpClient.GetStringAsync(url, cancellationToken);
```

### サンプリング付きツール
```csharp
[McpServerTool, Description("Analyzes content using the client's LLM")]
public static async Task<string> Analyze(
    McpServer server,
    [Description("Content to analyze")] string content,
    CancellationToken cancellationToken)
{
    var messages = new ChatMessage[]
    {
        new(ChatRole.User, $"Analyze this: {content}")
    };
    return await server.AsSamplingChatClient()
        .GetResponseAsync(messages, cancellationToken: cancellationToken);
}
```
