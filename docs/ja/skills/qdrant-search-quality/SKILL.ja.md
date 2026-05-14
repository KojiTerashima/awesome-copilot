---
name: qdrant-search-quality
description: "Diagnoses and improves Qdrant search relevance. Use when someone reports 'search results are bad', 'wrong results', 'low precision', 'low recall', 'irrelevant matches', 'missing expected results', or asks 'how to improve search quality?', 'which embedding model?', 'should I use hybrid search?', 'should I use reranking?'. Also use when search quality degrades after quantization, model change, or data growth."
allowed-tools:
  - Read
  - Grep
  - Glob
---
# Qdrant の検索品質

まず、問題が埋め込みモデルにあるのか、Qdrant 構成にあるのか、クエリ戦略にあるのかを判断します。ほとんどの品質問題は、Qdrant 自体ではなく、モデルまたはデータに起因します。検索品質が低い場合は、パラメーターを調整する前に、チャンクがどのように Qdrant に渡されるかを検査してください。文の途中で分割すると、品質が 30 ～ 40% 低下する可能性があります。

- 問題を切り分けるために完全一致検索でテストすることから始めます [検索 API](https://search.qdrant.tech/md/documentation/search/search/?s=search-api)


## 診断とチューニング

品質問題の原因を特定し、HNSW パラメータを調整し、適切な埋め込みモデルを選択します。 [診断とチューニング](diagnosis/SKILL.md)


## 検索戦略

結果の品質を向上させるためのハイブリッド検索、再ランキング、関連性フィードバック、探索 API。 [検索戦略](search-strategies/SKILL.md)