---
name: qdrant-minimize-latency
description: "Guides Qdrant query latency optimization. Use when someone asks 'search is slow', 'how to reduce latency', 'p99 is too high', 'tail latency', 'single query too slow', 'how to make search faster', or 'latency spikes'."
---
# クエリレイテンシーのスケーリング

単一クエリのレイテンシは、クエリ実行パス内の最も遅いコンポーネントによって決まります。スループットと相関関係がある場合もありますが、必ずしもそうとは限りません。スループットとレイテンシーは調整の方向が逆です。

低レイテンシーの最適化は、単一クエリのリソース飽和状態を最大限に活用することを目的としていますが、スループットの最適化は、クエリごとのリソース使用量を最小限に抑えて、より多くの並列クエリを可能にすることを目的としています。

## レイテンシを下げるためのパフォーマンス チューニング

- CPU コアに一致するようにセグメント数を増やす (`default_segment_number: 16`) [遅延の最小化](https://search.qdrant.tech/md/documentation/operations/optimize/?s=minimizing-latency)
- 量子化ベクトルと HNSW を RAM に保持 (`always_ram=true`)
- クエリ時に `hnsw_ef` を削減します (速度向上のためのトレード リコール) [検索パラメータ](https://search.qdrant.tech/md/documentation/operations/optimize/?s=fine-tuning-search-parameters)
- ローカル NVMe を使用し、ネットワーク接続ストレージを避ける

## メモリプレッシャーとレイテンシー

RAM は遅延にとって最も重要なリソースです。ワーキング セットが利用可能な RAM を超えると、OS キャッシュの削除により、レイテンシが大幅に持続的に低下します。

- 垂直スケール RAM が最初です。ワーキングセットが 80% を超える場合は重大です。
- 量子化を使用します: スカラー (4 倍縮小) またはバイナリ (16 倍縮小) [量子化](https://search.qdrant.tech/md/documentation/manage-data/quantization/)
- フィルタリングが頻繁に行われない場合は、ペイロード インデックスをディスクに移動します [ディスク上のペイロード インデックス](https://search.qdrant.tech/md/documentation/manage-data/indexing/?s=on-disk-payload-index)
- `optimizer_cpu_budget` を設定してバックグラウンド最適化 CPU を制限する
- インデックス作成のスケジュール: ピーク時間帯には `indexing_threshold` を高く設定します


## レイテンシの垂直スケーリング

より多くの RAM とより高速な CPU は、レイテンシを直接短縮します。ノードのサイジングのガイドラインについては、[Vertical Scaling](../scaling-data-volume/vertical-scaling/SKILL.md) を参照してください。


## してはいけないこと

- 同じノード上でレイテンシーとスループットを同時に最適化することは期待できません。
- レイテンシーの影響を受けやすいワークロードには、少数の大きなセグメントを使用しないでください (各セグメントの検索に時間がかかります)。
- 90% を超える RAM で実行しないでください (キャッシュの削除により、レイテンシが大幅に低下し、数日続く場合があります)
- パフォーマンスのデバッグ中にオプティマイザのステータスを無視しないでください
- 負荷テストを行わずに RAM をスケールダウンしないでください (キャッシュの削除により、数日間の遅延が発生する可能性があります)