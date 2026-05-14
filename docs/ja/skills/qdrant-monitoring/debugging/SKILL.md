---
name: qdrant-monitoring-debugging
description: "Diagnoses Qdrant production issues using metrics and observability tools. Use when someone reports 'optimizer stuck', 'indexing too slow', 'memory too high', 'OOM crash', 'queries are slow', 'latency spike', or 'search was fast now it's slow'. Also use when performance degrades without obvious config changes."
---
# メトリクスを使用して Qdrant をデバッグする方法

まずオプティマイザーのステータスを確認します。ほとんどの運用上の問題は、リソースをめぐるアクティブな最適化の競合に遡ります。オプティマイザがクリーンな場合は、メモリをチェックしてからメトリクスをリクエストします。


## オプティマイザーがスタックしているか遅すぎる

次の場合に使用します: オプティマイザーが何時間も実行されている場合、終了していない場合、またはエラーが表示されている場合。

- `/collections/{collection_name}/optimizations` エンドポイント (v1.17 以降) を使用してステータスを確認する [最適化監視](https://search.qdrant.tech/md/documentation/operations/optimizer/?s=optimization-monitoring)
- オプションの詳細フラグを使用したクエリ: `?with=queued,completed,idle_segments`
- 戻り値: キューに入れられた最適化の数、アクティブなオプティマイザーのタイプ、関連するセグメント、進行状況の追跡
- Web UI には、タイムライン ビューとタスクごとの期間メトリクスを備えた [最適化] タブがあります [Web UI](https://search.qdrant.tech/md/documentation/operations/optimizer/?s=web-ui)
- `optimizer_status` がコレクション情報にエラーを示している場合は、ディスクがいっぱいであるかセグメントが破損していないかログを確認してください。
- 大規模なデータセットでは、大規模なマージと HNSW の再構築には正当に数時間かかります。行き詰まっていると考える前に、進行状況を確認してください。


## メモリが多すぎるようです

メモリが予想を超える場合、OOM でノードがクラッシュする場合、またはメモリが増加し続ける場合に使用します。

- `/metrics` 経由で利用可能なプロセス メモリ メトリクス (RSS、割り当てられたバイト数、ページ フォールト)
- Qdrant は、常駐メモリ (データ構造、量子化ベクトル) と OS ページ キャッシュ (キャッシュされたディスク読み取り) の 2 種類の RAM を使用します。ページ キャッシュが利用可能な RAM を埋めるのは正常です。 [メモリ記事](https://qdrant.tech/articles/memory-consumption/)
- 常駐メモリ (RSSAnon) が合計 RAM の 80% を超える場合は、調査してください。
- コレクションごとのポイント数とベクトル構成の内訳については、`/telemetry` を確認してください。
- 予想されるメモリの見積もり: ベクトルの `num_vectors * dimensions * 4 bytes * 1.5`、ペイロードとインデックスのオーバーヘッド [キャパシティ プランニング](https://search.qdrant.tech/md/documentation/operations/capacity-planning/)
- 予期せぬ増加の一般的な原因: `always_ram=true` を含む量子化ベクトル、多すぎるペイロード インデックス、最適化中の大規模な `max_segment_size`


## クエリが遅い

クエリが予想よりも遅く、原因を特定する必要がある場合に使用します。

- エンドポイントごとに `rest_responses_avg_duration_seconds` と `rest_responses_max_duration_seconds` を追跡
- Grafana でのパーセンタイル分析にはヒストグラム メトリクス `rest_responses_duration_seconds` (v1.8+) を使用します
- `grpc_responses_` プレフィックスが付いた同等の gRPC メトリクス
- 最初にオプティマイザのステータスを確認します。アクティブな最適化により CPU と I/O が競合し、検索遅延が悪化します。
- コレクション情報を通じてセグメント数を確認します。一括アップロード後に結合されていないセグメントが多すぎると、検索が遅くなります。
- フィルタされたクエリ時間とフィルタされていないクエリ時間を比較します。ギャップが大きい場合は、ペイロード インデックスが欠落していることを意味します。 [ペイロードインデックス](https://search.qdrant.tech/md/documentation/manage-data/indexing/?s=payload-index)


## してはいけないこと

- 遅いクエリをデバッグするときにオプティマイザのステータスを無視します (最も一般的な根本原因)
- ページ キャッシュが RAM をいっぱいにしたときのメモリ リークを想定します (通常の OS の動作)
- オプティマイザーの実行中に構成を変更します (カスケード再最適化が発生します)。
- 一括アップロードが完了したかどうかを確認する前に Qdrant を非難する (セグメントがマージされていない)