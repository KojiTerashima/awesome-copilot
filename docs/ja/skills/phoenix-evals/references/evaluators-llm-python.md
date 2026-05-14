# エバリュエーター: Python の LLM エバリュエーター

LLM 評価者は、言語モデルを使用して出力を判断します。基準が主観的な場合に使用します。

## クイックスタート「」パイソン
phoenix.evals より、ClassificationEvaluator、LLM をインポート

llm = LLM(プロバイダー = "openai", モデル = "gpt-4o")

HELPFULNESS_TEMPLATE = """応答がどの程度役に立ったかを評価してください。

<質問>{{入力}}</質問>
<応答>{{出力}}</応答>

「役立つ」とは、質問に直接答えることを意味します。
「not_helpful」は、質問に答えていないことを意味します。

あなたの答え (役に立った/役に立たなかった):"""

有用性 = 分類評価者(
    名前=「役に立つ」、
    プロンプト_テンプレート=HELPFULNESS_TEMPLATE、
    llm=llm、
    選択肢={"役に立たない": 0, "役に立った": 1}
）
「」## テンプレート変数

わかりやすくするために、XML タグを使用して変数をラップします。

|変数 | XML タグ |
| -------- | ------- |
| `{{input}}` | `<question>{{input}}</question>` |
| `{{output}}` | `<response>{{output}}</response>` |
| `{{reference}}` | `<reference>{{reference}}</reference>` |
| `{{context}}` | `<context>{{context}}</context>` |

## create_classifier (ファクトリー)

`ClassificationEvaluator` を返す省略表現ファクトリ。直接を好む
追加のパラメーター/カスタマイズのための `ClassificationEvaluator` インスタンス化:「」パイソン
phoenix.evals から create_classifier、LLM をインポート

関連性 = create_classifier(
    名前 = "関連性",
    prompt_template="""この回答は質問に関連していますか?
<質問>{{入力}}</質問>
<応答>{{出力}}</応答>
回答 (関連/無関係):"""、
    llm=LLM(プロバイダー="openai", モデル="gpt-4o"),
    選択肢={"関連性": 1.0, "無関係": 0.0},
）
「」## 入力マッピング

列名はテンプレート変数と一致する必要があります。列の名前を変更するか、`bind_evaluator` を使用します。「」パイソン
# オプション 1: テンプレート変数に一致するように列の名前を変更します
df = df.rename(columns={"user_query": "入力", "ai_response": "出力"})

# オプション 2:bind_evaluator を使用する
phoenix.evalsからbind_evaluatorをインポート

バウンド = バインド評価者(
    評価者=有用性、
    input_mapping={"入力": "ユーザークエリ", "出力": "ai_response"},
）
「」## ランニング「」パイソン
phoenix.evalsからのインポートevaluate_dataframe

results_df = Evaluate_dataframe(dataframe=df, evaluators=[有用性])
「」## ベストプラクティス

1. **具体的である** - 合格/不合格が何を意味するかを正確に定義する
2. **例を含める** - 各ラベルの具体的なケースを示します
3. **デフォルトの説明** - `ClassificationEvaluator` には説明が自動的に含まれます
4. **組み込みプロンプトを研究する** - を参照してください。
   `phoenix.evals.__generated__.classification_evaluator_configs` の例
   適切に構造化された評価プロンプト (忠実性、正確さ、文書の関連性など)