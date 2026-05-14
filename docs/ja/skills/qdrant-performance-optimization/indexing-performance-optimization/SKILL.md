---
name: qdrant-indexing-performance-optimization
description: "Diagnoses and fixes slow Qdrant indexing and data ingestion. Use when someone reports 'uploads are slow', 'indexing takes forever', 'optimizer is stuck', 'HNSW build time too long', or 'data uploaded but search is bad'. Also use when optimizer status shows errors, segments won't merge, or indexing threshold questions arise."
---
# Qdrant インデックス作成が遅すぎる場合の対処方法

Qdrant は HNSW インデックスをすぐには構築しません。小さなセグメントでは、`indexing_threshold_kb` (デフォルト: 20 MB) を超えるまでブルート フォースが使用されます。このウィンドウ中の検索は仕様により遅くなり、バグではありません。

- インデックス作成オプティマイザーを理解する [インデックス作成オプティマイザー](https://search.qdrant.tech/md/documentation/operations/optimizer/?s=indexing-optimizer)


## アップロード/取り込みが遅すぎる

アップロードまたは更新/挿入 API 呼び出しが遅い場合に使用します。
ボトルネックの特定: クライアント側 (ネットワーク、バッチ処理) とサーバー側 (CPU、ディスク I/O)

クライアント側では、バッチ処理と並列処理を最適化します。

- バッチ更新/挿入を使用します (リクエストあたり 64 ～ 256 ポイント) [Points API](https://search.qdrant.tech/md/documentation/manage-data/points/?s=upload-points)
- 2 ～ 4 つの並列アップロード ストリームを使用する

サーバー側では、Qdrant 構成とインデックス作成戦略を最適化します。

- さらにシャード (3 ～ 12) を作成します。各シャードには独立した更新ワーカーがあります [シャーディング](https://search.qdrant.tech/md/documentation/operations/distributed_deployment/?s=sharding)
- HNSW を構築する前にペイロード インデックスを作成します (フィルター可能なベクトル インデックスに必要) [ペイロード インデックス](https://search.qdrant.tech/md/documentation/manage-data/indexing/?s=payload-index)

大規模なデータセットの初期一括ロードに適しています。

- 一括ロード中に HNSW を無効にする (`indexing_threshold_kb` を非常に高く設定し、後で復元) [コレクション パラメーター](https://search.qdrant.tech/md/documentation/manage-data/collections/?s=update-collection-parameters)
- HNSW を無効にする `m=0` の設定はレガシーです。代わりに高い `indexing_threshold_kb` を使用してください

インデックスを付けずに慎重に高速アップロードすると、オプティマイザーが追いつくまで一時的に RAM の使用量が増え、検索パフォーマンスが低下する可能性があります。

https://search.qdrant.tech/md/documentation/tutorials-develop/bulk-upload/ を参照してください。


## オプティマイザーがスタックするか時間がかかりすぎる

次の場合に使用します: オプティマイザーが数時間実行され、終了しない。

- 最適化エンドポイント (v1.17+) 経由で実際の進行状況を確認 [最適化モニタリング](https://search.qdrant.tech/md/documentation/operations/optimizer/?s=optimization-monitoring)
- 大規模なデータセットでは、大規模なマージと HNSW の再構築には正当に数時間かかります。
- CPU とディスク I/O を確認します (HNSW は CPU バウンド、マージは I/O バウンド、HDD は実行不可)
- `optimizer_status` にエラーが表示された場合は、ディスクがいっぱいになっているかセグメントが破損していないかログを確認してください。


## HNSW のビルド時間が長すぎます

次の場合に使用します: HNSW インデックスの構築が総インデックス作成時間の大半を占めます。

- `m` を削減します (デフォルトは 16、ほとんどの場合に適しています、32 以上はほとんど必要ありません) [HNSW パラメータ](https://search.qdrant.tech/md/documentation/manage-data/indexing/?s=vector-index)
- `ef_construct` を削減します (100 ～ 200 で十分です) [HNSW config](https://search.qdrant.tech/md/documentation/manage-data/collections/?s=indexing-vectors-in-hnsw)
- `max_indexing_threads` を CPU コアに比例させます [構成](https://search.qdrant.tech/md/documentation/operations/configuration/)
- インデックス作成に GPU を使用する [GPU インデックス作成](https://search.qdrant.tech/md/documentation/operations/running-with-gpu/)

## マルチテナント コレクションの HNSW インデックスすべてのデータが何らかのペイロード フィールド (`tenant_id` など) によって分割されるマルチテナントのユースケースがある場合は、グローバル HNSW インデックスの構築を回避し、代わりに `payload_m` を利用してデータのサブセットのみの HNSW インデックスを構築できます。
グローバル HNSW インデックスをスキップすると、インデックス作成時間を大幅に短縮できます。

詳細については、[マルチテナント コレクション](https://search.qdrant.tech/md/documentation/manage-data/multitenancy/) を参照してください。

## 追加のペイロード インデックスが遅すぎる

Qdrant は、フィルタリングされたベクトル検索の品質が低下しないように、すべてのペイロード インデックスに対して追加の HNSW リンクを構築します。
一部のペイロード インデックス (長いテキストを含む `text` フィールドなど) は、ポイントごとに非常に多くの一意の値を持つ可能性があり、HNSW のビルド時間が長くなる可能性があります。

特定のペイロード インデックスに対する追加の HNSW リンクの構築を無効にし、代わりに ACORN のようなわずかに遅いクエリ時間戦略に依存することができます。

追加の HNSW リンクの無効化について詳しくは、[ドキュメント](https://search.qdrant.tech/md/documentation/manage-data/indexing/?s=disable-the-creation-of-extra-edges-for-payload-fields) をご覧ください。

ACORN について詳しくは、[ドキュメント](https://search.qdrant.tech/md/documentation/search/search/?s=acorn-search-algorithm) をご覧ください。


## してはいけないこと

- HNSW の構築後にペイロード インデックスを作成しないでください (フィルター可能なベクトル インデックスが壊れます)
- 既存のコレクションへの一括アップロードには `m=0` を使用しないでください。既存の HNSW が削除され、インデックスの再作成に時間がかかる可能性があります。 
- 一度に 1 ポイントをアップロードしないでください (リクエストごとのオーバーヘッドが支配的になります)