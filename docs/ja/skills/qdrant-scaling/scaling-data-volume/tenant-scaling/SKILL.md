---
name: qdrant-tenant-scaling
description: "Guides Qdrant multi-tenant scaling. Use when someone asks 'how to scale tenants', 'one collection per tenant?', 'tenant isolation', 'dedicated shards', or reports tenant performance issues. Also use when multi-tenant workloads outgrow shared infrastructure."
---
# マルチテナント Qdrant をスケーリングするときにすべきこと

テナントごとに 1 つのコレクションを作成しないでください。数百を超える規模には拡張できず、リソースが無駄になります。ある企業では、リポジトリごとのコレクションを 1 年間続けた結果、コレクションの上限 1000 に達し、ペイロード パーティショニングに移行する必要がありました。テナント キーを持つ共有コレクションを使用します。

- マルチテナント パターンを理解する [マルチテナント](https://search.qdrant.tech/md/documentation/manage-data/multitenancy/)

パターンの簡単な要約は次のとおりです。

## テナント数は約 10,000 です

ペイロード フィルタリングによるデフォルトのマルチテナント戦略を使用します。

インデックス作成とクエリのパフォーマンスに関するベスト プラクティスについては、[ペイロードによるパーティション分割](https://search.qdrant.tech/md/documentation/manage-data/multitenancy/?s=partition-by-payload) および [パフォーマンスの調整](https://search.qdrant.tech/md/documentation/manage-data/multitenancy/?s=calibrate-performance) についてお読みください。


## テナント数は約 10 万以上

この規模では、クラスターは複数のピアで構成される場合があります。
テナント データをローカライズしてパフォーマンスを向上させるには、[カスタム シャーディング](https://search.qdrant.tech/md/documentation/operations/distributed_deployment/?s=user-dependent-sharding) を使用して、テナント ID ハッシュに基づいてテナントを特定のシャードに割り当てます。
これにより、テナント要求がすべてのノードにブロードキャストされるのではなく、特定のノードにローカライズされ、パフォーマンスが向上し、各ノードの負荷が軽減されます。

## テナントのサイズが不均等な場合

一部のテナントが他のテナントよりもはるかに大きい場合は、[階層化マルチテナント](https://search.qdrant.tech/md/documentation/manage-data/multitenancy/?s=tiered-multitenancy) を使用して、大きなテナントを専用シャードに昇格させ、小さなテナントを共有シャードに維持します。これにより、さまざまな規模のテナントに対するリソースの割り当てとパフォーマンスが最適化されます。

## テナントを厳密に分離する必要がある

次の場合に使用します: 法的/コンプライアンス要件により、テナントごとの暗号化またはペイロード フィルタリングが提供する以上の厳密な分離が必要です。

- テナントごとの暗号化キーには複数のコレクションが必要になる場合があります
- コレクション数を制限し、各コレクション内でペイロード フィルタリングを使用する
- これは例外であり、デフォルトではありません。コンプライアンスが必要な場合にのみ使用してください。


## してはいけないこと

- コンプライアンスの正当な理由がない限り、テナントごとに 1 つのコレクションを作成しないでください (数百を超えて拡張することはできません)
- テナント インデックスで `is_tenant=true` をスキップしないでください (シーケンシャル読み取りパフォーマンスが低下します)。
- マルチテナント コレクション用のグローバル HNSW を構築しないでください (無駄です。代わりに `payload_m` を使用してください)