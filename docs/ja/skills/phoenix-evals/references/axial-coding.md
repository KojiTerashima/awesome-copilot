# アキシャルコーディング

オープンエンドのメモを構造化された障害分類にグループ化します。

## プロセス

1. **収集** - オープンコーディングノートを収集します
2. **パターン** - 共通のテーマを持つグループノート
3. **名前** - 実用的なカテゴリ名を作成します
4. **定量化** - カテゴリごとの失敗の数

## 分類例の例```ヤムル
失敗分類:
  コンテンツの品質:
    幻覚: [発明された事実、フィクションの引用]
    不完全さ: [partial_answer、missing_key_info]
    不正確さ: [間違った番号、間違った日付]
  
  コミュニケーション:
    トーンの不一致: [カジュアルすぎる、フォーマルすぎる]
    明確さ: [曖昧、専門用語が多い]
  
  コンテキスト:
    user_context: [無視された設定、誤解された意図]
    取得されたコンテキスト: [無視されたドキュメント、間違ったコンテキスト]
  
  安全性:
    missing_disclaimers: [法律、医療、財務]
「」## アノテーションの追加 (Python)「」パイソン
phoenix.clientインポートクライアントから

client = クライアント()
client.spans.add_span_annotation(
    スパン_id="abc123",
    annotation_name="失敗カテゴリ",
    ラベル="幻覚",
    description="存在しない機能を発明しました",
    annotator_kind="人間",
    同期=真、
）
「」## 注釈の追加 (TypeScript)```タイプスクリプト
import { addSpanAnnotation } から "@arizeai/phoenix-client/spans";

await addSpanAnnotation({
  スパンアノテーション: {
    スパンID: "abc123",
    名前: "失敗カテゴリ",
    ラベル: 「幻覚」、
    説明: "存在しない機能を発明しました",
    アノテーターの種類: "人間"、
  }
});
「」## エージェント障害分類法```ヤムル
エージェントの失敗:
  計画: [間違った計画、不完全な計画]
  ツール選択: [間違ったツール、見逃したツール、不要な呼び出し]
  ツール実行: [間違ったパラメータ、タイプエラー]
  state_management: [ロストコンテキスト、スタックインループ]
  error_recovery: [no_fallback、間違った_fallback]
「」## 移行マトリックス (エージェント)

状態間で障害が発生する場所を示します。「」パイソン
def build_transition_matrix(会話、状態):
    行列 =defaultdict(lambda:defaultdict(int))
    会話でのコンバージョンの場合:
        conv["失敗"]の場合:
            last_success = find_last_success(conv)
            first_failure = find_first_failure(conv)
            行列[最後の成功][最初の失敗] += 1
    pd.DataFrame(行列).fillna(0) を返す
「」## 原則

- **MECE** - それぞれの失敗は 1 つのカテゴリに当てはまります
- **実用的** - カテゴリが修正を提案します
- **ボトムアップ** - データからカテゴリを出現させる