# 統合カタログ

Aspire には、13 カテゴリにわたって **144 以上の統合** があります。静的な一覧を維持する代わりに、MCP ツールを使って常に最新の統合データを取得してください。

---

## 統合を見つける（MCP ツール）

Aspire MCP サーバーは、統合の探索向けに 2 つのツールを提供しています。これらは **すべての CLI バージョン**（13.1+）で動作し、実行中の AppHost を **必要としません**。

| Tool                   | 何をするか                                                                                             | 使うタイミング                                                                                 |
| ---------------------- | -------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| `list_integrations`    | 利用可能な Aspire ホスティング統合をすべて、対応する NuGet パッケージ ID とともに返します                           | 「データベース向けにはどんな統合がある？」/「Redis 関連の統合をすべて見せて」 |
| `get_integration_docs` | 特定の統合パッケージの詳細ドキュメント（セットアップ、設定、コードサンプル）を取得します | 「PostgreSQL はどう設定するの？」/「`Aspire.Hosting.Redis` のドキュメントを見せて」            |

### ワークフロー

1. **一覧確認** — `list_integrations` を呼び出して利用可能な統合を確認します。カテゴリやキーワードで結果を絞り込みます。
2. **詳細確認** — `get_integration_docs` をパッケージ ID（例: `Aspire.Hosting.Redis`）とバージョン（例: `9.0.0`）付きで呼び出し、完全なセットアップ手順を取得します。
3. **追加** — `aspire add <integration>` を実行して、ホスティングパッケージを AppHost にインストールします。

> **Tip:** これらのツールは [official integrations gallery](https://aspire.dev/integrations/gallery/) と同じデータを返します。統合は頻繁に追加されるため、静的ドキュメントよりこちらを優先してください。

---

## 統合パターン

すべての統合は、2 パッケージ構成のパターンに従います。

- **Hosting package** (`Aspire.Hosting.*`) — AppHost にリソースを追加
- **Client package** (`Aspire.*`) — サービス内のクライアント SDK を、ヘルスチェック・テレメトリ・リトライ付きで設定
- **Community Toolkit** (`CommunityToolkit.Aspire.*`) — [Aspire Community Toolkit](https://github.com/CommunityToolkit/Aspire) によってコミュニティ保守されている統合

```csharp
// === AppHost (hosting side) ===
var redis = builder.AddRedis("cache");  // Aspire.Hosting.Redis
var api = builder.AddProject<Projects.Api>("api")
    .WithReference(redis);

// === Service (client side) — in API's Program.cs ===
builder.AddRedisClient("cache");        // Aspire.StackExchange.Redis
// Automatically configures: connection string, health checks, OpenTelemetry, retries
```

---

## カテゴリ一覧（概要）

完全で最新の一覧は `list_integrations` を使用してください。ここでは主要カテゴリを要約しています。

| Category            | 主な統合                                                                      | 例の hosting package                  |
| ------------------- | ------------------------------------------------------------------------------------- | ---------------------------------------- |
| **AI**              | Azure OpenAI, OpenAI, GitHub Models, Ollama                                           | `Aspire.Hosting.Azure.CognitiveServices` |
| **Caching**         | Redis, Garnet, Valkey, Azure Cache for Redis                                          | `Aspire.Hosting.Redis`                   |
| **Cloud / Azure**   | Storage, Cosmos DB, Service Bus, Key Vault, Event Hubs, Functions, SQL, SignalR (25+) | `Aspire.Hosting.Azure.Storage`           |
| **Cloud / AWS**     | AWS SDK 統合                                                                   | `Aspire.Hosting.AWS`                     |
| **Databases**       | PostgreSQL, SQL Server, MongoDB, MySQL, Oracle, Elasticsearch, Milvus, Qdrant, SQLite | `Aspire.Hosting.PostgreSQL`              |
| **DevTools**        | Data API Builder, Dev Tunnels, Mailpit, k6, Flagd, Ngrok, Stripe                      | `Aspire.Hosting.DevTunnels`              |
| **Messaging**       | RabbitMQ, Kafka, NATS, ActiveMQ, LavinMQ                                              | `Aspire.Hosting.RabbitMQ`                |
| **Observability**   | OpenTelemetry（組み込み）, Seq, OTel Collector                                         | `Aspire.Hosting.Seq`                     |
| **Compute**         | Docker Compose, Kubernetes                                                            | `Aspire.Hosting.Docker`                  |
| **Reverse Proxies** | YARP                                                                                  | `Aspire.Hosting.Yarp`                    |
| **Security**        | Keycloak                                                                              | `Aspire.Hosting.Keycloak`                |
| **Frameworks**      | JavaScript, Python, Go, Java, Rust, Bun, Deno, Orleans, MAUI, Dapr, PowerShell        | `Aspire.Hosting.Python`                  |

ポリグロットなフレームワークのメソッドシグネチャについては、[Polyglot APIs](polyglot-apis.md) を参照してください。

---

