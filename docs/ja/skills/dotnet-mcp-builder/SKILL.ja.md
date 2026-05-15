---
name: dotnet-mcp-builder
description: '現在の ModelContextProtocol 1.x NuGet パッケージに対して C#/.NET でモデル コンテキスト プロトコル (MCP) サーバーを構築します。特に、ガイダンスなしでモデルがよく間違っている場合に役立ちます。古いプレビュー バージョン (0.3 または 0.4 プレビューが選択される傾向があります)、MCP アプリ (ホストでレンダリングされるインタラクティブ UI)、引き出し URL モード、セッションごとの HTTP ワイヤリング、OAuth およびリバース プロキシ デプロイの詳細、具体的な MapMcp / STDIO / Streamable-HTTP エラーのデバッグなどです。また、日常的な作業 (STDIO およびストリーミング可能な HTTP トランスポート (SSE は非推奨)、ツール、プロンプト、リソース、サンプリング、ルート、補完、ロギング)、および基本的な .NET MCP クライアントについても説明します。ユーザーが .NET MCP サーバーの動作を発言または示唆した場合にトリガーされます: ModelContextProtocol、McpServerTool、MapMcp、WithStdioServerTransport、「C# の MCP サーバー」、「ドットネットの MCP ツール」、「これを MCP として公開する」、または .NET コンテキストでプリミティブ (プロンプト/リソース/誘導/MCP アプリ) に名前を付けます。他の言語での MCP 作業についてはスキップしてください。'
---

# .NET での MCP サーバーの構築

このスキルは、Microsoft と MCP プロジェクトによって保守されている **公式** [`ModelContextProtocol`](https://www.nuget.org/profiles/ModelContextProtocol) NuGet パッケージに対して C#/.NET で運用品質の MCP サーバーと基本的なクライアントを作成するのに役立ちます。 **安定版 1.x** ラインと現在の仕様 (2025 年 11 月 25 日) を対象としています。

## このスキルが維持されると

.NET MCP SDK には、`1.0` に到達するまでに何年もプレビュー パッケージ (`0.x-preview`) がありました。助けがなければ、モデルは次のような傾向があります。
- 現在のサンプルに対してコンパイルできない古いプレビュー バージョンを固定します。
- 最近の仕様機能 (引き出し URL モード、MCP アプリ、構造化コンテンツ ブロック) がありません。
- HTTP トランスポートの詳細が間違っている (ステートフル/ステートレス、プロキシ バッファリング、OAuth ワイヤリング)。
- STDIO stdout/stderr トラップは無視してください。

タスクがそれらのいずれかである場合は、*一致する参照をロード*し、それに従ってください。本当に簡単な場合 (例: 「このツールのメソッドの名前を変更する」)、すべてを読む必要はありません。以下の基本ルールは最低限のものです。

## 30秒でわかるメンタルモデル

.NET MCP サーバーは、DI を介して MCP サーバーに接続する通常の `Microsoft.Extensions.Hosting` (または `WebApplication`) アプリです。
```csharp
builder.Services
    .AddMcpServer()
    .WithStdioServerTransport()      // OR .WithHttpTransport(...)
    .WithToolsFromAssembly()         // discover [McpServerToolType] classes
    .WithPrompts<MyPrompts>()        // optional
    .WithResources<MyResources>();   // optional
```

プリミティブは、属性 (`[McpServerToolType]` + `[McpServerTool]`、`[McpServerPromptType]` + `[McpServerPrompt]`、`[McpServerResourceType]` + `[McpServerResource]`) でマークされたクラスのプレーンな C# メソッドです。パラメータは JSON-RPC からバインドされます。 SDK は、署名と `[Description]` 属性から JSON スキーマを構築します。

サーバーからクライアントへの機能 (サンプリング、抽出、ルート、ログ/進捗通知) は、注入された `IMcpServer` のメソッドです。

## デシジョン ツリー → ロードする参照先

新しいプロジェクトを作成している場合、または現在のパッケージのバージョンが不明な場合は、常に `references/packages.md` をロードしてください。

|タスク |ロード |
|---|---|
|新しい STDIO サーバー | `references/transport-stdio.md` |
|新しい HTTP (ストリーミング可能) サーバー | `references/transport-http.md` |
|ツールの追加/変更 | `references/tool-primitive.md` |
|プロンプトを追加/変更する | `references/prompt-primitive.md` |
|リソースの追加/変更 | `references/resource-primitive.md` |
|ツールの途中でユーザーに質問する | `references/elicitation.md` |
|ツールからクライアントの LLM を呼び出す | `references/sampling.md` |
|ユーザーのプロジェクト ルートを読み取る | `references/roots.md` |
|インタラクティブな UI を返す | `references/mcp-apps.md` |
|引数の完了、ログ/進捗通知、フィルター、サーバー命令 | `references/server-features.md` |
| MCP サーバーを **消費**する .NET プログラムを作成する | `references/client.md` |
| MCP インスペクター、メモリ内テスト、モック、CI | `references/testing.md` |

マルチプリミティブ タスクの場合は、複数のタスクを一度にロードします。既存のファイルの簡単な編集の場合、通常は何も必要ありません。

## 基本ルール (常に適用され、最も頻繁に起こる破損を防ぎます)

1. **プレビューではなく、現在の安定したパッケージを固定します。** 最新の **1.x** では `ModelContextProtocol` / `ModelContextProtocol.AspNetCore` / `ModelContextProtocol.Core` を使用してください。 `0.3-preview` または `0.4-preview` を書いていることに気付いた場合は、停止して NuGet を確認してください。プレビュー API には重大な違いがあります。
2. **STDIO サーバーは stdout に書き込んではなりません。** Stdout は JSON-RPC チャネルです。 `LogToStandardErrorThreshold = LogLevel.Trace` を構成するのは何よりも前であり、決してツールから `Console.WriteLine` を構成しないでください。
3. **HTTP のデフォルトはステートフルです。** サーバーが開始するトラフィックのない水平スケールの展開の場合は、`options.Stateless = true` を設定します。サーバーからクライアントへの機能 (サンプリング、引き出し、ルート、一方的な通知) にはステートフル HTTP **または** STDIO が必要です。`Stateless = true` は実行時に機能を中断します。
4. **SSE のみは非推奨です。** ストリーミング可能な HTTP を使用します。サポートする必要がある古いクライアントに対してのみレガシー SSE (`EnableLegacySse = true`) を有効にし、それを呼び出します。
5. **常に `[Description]` ツールとパラメータ。** これは、LLM がコールを選択および形成するときに確認するものです。あいまいな説明は、ツールが使用されない最大の理由です。
6. **プリミティブを追加するたびに登録行を表示します。** `.WithPrompts<...>()` (または `.WithPromptsFromAssembly()`) のない新しい `[McpServerPromptType]` クラスは表示されません。
7. **API を発明しないでください。** メソッドが存在するかどうかわからない場合は、そう言って [API リファレンス](https://csharp.sdk.modelcontextprotocol.io/api/ModelContextProtocol.html) を確認してください。メソッド名が間違っていると、サイレント エラーが発生します。

## 働き方

- **最小限の追加的な変更を加えます。** プロジェクトを再構築するのではなく、既存のツール クラスにメソッドを追加します。
- **重要なセットアップの場合は、`dotnet build` を実行してください。** 不足している using 、属性のタイプミス、および TFM の不一致を、ユーザーが目にする前に検出します。
- **コンテキストから明らかになっていない場合は、スキャフォールディングの前にトランスポート + .NET バージョン + プリミティブを確認してください**。新しいプロジェクトのデフォルトは **.NET 10** です。

## ユーザーが行き詰まったとき

推測する前に、このチェックリストを実行してください。
1. **STDIO:** 何かが stdout に書き込まれています (ロガー シンク、`Console.WriteLine`、ライブラリ バナー)。
2. **HTTP 404:** パスが一致しません — `app.MapMcp()` はルートですが、`app.MapMcp("/mcp")` は `/mcp` の下に置かれます。
3. **ツールが表示されません:** クラスに `[McpServerToolType]` がないか、`.WithToolsFromAssembly()` / `.WithTools<T>()` が登録されていません。
4. **引数はバインドされていません:** パラメータ名は JSON-RPC `arguments` キーと一致する必要があります。複合型は `System.Text.Json` を介してバインドされます。
5. **サンプリング/誘導/ルートの失敗:** トランスポートがステートレス HTTP であるか、クライアントが機能をアドバタイズしません。

まだ行き詰まっていますか? [`EverythingServer`](https://github.com/modelcontextprotocol/csharp-sdk/tree/main/samples/EverythingServer) サンプルをユーザーに指示します。このサンプルでは、​​すべての機能が実行されます。