---
name: qdrant-performance-optimization
description: "Different techniques to optimize the performance of Qdrant, including indexing strategies, query optimization, and hardware considerations. Use when you want to improve the speed and efficiency of your Qdrant deployment."
allowed-tools:
  - Read
  - Grep
  - Glob
---
# Qdrant パフォーマンスの最適化

Qdrant のパフォーマンスにはさまざまな側面があります。このドキュメントは、Qdrant のパフォーマンス最適化のさまざまな側面のナビゲーション ハブとして機能します。


## 検索速度の最適化

検索速度には、レイテンシーとスループットという 2 つの異なる基準があります。 
レイテンシは 1 つのクエリに対する応答を取得するのにかかる時間であり、スループットは特定の時間枠内に処理できるクエリの数です。
ユースケースに応じて、これらのメトリクスの一方または両方を最適化することが必要になる場合があります。

検索速度の最適化の詳細については、[検索速度の最適化](search-speed-optimization/SKILL.md) スキルをご覧ください。


## インデックス作成パフォーマンスの最適化

Qdrant は、効率的な類似性検索を実行するためにベクトル インデックスを構築する必要があります。インデックスの構築にかかる時間は、データセット、ハードウェア、構成のサイズによって異なります。

インデックス作成パフォーマンスの最適化の詳細については、[インデックス作成パフォーマンスの最適化](indexing-performance-optimization/SKILL.md) スキルを参照してください。


## メモリ使用量の最適化

ベクトル検索は、特に大規模なデータセットを扱う場合、メモリを大量に消費する可能性があります。
Qdrant には柔軟なメモリ管理システムがあり、ストレージのどの部分をメモリに保存し、どの部分をディスクに保存するかを正確に制御できます。これにより、パフォーマンスを犠牲にすることなくメモリ使用量を最適化できます。

メモリ使用量の最適化の詳細については、[メモリ使用量の最適化](memory-usage-optimization/SKILL.md) スキルを参照してください。