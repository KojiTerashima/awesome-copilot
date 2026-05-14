---
name: qdrant-search-strategies
description: "Guides Qdrant search strategy selection. Use when someone asks 'should I use hybrid search?', 'BM25 or sparse vectors?', 'how to rerank?', 'results are not relevant', 'I don't get needed results from my dataset but they're there', 'retrieval quality is not good enough', 'results too similar', 'need diversity', 'MMR', 'relevance feedback', 'recommendation API', 'discovery API', 'ColBERT reranking', or 'missing keyword matches'"
---
# 高度な戦略で検索結果を改善する方法

これらの戦略は、基本的なベクトル検索を補完します。埋め込みモデルがタスクに適合していること、および HNSW 構成が正しいことを確認してから使用してください。完全一致検索で悪い結果が返された場合は、最初に埋め込みモデル (取得者) の選択を確認してください。
小さく、高速で、安価であるため、ユーザーが弱い埋め込みモデルを使用したい場合は、再ランキングまたは関連性フィードバックを使用して検索品質を向上させます。

## 明らかなキーワード一致がありません

次の場合に使用します: 純粋なベクトル検索では、明らかなキーワード一致を含む結果が見つかりません。埋め込みトレーニング データに含まれないドメイン用語、重要なキーワードの正確な一致 (ブランド名、SKU)、一般的な頭字語。次の場合にスキップします: 純粋なセマンティック クエリ、トレーニング セット内のすべてのデータ、レイテンシ バジェットが非常に厳しい。

- `prefetch` と融合による密 + 疎 [ハイブリッド検索](https://search.qdrant.tech/md/documentation/search/hybrid-queries/?s=hybrid-search)
- 該当する場合、生の BM25 よりも学習されたスパース ([miniCOIL](https://search.qdrant.tech/md/documentation/fastembed/fastembed-minicoil/)、SPLADE、GTE) を優先します (ユーザーがスマートなキーワード マッチングを必要とし、学習されたスパース モデルがドメインの語彙を知っている場合)
- 英語以外の言語の場合、[スパース BM25 パラメータを適宜設定](https://search.qdrant.tech/md/documentation/search/text-search/?s= language- specific-settings)
- RRF: 適切なデフォルト、加重 (v1.17 以降) をサポート [RRF](https://search.qdrant.tech/md/documentation/search/hybrid-queries/?s=reciprocal-rank-fusion-rrf)
- 非対称制限 (sparse_limit=250、dense_limit=100) を持つ DBSF は、技術ドキュメントの RRF を上回るパフォーマンスを発揮できる [DBSF](https://search.qdrant.tech/md/documentation/search/hybrid-queries/?s=distribution-based-score-fusion-dbsf)
- リランキングによる融合も可能

## 正しいドキュメントが見つかりましたが、順序が間違っています

次の場合に使用します: 再現率は良いが、精度が低い (正しいドキュメントがトップ 10 ではなく、トップ 100 にある)。

- FastEmbed 経由のクロスエンコーダ リランカー [Rerankers](https://search.qdrant.tech/md/documentation/fastembed/fastembed-rerankers/)
- Qdrant での [マルチステージ クエリ](https://search.qdrant.tech/md/documentation/search/hybrid-queries/?s=multi-stage-queries) の使用方法を参照してください。
- ColBERT および ColPali/ColQwen の再ランキングは、遅延相互作用メカニズムにより特に正確ですが、重いです。リソースを節約するには、HNSW を構築せずにマルチベクトルを構成して保存することが重要です。 [マルチベクトル表現](https://search.qdrant.tech/md/documentation/tutorials-search-engineering/using-multivector-representations/) を参照してください。

## 適切なドキュメントが見つかりませんが、存在します

次の場合に使用します: 基本的な取得は行われているが、データセット内に存在することがわかっている関連アイテムを取得者が見逃します。埋め込み可能なデータ (テキスト、画像など) に対して機能します。関連性フィードバック (RF) クエリは、取得した結果のフィードバック モデルのスコアを使用して、取得者を通じてコレクション全体を再ランク付けするなど、後続の反復で完全なベクトル空間を通じて取得者を操作します。リランキングの補完: リランカーは限定されたサブセットを認識し、RF はコレクション全体のフィードバック信号を活用します。フィードバック スコアは 3 ～ 5 でも十分です。複数の反復を実行できます。

フィードバック モデルとは、バイエンコーダー、クロスエンコーダー、遅延インタラクション モデル、裁判官としての LLM など、ドキュメントごとの関連性スコアを生成するものです。フィードバックが段階的な関連性スコア (高い = より関連性が高い) として表現されるため、あいまいな関連性スコアは 2 値 (良い/悪い、関連性がある/無関係) だけでなく機能します。

次の場合はスキップします: 取得者がすでに強い再現率を持っている場合、または取得者とフィードバック モデルが関連性に関して強く一致している場合。

- RF クエリは現在、普遍的なデフォルトがない [3 パラメーターの単純な式](https://search.qdrant.tech/md/documentation/search/search-relevance/?s=naive-strategy) に基づいているため、データセット、取得者、およびフィードバック モデルごとに調整する必要があります。
- [qdrant-relevance-フィードバック](https://pypi.org/project/qdrant-relevance-フィードバック/) を使用してパラメーターを調整し、エバリュエーターで影響を評価し、取得者とフィードバックの一致を確認します。セットアップ手順については、README を参照してください。 GPU は必要なく、フレームワークには事前定義された取得およびフィードバック モデルのオプションも提供されます。
- [関連性フィードバッククエリAPI](https://search.qdrant.tech/md/documentation/search/search-relevance/?s=relevance-フィードバック)の設定を確認してください。
- これを、API の使用方法と `qdrant-relevance-feedback` フレームワークの実行方法を理解するために、パラメータ調整と eval を備えたヘルパー エンドツーエンドのテキスト取得例として使用します: [RF チュートリアル](https://search.qdrant.tech/md/documentation/tutorials-search-engineering/using-relevance-feedback/)

## 結果が類似しすぎています

次の場合に使用します: 上位の結果が冗長、重複に近い、または多様性に欠けている。密度の高いコンテンツ領域 (学術論文、製品カタログ) でよく見られます。

- MMR (v1.15+) を `diversity` とともにクエリ パラメーターとして使用して、関連性と多様性のバランスをとります [MMR](https://search.qdrant.tech/md/documentation/search/search-relevance/?s=maximal-marginal-relevance-mmr)
- `diversity=0.5` から始めます。精度を高めるには値を低くし、探索を増やすには値を高くします。
- MMR は標準の検索よりも時間がかかります。冗長性が実際に問題となる場合にのみ使用してください。

## 良い結果がどのように見えるかを知っているが、それを得ることができない

使用する場合: 肯定的な例と否定的な例のポイントを提供して、検索を肯定的な方向に近づけたり、否定的な方向から遠ざけたりすることができます。- レコメンデーション API: フィッティング ベクトルを推奨するためのポジティブ/ネガティブの例 [レコメンデーション API](https://search.qdrant.tech/md/documentation/search/explore/?s=recommendation-api)
  - ベスト スコア戦略: 多様な例に適し、負のみをサポート [ベスト スコア](https://search.qdrant.tech/md/documentation/search/explore/?s=best-score-strategy)
- Discovery API: リクエストターゲットなしで検索領域を制限するためのコンテキストペア (ポジティブ/ネガティブ) [Discovery](https://search.qdrant.tech/md/documentation/search/explore/?s=discovery-api)

## 関連性の背後にビジネス ロジックがある
使用する場合: 結果は、最新性や距離などのデータに基づくビジネス ロジックに従って追加でランク付けされる必要があります。

設定方法は[Score Boosting docs](https://search.qdrant.tech/md/documentation/search/search-relevance/?s=score-boosting)で確認してください。

## してはいけないこと

- 純粋なベクトルの品質を検証する前にハイブリッド検索を使用します (複雑さが増し、モデルの問題が隠れる可能性があります)
- 言語固有のストップワード削除を正しく設定せずに英語以外のテキストで BM25 を使用すると (結果が大幅に低下します)
- 関連性フィードバックを追加するときに評価をスキップします (実際に役立つかどうか実際のクエリで確認することをお勧めします)