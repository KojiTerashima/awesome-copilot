# エバリュエーター: Python のコード エバリュエーター

LLM を使用しない決定論的評価器。速く、安く、再現可能。

## 基本パターン「」パイソン
輸入再
jsonをインポートする
phoenix.evals からインポート create_evaluator

@create_evaluator(name="引用あり", kind="コード")
def has_quote(出力: str) -> ブール:
    return bool(re.search(r'\[\d+\]', 出力))

@create_evaluator(name="json_valid", kind="code")
def json_valid(出力: str) -> ブール:
    試してみてください:
        json.loads(出力)
        Trueを返す
    json.JSONDecodeError を除く:
        Falseを返す
「」## パラメータのバインド

|パラメータ |説明 |
| --------- | ----------- |
| `output` |タスクの出力 |
| `input` |入力例 |
| `expected` |期待される出力 |
| `metadata` |メタデータの例 |「」パイソン
@create_evaluator(name="matches_expected", kind="code")
defmatches_expected(出力: str、期待値: dict) -> bool:
    return Output.strip() == Expected.get("answer", "").strip()
「」## 一般的なパターン

- **正規表現**: `re.search(pattern, output)`
- **JSON スキーマ**: `jsonschema.validate()`
- **キーワード**: `keyword in output.lower()`
- **長さ**: `len(output.split())`
- **類似性**: `editdistance.eval()` または Jaccard

## 戻り値の型

|戻り値の型 |結果 |
| ----------- | ------ |
| `bool` | `True` → スコア=1.0、ラベル="True"; `False` → スコア=0.0、ラベル="False" |
| `float`/`int` | `score` 値として直接使用されます。
| `str` (短い、≤3 ワード) | `label` 値として使用される |
| `str` (長い、4 ワード以上) | `explanation` 値として使用されます。
| `dict` と `score`/`label`/`explanation` |スコアフィールドに直接マッピング |
| `Score` オブジェクト |そのまま使用 |

## 重要: コードと LLM の評価

`@create_evaluator` デコレーターは、プレーンな Python 関数をラップします。

- `kind="code"` (デフォルト): LLM を呼び出さない決定的評価の場合。
- `kind="llm"`: エバリュエーターを LLM ベースとしてマークしますが、**あなた** は LLM を実装する必要があります
  関数内で呼び出します。デコレーターは、ユーザーに代わって LLM を呼び出しません。

ほとんどの LLM ベースの評価では、処理を行う `ClassificationEvaluator` を優先します。
LLM 呼び出し、構造化された出力解析、および説明が自動的に行われます。「」パイソン
phoenix.evals より、ClassificationEvaluator、LLM をインポート

関連性 = 分類評価者(
    名前 = "関連性",
    prompt_template="これは関係ありますか?\n{{input}}\n{{output}}\n答え:",
    llm=LLM(プロバイダー="openai", モデル="gpt-4o"),
    選択肢={"関連性": 1.0, "無関係": 0.0},
）
「」## 事前構築済み「」パイソン
phoenix.experiments.evaluators からインポート ContainsAnyKeyword、JSONParseable、MatchesRegex

評価者 = [
    ContainsAnyKeyword(keywords=["免責事項"])、
    JSONParseable()、
    MatchesRegex(pattern=r"\d{4}-\d{2}-\d{2}"),
】
「」
