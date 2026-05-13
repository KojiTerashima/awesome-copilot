---
name: cloud-design-patterns
description: '信頼性、パフォーマンス、メッセージング、セキュリティ、デプロイメントのカテゴリにまたがる業界標準42パターンを扱う、分散システムアーキテクチャ向けクラウド設計パターンです。分散システムアーキテクチャを設計・レビュー・実装する際に使用してください。'
---

# Cloud Design Patterns

アーキテクトは、機能要件と非機能要件の両方を満たすために、プラットフォームサービス、機能、コードを統合してワークロードを設計します。効果的なワークロードを設計するには、これらの要件を理解し、ワークロードの制約に伴う課題へ対応できるトポロジーと手法を選ぶ必要があります。クラウド設計パターンは、多くの一般的な課題に対する解決策を提供します。

システム設計は、確立された設計パターンに大きく依存します。これらのパターンを組み合わせることで、インフラ、コード、分散システムを設計できます。これらのパターンは、クラウド上で信頼性が高く、高セキュアで、コスト最適化され、運用効率が高く、高性能なアプリケーションを構築するうえで重要です。

以下のクラウド設計パターンは技術非依存であるため、あらゆる分散システムに適用できます。Azure、他のクラウドプラットフォーム、オンプレミス構成、ハイブリッド環境にまたがって利用可能です。

## クラウド設計パターンが設計プロセスを強化する方法

クラウドワークロードは、分散コンピューティングの誤謬（分散システムの動作に関する一般的だが誤った前提）に弱い傾向があります。例:

- ネットワークは信頼できる。
- レイテンシはゼロである。
- 帯域幅は無限である。
- ネットワークは安全である。
- トポロジーは変わらない。
- 管理者は1人だけである。
- コンポーネントのバージョニングは簡単である。
- 可観測性の実装は後回しにできる。

これらの誤解は、欠陥のあるワークロード設計につながります。設計パターンは誤解そのものをなくすものではありませんが、認識を高め、補償戦略と緩和策を提供します。各クラウド設計パターンにはトレードオフがあります。実装方法よりも、なぜそのパターンを選ぶべきかに注目してください。

---

## 参考資料

| Reference | When to load |
|---|---|
| [Reliability & Resilience Patterns](references/reliability-resilience.md) | Ambassador, Bulkhead, Circuit Breaker, Compensating Transaction, Retry, Health Endpoint Monitoring, Leader Election, Saga, Sequential Convoy |
| [Performance Patterns](references/performance.md) | Async Request-Reply, Cache-Aside, CQRS, Index Table, Materialized View, Priority Queue, Queue-Based Load Leveling, Rate Limiting, Sharding, Throttling |
| [Messaging & Integration Patterns](references/messaging-integration.md) | Choreography, Claim Check, Competing Consumers, Messaging Bridge, Pipes and Filters, Publisher-Subscriber, Scheduler Agent Supervisor |
| [Architecture & Design Patterns](references/architecture-design.md) | Anti-Corruption Layer, Backends for Frontends, Gateway Aggregation/Offloading/Routing, Sidecar, Strangler Fig |
| [Deployment & Operational Patterns](references/deployment-operational.md) | Compute Resource Consolidation, Deployment Stamps, External Configuration Store, Geode, Static Content Hosting |
| [Security Patterns](references/security.md) | Federated Identity, Quarantine, Valet Key |
| [Event-Driven Architecture Patterns](references/event-driven.md) | Event Sourcing |
| [Best Practices & Pattern Selection](references/best-practices.md) | 適切なパターン選定、Well-Architected Frameworkとの整合、ドキュメント化、監視 |
| [Azure Service Mappings](references/azure-service-mappings.md) | 各パターンカテゴリに対応する一般的なAzureサービス |

---

## パターンカテゴリ一覧

| Category | Patterns | Focus |
|---|---|---|
| Reliability & Resilience | 9 patterns | フォールトトレランス、自己修復、グレースフルデグラデーション |
| Performance | 10 patterns | キャッシュ、スケーリング、負荷管理、データ最適化 |
| Messaging & Integration | 7 patterns | 疎結合、イベント駆動通信、ワークフロー調整 |
| Architecture & Design | 7 patterns | システム境界、APIゲートウェイ、移行戦略 |
| Deployment & Operational | 5 patterns | インフラ管理、地理分散、構成管理 |
| Security | 3 patterns | ID、アクセス制御、コンテンツ検証 |
| Event-Driven Architecture | 1 pattern | イベントソーシングと監査証跡 |

## 外部リンク

- [Cloud Design Patterns - Azure Architecture Center](https://learn.microsoft.com/azure/architecture/patterns/)
- [Azure Well-Architected Framework](https://learn.microsoft.com/azure/architecture/framework/)
