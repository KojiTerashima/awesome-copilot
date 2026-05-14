---
name: qdrant-search-quality-diagnosis
description: "Diagnoses Qdrant search quality issues. Use when someone reports 'results are bad', 'wrong results', 'not relevant results', 'missing matches', 'recall is low', 'approximate search worse than exact', 'which embedding model', or 'quality dropped after quantization'. Also use when search quality degrades without obvious changes."
---
# 悪い検索品質を診断する方法

チューニングの前に、ベースラインを確立します。正確な KNN をグラウンド トゥルースとして使用し、近似的な HNSW と比較します。生産では、95% 以上のリコール @K を目標にします。

## 何が問題なのかまだわかりません

次の場合に使用します。結果が無関係であるか、期待された一致が見つからず、原因を切り分ける必要がある場合に使用します。

- `exact=true` を使用して HNSW 近似をバイパスするテスト [検索 API](https://search.qdrant.tech/md/documentation/tutorials-search-engineering/retrieval-quality/?s=standard-mode-vs-exact-search)
- 正確な検索が不適切 = モデルまたは検索パイプラインの問題。正確に良い、おおよそ悪い = HNSW を調整します。
- 量子化により品質が低下するかどうかを確認します (量子化の有無を比較)
- フィルターの制限が厳しすぎるかどうかを確認します (その場合は、ACORN を使用する必要がある場合があります)。
- チャンク化されたドキュメントから結果が重複する場合は、グループ化 API を使用して重複を排除します [グループ化](https://search.qdrant.tech/md/documentation/search/search/?s=grouping-api)

ペイロード フィルタリングとスパース ベクトル検索は別のものです。メタデータ (日付、カテゴリ、タグ) は、フィルタリングのためにペイロードに組み込まれます。テキスト コンテンツは検索用にスパース ベクトルに入ります。

## 正確な検索よりも悪い近似検索

次の場合に使用します。完全検索では良好な結果が返されるが、HNSW 近似では結果が得られません。

- クエリ時に `hnsw_ef` を増やす [検索パラメータ](https://search.qdrant.tech/md/documentation/operations/optimize/?s=fine-tuning-search-parameters)
- `ef_construct` を増やします (高品質の場合は 200 以上) [HNSW config](https://search.qdrant.tech/md/documentation/manage-data/indexing/?s=vector-index)
- `m` を増やします (デフォルトは 16、再現率が高い場合は 32) [HNSW config](https://search.qdrant.tech/md/documentation/manage-data/indexing/?s=vector-index)
- オーバーサンプリング + 量子化による再スコアを有効にする [量子化による検索](https://search.qdrant.tech/md/documentation/manage-data/quantization/?s=searching-with-quantization)
- フィルタリングされたクエリ用の ACORN (v1.16+) [ACORN](https://search.qdrant.tech/md/documentation/search/search/?s=acorn-search-algorithm)

バイナリ量子化には再スコアが必要です。これがなければ、品質の低下は深刻です。リコールを回復するには、オーバーサンプリング (バイナリの場合は最小 3 ～ 5 倍) を使用します。実稼働前に、データに対する量子化の影響を必ずテストしてください。 [量子化](https://search.qdrant.tech/md/documentation/manage-data/quantization/)

## 間違った埋め込みモデル

次の場合に使用します。完全一致検索でも悪い結果が返されます。

100 ～ 1000 のサンプル クエリで上位 3 つの MTEB モデルをテストし、リコール @ 10 を測定します。ドメイン固有のモデルは、多くの場合、一般的なモデルよりも優れたパフォーマンスを発揮します。 [ホスト型推論](https://search.qdrant.tech/md/documentation/inference/)

## 最適化されていない検索パイプライン

使用する場合: 完全一致検索でも悪い結果が返され、モデルの選択がユーザーによって確認される場合。

高度な検索戦略スキルに従って検索を最適化します。

## してはいけないこと- モデルがタスクに適していることを確認する前に Qdrant を調整します (品質の問題のほとんどはモデルの問題です)
- スコアを再設定せずにバイナリ量子化を使用します (重大な品質の低下)
- `hnsw_ef` を要求された結果よりも低く設定します (不正な再現が保証されます)
- フィルタリングされたフィールドのペイロード インデックスをスキップし、品質を非難します (HNSW はフィルタリングされたノードを走査できず、フィルタリング可能な HNSW はペイロード インデックスが事前に設定されている場合にのみ構築されます)
- ベースライン再現率やその他の検索関連性指標を使用せずに展開します (回帰を測定する方法がありません)。
- ペイロード フィルタリングとスパース ベクトル検索を混同する (異なるもの、異なる構成)