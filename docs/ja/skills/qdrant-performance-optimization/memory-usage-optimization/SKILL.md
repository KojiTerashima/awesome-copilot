---
name: qdrant-memory-usage-optimization
description: "Diagnoses and reduces Qdrant memory usage. Use when someone reports 'memory too high', 'RAM keeps growing', 'node crashed', 'out of memory', 'memory leak', or asks 'why is memory usage so high?', 'how to reduce RAM?'. Also use when memory doesn't match calculations, quantization didn't help, or nodes crash during recovery."
---
# メモリ使用量を理解する

Qdrant は 2 種類のメモリで動作します。

- 常駐メモリ (別名 RSSAnon) - ID トラッカーなどの内部データ構造に使用されるメモリと、`always_ram=true` の量子化ベクトルやペイロード インデックスなど、RAM に常駐する必要があるコンポーネントに使用されるメモリ。

- OS ページ キャッシュ - ディスク読み取りのキャッシュに使用されるメモリ。必要に応じて解放できます。通常、元のベクトルはページ キャッシュに保存されるため、RAM がいっぱいになってもサービスがクラッシュすることはありませんが、パフォーマンスが低下する可能性があります。

OS ページ キャッシュが使用可能なすべての RAM を占有するのは正常ですが、常駐メモリが合計 RAM の 80% を超えている場合は、問題の兆候です。

## メモリ使用量の監視

- Qdrant は、`/metrics` エンドポイントを通じてメモリ使用量を公開します。 [監視ドキュメント](https://search.qdrant.tech/md/documentation/operations/monitoring/) を参照してください。

<!-- ToDo: API が利用可能になったら、各コンポーネントのメモリ使用量について説明します -->


## Qdrant にはどれくらいのメモリが必要ですか?

最適なメモリ使用量はユースケースによって異なります。

- 通常の検索シナリオの場合、一般的なガイドラインが [キャパシティ プランニング ドキュメント](https://search.qdrant.tech/md/documentation/operations/capacity-planning/) に記載されています。

大規模なメモリ使用量の詳細な内訳については、[大規模なメモリ使用量の例](https://search.qdrant.tech/md/documentation/tutorials-operations/large-scale-search/?s=memory-usage) を参照してください。

ペイロード インデックスと HNSW グラフはベクトル自体に加えてメモリも必要とするため、計算ではそれらを考慮することが重要です。

さらに、Qdrant は最適化のために追加のメモリを必要とします。最適化中、最適化されたセグメントは RAM に完全にロードされるため、十分なヘッドルームを残すことが重要です。
`max_segment_size` が大きいほど、より多くのヘッドルームが必要になります。


### HNSW インデックスをディスクに配置するタイミング

頻繁に使用されるコンポーネント (HNSW インデックスなど) をディスクに配置すると、パフォーマンスが大幅に低下する可能性があります。
ただし、これが良い選択肢となるシナリオもいくつかあります。

- 低遅延ディスクを使用した展開 - ローカル NVMe など。
- マルチテナント展開。テナントのサブセットのみが頻繁にアクセスされるため、一度に RAM に読み込まれるデータとインデックスの一部のみが使用されます。
- [インライン ストレージ](https://search.qdrant.tech/md/documentation/operations/optimize/?s=inline-storage-in-hnsw-index) が有効になっている展開の場合。


## メモリ使用量を最小限に抑える方法

主な課題は、めったにアクセスされないデータの部分をディスクに置くことです。
それを実現するための主なテクニックは次のとおりです。

- 量子化を使用して、圧縮ベクトルのみを RAM に保存します [量子化ドキュメント](https://search.qdrant.tech/md/documentation/manage-data/quantization/)

- float16 または int8 データ型を使用すると、ベクトルのメモリ使用量がそれぞれ 2 倍または 4 倍に削減されますが、精度は若干犠牲になります。ベクター データ型の詳細については、[ドキュメント](https://search.qdrant.tech/md/documentation/manage-data/vectors/?s=datatypes) をご覧ください。- Matryoshka Representation Learning (MRL) を利用して、大きなベクトルをディスク上に保持しながら、小さなベクトルのみを RAM に保存します。 Qdrant Cloud 推論で MRL を使用する方法の例: [MRL ドキュメント](https://search.qdrant.tech/md/documentation/inference/?s=reduce-vector-digitality-with-matryoshka-models)

- 小規模なテナントを含むマルチテナント展開の場合、同じテナントのデータが一緒に保存されるため、ベクトルがディスクに保存される可能性があります [マルチテナントのドキュメント](https://search.qdrant.tech/md/documentation/manage-data/multitenancy/?s=calibrate-performance)

- 高速なローカル ストレージと検索スループットの要件が比較的低い展開の場合、ベクター ストアのすべてのコンポーネントをディスクに保存できる場合があります。オンディスク ストレージのパフォーマンスへの影響について詳しくは、[記事](https://qdrant.tech/articles/memory-consumption/) をご覧ください。

- RAM が少ない環境の場合は、`async_scorer` 構成を検討してください。これにより、並列ディスク アクセス用の `io_uring` のサポートが有効になり、ディスク上のストレージのパフォーマンスが大幅に向上します。 `async_scorer` について詳しくは、[記事](https://qdrant.tech/articles/io_uring/) をご覧ください (カーネル 5.11 以降の Linux でのみ利用可能)

- スパース ベクターとテキスト ペイロードは通常、密なベクターよりもディスクに優しいため、ディスクに保存することを検討してください。
- ディスクに保存されるペイロード インデックスを構成する [ドキュメント](https://search.qdrant.tech/md/documentation/manage-data/indexing/?s=on-disk-payload-index)
- スパース ベクトルをディスクに保存するように構成する [ドキュメント](https://search.qdrant.tech/md/documentation/manage-data/indexing/?s=sparse-vector-index)