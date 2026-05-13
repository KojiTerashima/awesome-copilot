---
description: '分離ワーカーモデルを使用して C# で Azure Durable Functions を構築するためのガイドラインとベストプラクティス'
applyTo: '**/*.cs, **/host.json, **/local.settings.json, **/*.csproj'
---

# Azure Durable Functions C# 開発

## 一般的な指示

- 新しい Durable Functions プロジェクトの場合は、常に **分離ワーカーモデル** を `Microsoft.Azure.Functions.Worker.Extensions.DurableTask` NuGet パッケージとともに使用してください。
- オーケストレーターおよびアクティビティコンテキスト タイプ (`TaskOrchestrationContext`、`TaskActivityContext`) には `Microsoft.DurableTask` 名前空間を使用します。
- 明確にするために、オーケストレーター、アクティビティ、エンティティ、およびクライアントスターター関数を個別のクラスまたはファイルに分離します。
- オーケストレーションロジックとアクティビティロジックを決して混合しないでください。オーケストレーターが調整します。活動は機能します。
- ロギングにはオーケストレーター関数内で常に `context.CreateReplaySafeLogger(nameof(OrchestratorName))` を使用します。注入された `ILogger<T>` はリプレイごとにログに記録されるため、オーケストレーター内で直接使用しないでください。
- すべてのオーケストレーターおよびアクティビティメソッドには `async Task` または `async Task<T>` を使用します。`async void` は使用しないでください。
- オーケストレーターコードを **決定的でリプレイセーフ** として扱います。オーケストレーター内の `DateTime.Now`、`Guid.NewGuid()`、`Random`、直接 HTTP 呼び出し、または非決定的 I/O は禁止します。
- オーケストレーター内では `DateTime.UtcNow` の代わりに `context.CurrentUtcDateTime` を使用します。

## プロジェクトの構造

- `builder.Services.AddDurableTaskClient()` および `builder.ConfigureFunctionsWorkerDefaults(x => x.UseDurableTask())` を介して `Program.cs` で Durable Function サポートを登録します。
- オーケストレーター、アクティビティ、エンティティを、関数タイプごとではなく、機能ベースのフォルダー (`/Orchestrations/OrderProcessing/` など) に整理します。
- オーケストレーターにはサフィックス `Orchestrator` (例: `ProcessOrderOrchestrator`)、アクティビティにはサフィックス `Activity` (例: `ChargePaymentActivity`)、エンティティにはサフィックス `Entity` (例: `CartEntity`) を付けます。
- タイプミスを防ぐために、`CallActivityAsync`、`CallSubOrchestratorAsync`、および `GetEntityStateAsync` に渡されるアクティビティ/オーケストレーター/エンティティ名には定数または静的な読み取り専用文字列を使用してください。

## 設定ファイル

### ローカル設定.json
- ローカル開発には `AzureWebJobsStorage` 接続文字列を常に含めてください — Durable Functions では、オーケストレーション状態を維持するためにストレージが必要です。
- ローカルテストには `"UseDevelopmentStorage=true"` または Azurite 接続文字列を使用します。ローカル開発者の運用ストレージアカウントは決して使用しないでください。
- local.settings.json で `FUNCTIONS_WORKER_RUNTIME` を `"dotnet-isolated"` に設定します。
- Netherite または MSSQL ストレージプロバイダーの場合は、プロバイダー固有の接続文字列 (例: Netherite の場合は `EventHubsConnection`) を含めます。
- `local.settings.json` をソース管理にコミットしないでください。`.gitignore` に追加してください。代わりに `local.settings.json.example` をプレースホルダー値とともに使用してください。
- 必要に応じて、Azure Key Vault を使用して `@Microsoft.KeyVault(...)` 参照を介してローカルに機密値 (ストレージキー、Event Hub 接続文字列) を保存します。

### ホスト.json
- `"extensions": { "durableTask": { ... } }` で Durable Functions 固有の設定を構成します。運用環境ではデフォルトに依存しません。
- `"hubName"` を意味のある環境固有の値 (`"MyAppProd"`、`"MyAppDev"` など) に設定して、同じストレージアカウントを共有する環境間でタスクハブを分離します。
- 予想されるスループットとホスティングプランに基づいて `"maxConcurrentActivityFunctions"` と `"maxConcurrentOrchestratorFunctions"` を調整します。デフォルトは保守的です。
- プレミアム/専用プランで長時間実行されるオーケストレーションの拡張セッション (`"extendedSessionsEnabled": true`) を有効にして、再生のオーバーヘッドを削減します。
- ストレージプロバイダーを構成します。大規模なシナリオでは、既定の Azure Storage の代わりに `"storageProvider": { "type": "netherite" }` または `"mssql"` を使用します。
- `"maxQueuePollingInterval"` を適切に設定します。値を低くすると応答性が向上しますが、従量課金プランでのストレージトランザクション コストが増加します。
- Application Insights のサンプリングレートを `"logging": { "applicationInsights": { "samplingSettings": { ... } } }` で構成して、テレメトリの量を制御します。

## オーケストレーションパターン

### 関数の連鎖
- 各ステップが前のステップの結果に依存するステップバイステップのワークフローには、連続した `await context.CallActivityAsync<T>(nameof(ActivityName), input)` 呼び出しを使用します。
- アクティビティ間の入力/出力としてシリアル化可能な軽量データのみを渡します。循環参照を含むドメインオブジェクト全体を渡すことは避けてください。

### ファンアウト / ファンイン
- 複数の `context.CallActivityAsync` 呼び出しでファンアウトした後、`Task.WhenAll(tasks)` を使用して並列結果を集計します。
- 大規模なコレクションにファンアウトする場合は、並列処理の度合いを制限します。ダウンストリームサービスに負担をかけたり、Durable Functions のストレージ制限に達したりするのを避けるために、バッチ処理 (入力リストの分割など) を使用します。
- 動的タスク配列よりも `List<Task<T>>` を優先します。再生の問題を避けるために、待機する前にすべてのタスクをキャプチャします。

### 非同期 HTTP API (人間による対話 / 長時間実行)
- HTTP トリガースターター関数の `client.ScheduleNewOrchestrationInstanceAsync` を使用します。 `await client.CreateCheckStatusResponseAsync(req, instanceId)` を返して、呼び出し元にポーリング URL を提供します。
- `context.WaitForExternalEvent<T>("EventName", timeout)` を `context.CreateTimer(deadline, CancellationToken)` と組み合わせて使用​​し、タイムアウトのある承認/コールバックパターンを実装します。
- 常にタイムアウトレースを処理します。`Task.WhenAny(externalEventTask, timerTask)` を使用し、イベントが最初に到着した場合はタイマーをキャンセルします。

### モニタリング/ポーリングパターン
- ワークフローのポーリングには、タイマーでトリガーされる個別の関数の代わりに、`while` ループと `context.CreateTimer(context.CurrentUtcDateTime.Add(interval), CancellationToken.None)` を使用します。
- 決して終了しない無限ループを避けるために、監視ループに明確な終了条件があることを確認してください。
- 永遠に繰り返されるワークフローの場合は、`context.ContinueAsNew(input)` を使用してオーケストレーションを新しい状態で再開し、無制限の履歴の増加を防ぎます。

### 永遠のオーケストレーション
- オーケストレーター本体の最後に `context.ContinueAsNew(newInput)` を使用すると、長期間継続する繰り返しワークフローをクリーンな状態で再起動できます。
- `isKeepRunning` パターンを使用する場合は、`ContinueAsNew` を呼び出す前に保留中の外部イベントをすべて排出します。
- `ContinueAsNew` と `context.CreateTimer` を組み合わせて、定期的なタスク (日次レポートの生成、キャッシュの更新など) を実装します。

### サブオーケストレーション
- `context.CallSubOrchestratorAsync<T>(nameof(SubOrchestrator), instanceId, input)` を使用して、複雑なワークフローを再利用可能な子オーケストレーションに分解します。
- べき等性または相関関係が必要な場合は、サブオーケストレーションに明示的な `instanceId` を指定します。
- 履歴サイズの問題を回避するために、サブオーケストレーションのネストの深さを制限します。可能な場合はワークフローを平坦化します。

### エンティティ関数 (ステートフルエンティティ)
- 型指定され、カプセル化された状態管理のために `TaskEntity<TState>` を実装するクラスベースの構文を使用してエンティティを定義します。
- エンティティの状態にはエンティティの操作 (`entity.State`) を介してのみアクセスします。エンティティストレージを直接読み書きしないでください。
- Fire-and-Forget エンティティ操作には、アクティビティから `context.Entities.CallEntityAsync<T>` を使用するか、オーケストレーターから `context.Entities.SignalEntityAsync` を使用します。
- 戻り値が必要ない場合は、不必要なブロックを避けるために、オーケストレーターからの `CallEntityAsync` よりも `SignalEntityAsync` を優先します。
- 分散カウンター、分散ロック、アグリゲーター、またはユーザーごと/セッションごとの状態を必要とするシナリオにはエンティティを使用します。
- エンティティの状態を小さくし、シリアル化可能に保ちます。エンティティ状態で際限なく増大する大きな BLOB やコレクションを保存することは避けてください。

## アクティビティ関数

- アクティビティ関数は単一の作業単位に集中してください。アクティビティ関数は I/O (データベースの読み取り/書き込み、HTTP 呼び出し、キューの送信) を実行する唯一の場所です。
- コンストラクター DI を介してサービス (例: `IRepository`、`IHttpClientFactory`) をアクティビティ関数を含むクラスに挿入します。アクティビティメソッド内では `[FromServices]` を使用しないでください。
- 可能な限りアクティビティを*冪等**にします。オーケストレーターは再試行時に同じアクティビティを複数回呼び出すことができます。
- アクティビティコンテキストには `TaskActivityContext` パラメータタイプを使用します。挿入された `ILogger<T>` を使用してログを記録します (リプレイセーフなロガーではありません。アクティビティはリプレイされません)。
- アクティビティからはシリアル化可能な型のみを返します。ナビゲーションプロパティを持つドメインエンティティを返さないようにします。

## エラー処理と補償

- オーケストレーター内の try/catch ブロックで `context.CallActivityAsync` 呼び出しをラップし、`TaskFailedException` を処理して、適切なエラー処理と補正を行います。
- ワークフローの途中でステップが失敗した場合に、元に戻すアクティビティを呼び出すことで、catch ブロックに補償トランザクション (サガパターン) を実装します。
- アクティビティ呼び出しで `RetryPolicy` (`new TaskOptions(new RetryPolicy(maxRetries, firstRetryInterval))` 経由) を使用すると、一時的な障害が発生した場合にバックオフを伴う自動再試行が行われます。
- 一時的なエラー (再試行) とビジネスエラー (フェイルファストと補正) を区別します。検証や承認の失敗を再試行しません。
- スタックしたオーケストレーションが自己解決できないエラー状態になった場合は、Durable Functions 管理 API またはクライアントを介して常に終了します。

## タイマー

- オーケストレーター内の永続的な遅延には `context.CreateTimer(fireAt, CancellationToken)` を使用します。`Task.Delay` や `Thread.Sleep` は決して使用しないでください。
- 不要になったタイマー (タイマーが起動する前に外部イベントが到着した場合など) は、常に `CancellationTokenSource` を渡してキャンセルすることでキャンセルします。
- 従量課金プランの運用では、非常に短いタイマー間隔 (1 分未満) を避けてください。過剰なストレージポーリング コストが発生する可能性があります。

## インスタンス管理

- オーケストレーションをビジネスエンティティに関連付ける必要がある場合は、GUID の代わりに意味のある決定論的な `instanceId` 値 (例: `$"order-{orderId}"`) を使用します。
- オーケストレーションの重複 (シングルトンパターン) を防ぐために、新しいインスタンスをスケジュールする前に `client.GetInstanceMetadataAsync(instanceId)` を使用して既存のインスタンスを確認してください。
- 管理 API または管理機能でのライフサイクル管理には、`client.TerminateInstanceAsync`、`client.SuspendInstanceAsync`、および `client.ResumeInstanceAsync` を使用します。
- `client.PurgeInstanceAsync` または一括パージを使用して、完了/失敗したオーケストレーション履歴を定期的にパージして、タスクハブのストレージの増加を制御します。

## 可観測性

- オーケストレーター内のすべてのログ記録に `context.CreateReplaySafeLogger(nameof(Orchestrator))` を使用して、再生中のログエントリの重複を防ぎます。
- エンドツーエンドの追跡可能性を確保するために、オーケストレーターとスターターからのすべてのログステートメントに `instanceId` を記録します。
- Application Insights と Durable Functions の統合を使用して、オーケストレーションのライフサイクルイベント、アクティビティの期間、および障害を追跡します。
- Durable Functions HTTP 管理 API エンドポイント (`/runtime/webhooks/durabletask/instances`) または Durable Functions Monitor VS Code 拡張機能を介してオーケストレーションの状態を監視します。
- `host.json` に `durableTask.maxConcurrentOrchestratorFunctions` と `durableTask.maxConcurrentActivityFunctions` を設定して同時実行を制御し、リソースの枯渇を防ぎます。

## ストレージとタスクハブの構成

- 同じストレージアカウントを共有する環境 (dev/staging/prod) を分離するには、`"extensions": { "durableTask": { "hubName": "MyTaskHub" } }` の下の `host.json` でタスクハブ名を構成します。
- 環境間の干渉を避けるために、環境ごとに別のストレージアカウントまたはタスクハブ名を使用します。
- 高スループットのシナリオでは、デフォルトの Azure ストレージプロバイダーの代わりに **Netherite** または **MSSQL** ストレージプロバイダーを使用して、パフォーマンスを向上させ、コストを削減します。
- 大きなペイロード (>64KB) をオーケストレーションの入力/出力として直接保存することは避けてください。大きなデータを Blob Storage に保存し、代わりに参照 (URL/ID) を渡します。

## 耐久性のある関数のテスト

- `Microsoft.Azure.Functions.Worker.Extensions.DurableTask.Tests` NuGet パッケージ (利用可能な場合) を使用するか、単体テストオーケストレーター用に `TaskOrchestrationContext` を手動でモックします。
- アクティビティ関数を通常のメソッドとして分離してテストします。依存関係 (リポジトリ、HTTP クライアント) のモックを挿入し、戻り値でアサートします。
- テストハーネスまたは手動モックを使用して `context.CallActivityAsync`、`context.CreateTimer`、および `context.WaitForExternalEvent` をモックして、オーケストレーターロジックをテストします。
- Durable Functions ランタイム自体 (イベントソーシング、リプレイ) のテストは避けてください。オーケストレーターとアクティビティ内のビジネスロジックのテストに重点を置きます。
- Azurite または分離された Azure Storage アカウントとの統合テストを使用して、スターター → オーケストレーター → アクティビティ → 完了などのエンドツーエンドのワークフローをテストします。
- テストで決定的なインスタンス ID (例: `$"test-{Guid.NewGuid()}"`) を使用して、`client.GetInstanceMetadataAsync` を介してオーケストレーション状態のクエリと検証を有効にします。
- `context.CreateTimer` をモックしてすぐに起動し、オーケストレーターがタイムアウトブランチを処理することを確認して、タイムアウトシナリオをテストします。
- アクティビティの失敗を強制し (モック化されたアクティビティで例外をスローし)、オーケストレーターの呼び出しを補償するアクティビティをアサートすることで、補償/エラー処理をテストします。
- 統合テストではポーリングの代わりに `client.WaitForInstanceCompletionAsync` を使用します。オーケストレーションが完了するかタイムアウトになるまでブロックされます。
- エンティティテストの場合は、テストオーケストレーターで `context.Entities.SignalEntityAsync` を使用し、オーケストレーションの完了後に `client.ReadEntityStateAsync` を介してエンティティの状態を確認します。

## 既存のコードレビューガイダンス

- `DateTime.UtcNow` または `DateTime.Now` がオーケストレーター内で使用されている場合は、それにフラグを立てて `context.CurrentUtcDateTime` に置き換えます。
- オーケストレーター内で `Guid.NewGuid()` または `Random` が使用されている場合は、非決定的としてフラグを立てて、アクティビティに移動します。
- オーケストレーター内で直接 HTTP 呼び出し (`HttpClient.GetAsync` など) が行われた場合は、すぐにフラグを立てて、呼び出しをアクティビティ関数に移動します。
- オーケストレーター内で `Task.Delay` または `Thread.Sleep` が使用されている場合は、`context.CreateTimer` に置き換えます。
- 長時間実行ループで `ContinueAsNew` を使用しないとオーケストレーション履歴が際限なく増大する場合は、`ContinueAsNew` を追加して履歴をリセットすることをお勧めします。
- エンティティ状態が大規模なコレクションまたは BLOB データを保存している場合は、大規模なデータを Blob Storage に外部化し、参照のみをエンティティ状態に保存することをお勧めします。
- アクティビティ関数がべき等でなく、ワークフローに再試行/補償ロジックがない場合は、これを信頼性リスクとしてフラグを立てます。
