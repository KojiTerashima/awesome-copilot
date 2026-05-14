---
name: qdrant-search-speed-optimization
description: "Diagnoses and fixes slow Qdrant search. Use when someone reports 'search is slow', 'high latency', 'queries take too long', 'low QPS', 'throughput too low', 'filtered search is slow', or 'search was fast but now it's slow'. Also use when search performance degrades after config changes or data growth."
---
# 問題を診断する

検索パフォーマンスの低下には複数の理由が考えられます。最も一般的なものは次のとおりです。

* メモリプレッシャー: ワーキングセットが利用可能な RAM を超えた場合
* 複雑なリクエスト (例: 高い `hnsw_ef`、ペイロード インデックスのない複雑なフィルター)
* 競合するバックグラウンド プロセス (例: 一括アップロード後もオプティマイザーが実行中)
* クラスターの問題 (ネットワークの問題、ハードウェアの劣化など)


## 単一クエリが遅すぎる (レイテンシー)

負荷に関係なく、個々のクエリに時間がかかりすぎる場合に使用します。

### 診断手順:

- 同じリクエストの 2 回目の実行が大幅に高速化されているかどうかを確認します (メモリ負荷を示します)。
- `with_payload: false` と `with_vectors: false` を使用して同じクエリを試し、ペイロードの取得がボトルネックになっているかどうかを確認します
- リクエストでフィルターが使用されている場合は、フィルターを 1 つずつ削除して、特定のフィルター条件がボトルネックになっているかどうかを特定します。

### 一般的な修正:

- HNSW パラメーターの調整: [検索の微調整](https://search.qdrant.tech/md/documentation/operations/optimize/?s=fine-tuning-search-parameters)
- メモリ内量子化を有効にする: [スカラー量子化](https://search.qdrant.tech/md/documentation/manage-data/quantization/?s=scalar-quantization)
- マトリョーシカ モデルを使用してベクトルの次元を削減: [マトリョーシカ モデル](https://search.qdrant.tech/md/documentation/inference/?s=reduce-vector-dimensionity-with-matryoshka-models)
- 高次元ベクトルに対してオーバーサンプリング + リスコアを使用する [量子化による検索](https://search.qdrant.tech/md/documentation/manage-data/quantization/?s=searching-with-quantization)
- Linux 上のディスク負荷の高いワークロードに対して io_uring を有効にする [io_uring](https://qdrant.tech/articles/io_uring/)


## 十分な QPS (スループット) を処理できません

次の場合に使用します: システムが負荷の下で 1 秒あたり十分なクエリを処理できない。

- セグメント数を減らす (`default_segment_number` を 2 に) [スループットの最大化](https://search.qdrant.tech/md/documentation/operations/optimize/?s=maximizing-throughput)
- 単一クエリの代わりにバッチ検索 API を使用する [バッチ検索](https://search.qdrant.tech/md/documentation/search/search/?s=batch-search-api)
- 量子化を有効にして CPU コストを削減 [スカラー量子化](https://search.qdrant.tech/md/documentation/manage-data/quantization/?s=scalar-quantization)
- 読み取り負荷を分散するためにレプリカを追加する [レプリケーション](https://search.qdrant.tech/md/documentation/operations/distributed_deployment/?s=replication)


## フィルター検索が遅い

次の場合に使用します: フィルターを使用した検索は、フィルターを使用しない検索よりも大幅に時間がかかります。記憶に次いで最も一般的な SA の訴え。- フィルタリングされたフィールドにペイロード インデックスを作成します [ペイロード インデックス](https://search.qdrant.tech/md/documentation/manage-data/indexing/?s=payload-index)
- プライマリ フィルタリング条件には `is_tenant=true` を使用します: [テナント インデックス](https://search.qdrant.tech/md/documentation/manage-data/indexing/?s=tenant-index)
- 複雑なフィルターの ACORN アルゴリズムを試してください: [ACORN](https://search.qdrant.tech/md/documentation/search/search/?s=acorn-search-algorithm)
- `nested` フィルタリング条件をプライマリ フィルタとして使用することは避けてください。 qdrant にインデックスを使用する代わりに生のペイロード値を強制的に読み取る可能性があります。
- HNSW ビルド後にペイロード インデックスが追加された場合、再インデックスをトリガーしてフィルタリング可能なサブグラフ リンクを作成します


## 並列更新による検索パフォーマンスの最適化

### 診断手順

- `indexed_only=true` パラメータを使用して同じクエリを実行してみます。クエリが大幅に高速な場合は、オプティマイザがまだ実行中であり、すべてのセグメントのインデックスがまだ作成されていないことを意味します。
- クエリがないにもかかわらず CPU または IO の使用率が高い場合は、オプティマイザがまだ実行中であることも示します。

### 推奨される構成変更

- `optimizer_cpu_budget` を減らして、より多くの CPU をクエリ用に予約します
- `prevent_unoptimized=true` を使用して、検索用に大量のインデックスのないデータを含むセグメントが作成されるのを防ぎます。代わりに、セグメントがいわゆるindexing_thresholdに達すると、すべての追加ポイントは「遅延状態」で追加されます。 

詳細は[こちら](https://search.qdrant.tech/md/documentation/search/low-latency-search/?s=query-indexed-data-only)


## してはいけないこと

- 量子化に `always_ram=false` を設定します (検索ごとにディスク スラッシングが発生します)。
- レイテンシの影響を受けやすい運用のために HNSW をディスクに配置します (コールド ストレージのみ)
- スループットのためにセグメント数を増やす (逆: 少ない = 優れている)
- すべてのフィールドにペイロード インデックスを作成します (メモリを無駄にします)
- オプティマイザーのステータスを確認する前に Qdrant を責める