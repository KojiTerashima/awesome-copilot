# Best Practices for Pattern Selection

## 適切なパターンの選定

- **問題を理解する**: パターンを選ぶ前に、具体的な課題を明確にする
- **トレードオフを考慮する**: どのパターンも複雑性とトレードオフを持つ
- **パターンを組み合わせる**: 多くのパターンは組み合わせると効果が高い（Circuit Breaker + Retry、CQRS + Event Sourcing）
- **シンプルに始める**: 過剰設計を避け、必要が明確なときに適用する
- **プラットフォーム固有性**: パターンをネイティブ実装できるAzureサービスを考慮する

## Well-Architected Framework との整合

選択したパターンを Well-Architected Framework の柱にマッピングします:
- **Reliability**: Circuit Breaker, Bulkhead, Retry, Health Endpoint Monitoring
- **Security**: Federated Identity, Valet Key, Gateway Offloading, Quarantine
- **Cost Optimization**: Compute Resource Consolidation, Static Content Hosting, Throttling
- **Operational Excellence**: External Configuration Store, Sidecar, Deployment Stamps
- **Performance Efficiency**: Cache-Aside, CQRS, Materialized View, Sharding

## パターン実装のドキュメント化

パターンを実装する際は、次を記録します:
- どのパターンを使うか、そしてその理由
- 受け入れたトレードオフ
- 設定値とチューニングパラメータ
- 監視と可観測性の方針
- 障害シナリオと復旧手順

## パターンの監視

- すべてのパターンに対して包括的な可観測性を実装する
- パターン固有メトリクス（circuit breaker state、cache hit ratio、queue depth）を追跡する
- 複数サービスにまたがるパターンでは分散トレーシングを使う
- パターン劣化（回路が頻繁にOpen、高いリトライ率）でアラートする
