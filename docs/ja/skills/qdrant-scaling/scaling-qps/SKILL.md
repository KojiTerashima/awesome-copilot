---
name: qdrant-scaling-qps
description: "Guides Qdrant query throughput (QPS) scaling. Use when someone asks 'how to increase QPS', 'need more throughput', 'queries per second too low', 'batch search', 'read replicas', or 'how to handle more concurrent queries'."
---
# クエリ スループット (QPS) のスケーリング

スループットのスケーリングとは、1 秒あたりにより多くの並列クエリを処理することを意味します。 
これはレイテンシとは異なります。スループットとレイテンシは調整方向が逆であり、同じノード上で同時に最適化することはできません。

スループットが高いと、セグメントが少なく、より大きなセグメントが優先されるため、各クエリのオーバーヘッドが少なくなります。


## より高い RPS のためのパフォーマンス チューニング

- より少ない、より大きなセグメントを使用します (`default_segment_number: 2`) [スループットの最大化](https://search.qdrant.tech/md/documentation/operations/optimize/?s=maximizing-throughput)
- `always_ram=true` で量子化を有効にしてディスク IO を削減します [量子化](https://search.qdrant.tech/md/documentation/manage-data/quantization/)
- バッチ検索 API を使用してオーバーヘッドを償却 [バッチ検索](https://search.qdrant.tech/md/documentation/search/search/?s=batch-search-api)

## 更新ワークロードの影響を最小限に抑える

- 更新スループット制御 (v1.17 以降) を構成して、最適化されていない検索による読み取りの低下を防止します [低遅延検索](https://search.qdrant.tech/md/documentation/search/low-latency-search/)
- `optimizer_cpu_budget` を設定してインデックス作成 CPU を制限します (例: 8 CPU ノードの `2` はクエリ用に 6 を予約します)
- テール レイテンシの遅延読み取りファンアウト (v1.17 以降) を構成する [遅延ファンアウト](https://search.qdrant.tech/md/documentation/search/low-latency-search/?s=use-layed-fan-outs)



## スループットの水平スケーリング

上記のチューニングを適用した後に単一ノードの CPU が飽和状態になった場合は、リードレプリカを使用して水平方向にスケーリングします。

- シャード レプリカは、複製されたシャードからのクエリを処理し、読み取り負荷をノード全体に分散します。
- 各レプリカは、再シャーディングせずに独立したクエリ容量を追加します
- `replication_factor: 2+` を使用し、読み取りをレプリカにルーティングします [分散展開](https://search.qdrant.tech/md/documentation/operations/distributed_deployment/?s=replication)

一般的な水平スケーリングのガイダンスについては、[水平スケーリング](../scaling-data-volume/horizontal-scaling/SKILL.md) も参照してください。


## ディスク I/O ボトルネック

すべてのベクトルを RAM に保持できない場合、ディスク I/O がスループットのボトルネックになる可能性があります。 
この場合:

- 最初にプロビジョンド IOPS またはローカル NVMe にアップグレードします。 [ディスク パフォーマンスの記事](https://qdrant.tech/articles/memory-consumption/) でディスク パフォーマンスのベクトル検索への影響を参照してください。
- Linux (カーネル 5.11 以降) で `io_uring` を使用する [io_uring 記事](https://qdrant.tech/articles/io_uring/)
- 量子化されたベクトルの場合、ディスク読み取りを減らすために、セグメントごとの再スコアリングよりもグローバルな再スコアリングを優先します。 [チュートリアル](https://search.qdrant.tech/md/documentation/tutorials-operations/large-scale-search/?s=search-query) の例
- ディスク読み取りを並列化するために、より多くの検索スレッドを構成します。デフォルトは `cpu_count - 1` で、RAM ベースの検索には最適ですが、ディスク ベースの検索には低すぎる可能性があります。 [構成リファレンス](https://search.qdrant.tech/md/documentation/operations/configuration/?s=configuration-options) を参照してください。
- まだ飽和している場合は、水平方向にスケールアウトします (各ノードは独立した IOPS を追加します)


## してはいけないこと- 同じノード上でスループットとレイテンシーを同時に最適化することは期待できません。
- スループットワークロードに多数の小さなセグメントを使用しないでください (クエリごとのオーバーヘッドが増加します)。
- IOPS 制限がある場合は、ディスク層もアップグレードせずに水平方向にスケーリングしないでください。
- 90% を超える RAM で実行しないでください (OS キャッシュの削除 = 重大なパフォーマンスの低下)