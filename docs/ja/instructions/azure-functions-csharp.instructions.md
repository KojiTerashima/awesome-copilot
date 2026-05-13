---
description: '分離ワーカーモデルを使用して C# で Azure Functions を構築するためのガイドラインとベストプラクティス'
applyTo: '**/*.cs, **/host.json, **/local.settings.json, **/*.csproj'
---

# Azure Functions C# 開発

## 一般的な指示

- .NET 6 以降を対象とするすべての新しい Azure Functions プロジェクトでは、常に **分離ワーカーモデル** (従来のインプロセスモデルではない) を使用してください。
- ホストのセットアップと依存関係の注入には、`Program.cs` で `FunctionsApplication.CreateBuilder(args)` または `HostBuilder` を使用します。
- 関数メソッドを `[Function("FunctionName")]` で装飾し、厳密に型指定されたトリガーとバインディング属性を使用します。
- 関数メソッドに焦点を当ててください。各関数は 1 つのことを実行し、ビジネスロジックを挿入されたサービスに委任する必要があります。
- ビジネスロジックを関数メソッド本体内に直接配置しないでください。それを DI 経由で登録されたテスト可能なサービスクラスに抽出します。
- 一貫した構造化ログを作成するには、関数パラメータとして渡された `ILogger` ではなく、コンストラクターを通じて挿入された `ILogger<T>` を使用します。
- すべての I/O バウンド操作には常に `async/await` を使用します。 `.Result` や `.Wait()` でブロックしないでください。
- 正常なシャットダウンを有効にするためにサポートされている場合は、`CancellationToken` パラメータを優先してください。

## プロジェクトの構造とセットアップ

- `Microsoft.Azure.Functions.Worker` および `Microsoft.Azure.Functions.Worker.Extensions.*` NuGet パッケージを使用します。
- クリーンな依存関係注入のための `builder.Services.Add*` 拡張メソッドを使用して、`Program.cs` にサービスを登録します。
- 関連する関数をトリガーの種類ではなくドメインの関心ごとに別のクラスにグループ化します。
- ローカル開発用に設定を `local.settings.json` に保存します。デプロイされた環境には Azure App Configuration または Application Settings を使用します。
- コード内に接続文字列やシークレットをハードコーディングしないでください。常に `IConfiguration` または環境変数から読み取ります。
- デプロイされた環境のシークレットには、アプリ設定で Key Vault 参照 (`@Microsoft.KeyVault(SecretUri=...)`) を使用します。
- Azure サービスへの認証には `Managed Identity` (`DefaultAzureCredential`) を使用します。キーを含む接続文字列は可能な限り避けてください。
- `host.json` をトリガータイプごとに調整してください。`maxConcurrentCalls`、`batchSize`、および再試行ポリシーをホストレベルで構成します。

## トリガー

- **HttpTrigger**: 運用エンドポイントには `AuthorizationLevel.Function` 以降を使用します。 `AuthorizationLevel.Anonymous` は、明示的な理由がある公開 API のためにのみ予約してください。 ASP.NET Core 統合モデルを使用する場合は、ASP.NET Core 統合 (`UseMiddleware`、`IActionResult` を返す) を使用します。
- **TimerTrigger**: スケジュールには NCRONTAB 式 (`"0 */5 * * * *"`) を使用します。 `RunOnStartup = true` はコールドスタートのたびにすぐに実行されるため、本番環境では使用しないでください。
- **QueueTrigger / ServiceBusTrigger**: `host.json` と Azure portal で `MaxConcurrentCalls`、配信不能ポリシー、および `MaxDeliveryCount` を構成します。 `ServiceBusReceivedMessage` を直接処理して、高度なメッセージ制御 (完了、破棄、配信不​​能) を実現します。
- **BlobTrigger**: 待ち時間を短縮し、ストレージトランザクション コストを削減するために、ポーリングベースの BLOB トリガーよりも Event Grid ベースの BLOB トリガー (`Microsoft.Azure.Functions.Worker.Extensions.EventGrid`) を優先します。
- **EventHubTrigger**: バッチ処理用に `cardinality` を `many` に設定します。バッチモードでは `EventData[]` または `string[]` パラメータタイプを使用します。 `EventHubTriggerAttribute` の組み込みチェックポイントを使用して常にチェックポイントを作成します。
- **CosmosDBTrigger**: Cosmos DB 変更のイベント駆動型処理に変更フィードトリガーを使用します。 `LeaseContainerName` を設定し、リースコンテナをデータコンテナとは別に管理します。

## 入力と出力のバインディング

- バインディングがユースケースをカバーする関数本体内で SDK を直接使用するのではなく、入力バインディングを使用して宣言的にデータを読み取ります。
- 複数の出力バインディングの場合は、適切な出力バインディング属性 (例: `[QueueOutput]`、`[BlobOutput]`、`[HttpResult]`) で注釈が付けられたプロパティを使用してカスタム戻り値の型を定義します。
- BLOB の読み取り/書き込みには `[BlobInput]` と `[BlobOutput]` を使用します。メモリの圧迫を避けるため、大きな BLOB の場合は `byte[]` よりも `Stream` を優先してください。
- ポイント読み取りと単純なクエリには `[CosmosDBInput]` を使用します。複雑なクエリの場合は、DI 経由で `Managed Identity` を使用して `CosmosClient` を挿入します。
- 単一メッセージの送信には `[ServiceBusOutput]` を使用します。バッチ処理または高度な送信シナリオの場合は、DI 経由で `ServiceBusSender` を挿入します。
- 同じリソースに対して、DI 経由で取得した SDK クライアントとバインディングベースの I/O を混在させないでください。一貫性を維持するには、リソースごとに 1 つのパターンを選択してください。

## 依存関係の注入と構成

- `DefaultAzureCredential` を使用して、`Azure.Extensions.AspNetCore.Configuration.Secrets` パッケージの `services.AddAzureClients()` を使用して、すべての外部クライアント (例: `BlobServiceClient`、`ServiceBusClient`、`CosmosClient`) をシングルトンとして登録します。
- 厳密に型指定された構成セクションには `IOptions<T>` または `IOptionsMonitor<T>` を使用します。
- 関数内で `static` 状態を使用することは避けてください。すべての共有状態は、DI 登録サービスを通じて流れる必要があります。
- `IHttpClientFactory` 経由で `HttpClient` インスタンスを登録し、接続プーリングを管理し、ソケットの枯渇を回避します。

## エラー処理と再試行

- `host.json` で組み込みの再試行ポリシーを構成します。`"retry"` と `fixedDelay` または `exponentialBackoff` 戦略を使用して、トリガーレベルの再試行を行います。
- コードレベルで一時的な障害を処理するには、再試行、サーキットブレーカー、およびタイムアウト戦略を備えた `Microsoft.Extensions.Http.Resilience` または Polly v8 (`ResiliencePipeline`) を使用します。
- 再スローまたはデッドレタリングの前に、常に特定の例外をキャッチし、構造化コンテキスト (相関 ID、入力識別子など) とともにログに記録します。
- すべての再試行後に失敗したメッセージには配信不能キューを使用します。関数ハンドラーで例外を黙って飲み込まないでください。
- HTTP トリガーの場合、予期されるエラー条件に対して例外をスローするのではなく、適切な `IActionResult` タイプ (`BadRequestObjectResult`、`NotFoundObjectResult`) を返します。

## 可観測性とロギング

- 構造化ログプロパティでは `ILogger<T>` を使用します: `_logger.LogInformation("Processing message {MessageId}", messageId)`。
- `Program.cs` の `builder.Services.AddApplicationInsightsTelemetryWorkerService()` および `builder.Logging.AddApplicationInsights()` を介して Application Insights を構成します。
- `TelemetryClient` は、自動的に収集されるもの以外のカスタムイベント、メトリクス、および依存関係の追跡に使用します。
- 運用環境での過剰なテレメトリコストを避けるために、`"logging"` の下の `host.json` に適切なログレベルを設定します。
- 関数とダウンストリームサービス間の分散トレースコンテキストの伝播には、`System.Diagnostics` から `Activity` および `ActivitySource` を使用します。
- 機密データ (PII、シークレット、接続文字列) をログステートメントに記録することは避けてください。

## パフォーマンスとスケーラビリティ

- 関数の起動時間を最小限に抑える: 高価な初期化を関数コンストラクターではなく、遅延ロードされたシングルトンに延期します。
- イベント駆動型の予測不可能なワークロードには従量課金プランを使用します。低遅延、高スループット、または VNet 統合シナリオには、プレミアムプランまたは専用プランを使用します。
- CPU を集中的に使用する作業の場合は、関数ホストスレッドをブロックするのではなく、バックグラウンド `Task` にオフロードするか、Durable Functions を使用してください。
- 可能な場合はバッチ操作: `IEnumerable<EventData>` または `ServiceBusReceivedMessage[]` 配列を、一度に 1 つのメッセージではなく、単一の関数呼び出しで処理します。
- `FUNCTIONS_WORKER_PROCESS_COUNT` と `maxConcurrentCalls` をホスティングプランと予想されるスループットに応じて適切に設定します。
- 展開パッケージから直接実行することでコールドスタートを高速化するには、アプリ設定で `WEBSITE_RUN_FROM_PACKAGE=1` を有効にします。

## 安全

- 処理する前に、HTTP トリガー入力を常に検証してサニタイズしてください。 FluentValidation またはデータアノテーションを使用します。
- 内部 API 間呼び出しには、Key Vault に保存されているファンクションキーとともに `AuthorizationLevel.Function` を使用します。
- 認証、レート制限、ルーティングを処理するために、公開 API の HTTP によってトリガーされる関数の前に Azure API Management (APIM) を統合します。
- 機密機能については、App Service ネットワーク機能 (IP 制限、プライベートエンドポイント) を使用して受信アクセスを制限します。
- PII またはシークレットを含むリクエスト本文をログに記録しないでください。

## テスト

- モック化された依存関係を持つ標準の xUnit/NUnit を使用して、関数ホストから独立してサービスクラスを単体テストします。
- `Azurite` (ローカル Azure Storage エミュレーター) と `TestServer` または Azure Functions Core Tools を使用した統合テスト機能。
- `Microsoft.Azure.Functions.Worker.Testing` ヘルパーを使用できる場合は、モック `FunctionContext` インスタンスを構築します。
- トリガー配管自体のテストは避けてください。サービスに抽出されたビジネスロジックのテストに焦点を当てます。

## 既存のコードレビューガイダンス

- プロジェクトで従来の**インプロセスモデル** (`FunctionsStartup`、`IWebJobsStartup`) を使用している場合は、分離ワーカーモデルに移行することを提案し、`dotnet-isolated-process-guide` 経由で移行パスを提供してください。
- ハードコーディングされた接続文字列またはストレージアカウント キーがコードファイルまたは構成ファイルで見つかった場合は、それらにフラグを立てて、`DefaultAzureCredential` および Key Vault 参照に置き換えることを提案します。
- 実稼働アプリで `TimerTrigger` に `RunOnStartup = true` が設定されている場合は、リスクとしてフラグを立て、代わりにデプロイメントスロットまたは機能フラグを使用することを提案します。
- `async void` が関数で使用されている場合は、すぐにフラグを立ててください。代わりに `async Task` を使用してください。
- 再試行ロジックが関数内で `Thread.Sleep` または `Task.Delay` を使用して手動で実装されている場合は、ホストレベルの再試行ポリシーまたは Polly 復元パイプラインに置き換えることをお勧めします。

