# Evaluate_dataframe によるバッチ評価 (Python)

データフレーム全体でエバリュエーターを実行します。コア 2.0 バッチ評価 API。

## 推奨: async_evaluate_dataframe

バッチ評価 (特に LLM エバリュエーターを使用) の場合は、非同期バージョンを推奨します。
スループットを向上させるには:「」パイソン
phoenix.evals から async_evaluate_dataframe をインポートします

results_df = await async_evaluate_dataframe(
    dataframe=df, # pandas エバリュエーターパラメータに一致する列を持つデータフレーム
    evaluators=[eval1, eval2], # 評価者のリスト
    concurrency=5、最大同時 LLM 呼び出し数 (デフォルトは 3)
    exit_on_error=False、# オプション: 最初のエラーで停止 (デフォルトは True)
    max_retries=3, # オプション: 失敗した LLM 呼び出しを再試行します (デフォルトは 10)
）
「」## 同期バージョン「」パイソン
phoenix.evalsからのインポートevaluate_dataframe

results_df = 評価データフレーム(
    dataframe=df, # pandas エバリュエーターパラメータに一致する列を持つデータフレーム
    evaluators=[eval1, eval2], # 評価者のリスト
    exit_on_error=False、# オプション: 最初のエラーで停止 (デフォルトは True)
    max_retries=3, # オプション: 失敗した LLM 呼び出しを再試行します (デフォルトは 10)
）
「」## 結果列の形式

`async_evaluate_dataframe` / `evaluate_dataframe` は、列が追加された入力データフレームのコピーを返します。
**結果列には、生の数値ではなく、辞書が含まれます。**

`"foo"` という名前の評価者ごとに、次の 2 つの列が追加されます。

|コラム |タイプ |目次 |
| ------ | ---- | -------- |
| `foo_score` | `dict` | `{"name": "foo", "score": 1.0, "label": "True", "explanation": "...", "metadata": {...}, "kind": "code", "direction": "maximize"}` |
| `foo_execution_details` | `dict` | `{"status": "success", "exceptions": [], "execution_seconds": 0.001}` |

None 以外のフィールドのみがスコア辞書に表示されます。

### 数値スコアの抽出「」パイソン
# 間違っています - これらは失敗するか、予期しない結果をもたらします
スコア = results_df["関連性"].mean() # KeyError!
スコア = results_df["relevance_score"].mean() # 辞書の平均を試みます!

# RIGHT — 各辞書から数値スコアを抽出します
スコア = results_df["関連性_スコア"].apply(
    ラムダ x: x.get("スコア", 0.0) if isinstance(x, dict) else 0.0
）
平均スコア = スコア.平均()
「」### ラベルの抽出「」パイソン
ラベル = results_df["関連性_スコア"].apply(
    ラムダ x: x.get("label", "") if isinstance(x, dict) else ""
）
「」### 説明の抽出 (LLM 評価者)「」パイソン
説明 = results_df["関連性_スコア"].apply(
    ラムダ x: x.get("説明", "") if isinstance(x, dict) else ""
）
「」### 失敗の発見「」パイソン
スコア = results_df["関連性_スコア"].apply(
    ラムダ x: x.get("スコア", 0.0) if isinstance(x, dict) else 0.0
）
failed_mask = スコア < 0.5
失敗 = results_df[失敗マスク]
「」## 入力マッピング

評価者は各行を辞書として受け取ります。列名は評価者の名前と一致する必要があります
予期されるパラメータ名。一致しない場合は、`.bind()` または `bind_evaluator` を使用します。「」パイソン
phoenix.evalsからインポートbind_evaluator、create_evaluator、async_evaluate_dataframe

@create_evaluator(name="チェック", kind="コード")
def check(response: str) -> bool:
    戻り値 len(response.strip()) > 0

# オプション 1: エバリュエーターで .bind() メソッドを使用する
check.bind(input_mapping={"response": "answer"})
results_df = 待機 async_evaluate_dataframe(dataframe=df, evaluators=[check])

# オプション 2:bind_evaluator 関数を使用する
bound = binding_evaluator(evaluator=check, input_mapping={"response": "answer"})
results_df = 待機 async_evaluate_dataframe(dataframe=df, evaluators=[bound])
「」または、一致するように単に列の名前を変更します。「」パイソン
df = df.rename(columns={
    "attributes.input.value": "入力",
    "attributes.output.value": "出力",
})
「」## run_evals は使用しないでください「」パイソン
# 間違っています - レガシー 1.0 API
phoenix.evals から run_evals をインポート
結果 = run_evals(データフレーム=df、評価者=[eval1])
# List[DataFrame] を返します — エバリュエーターごとに 1 つ

# 右 - 現在の 2.0 API
phoenix.evals から async_evaluate_dataframe をインポートします
results_df = 待機 async_evaluate_dataframe(dataframe=df, evaluators=[eval1])
# {name}_score dict 列を持つ単一の DataFrame を返します
「」主な違い:
- `run_evals` は、DataFrame の **リスト** を返します (評価者ごとに 1 つ)
- `async_evaluate_dataframe` は、すべての結果がマージされた **単一** データフレームを返します
- `async_evaluate_dataframe` は `{name}_score` dict 列形式を使用します
- `async_evaluate_dataframe` は入力マッピングに `bind_evaluator` を使用します (`input_mapping=` パラメータではありません)