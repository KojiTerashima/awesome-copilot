# Step 3: Define Evaluators

**Why this step**: アプリを instrument したので (Step 2)、次は各 eval criterion を具体的な evaluator に対応付けます。必要なら custom evaluator も実装し、その後の dataset (Step 4) から名前で参照できるようにします。

---

## 3a. Map criteria to evaluators

**Step 1b のすべての eval criterion — prompt 内でユーザーが指定した dimension を含む — には、対応する evaluator が必ず必要です。** ユーザーが "factuality, completeness, and bias" を求めたなら、3 つの evaluator (またはそれらをすべてカバーする multi-criteria evaluator) が必要です。要求された dimension を黙って落としてはいけません。

各 eval criterion について、どう評価するかを決めます。

- **Can it be checked with a built-in evaluator?** (`Factuality`、`ExactMatch`、`Faithfulness` など)
- **Does it need a custom evaluator?** 多くの app-specific criterion は必要です。criterion を operationalize する prompt を使って `create_llm_evaluator` を使います。
- **Is it universal or case-specific?** universal criterion は全 dataset item に適用されます。case-specific criterion は一部の行だけに適用されます。

open-ended な LLM text には **絶対に** `ExactMatch` を使ってはいけません。LLM 出力は非決定的です。

`AnswerRelevancy` は **RAG 専用** です。trace に `context` value が必要で、それがないと 0.0 を返します。RAG ではない一般的な relevance には、custom prompt を使った `create_llm_evaluator` を使ってください。

## 3b. Implement custom evaluators

criterion に custom evaluator が必要なら、今ここで実装します。custom evaluator は `pixie_qa/evaluators.py` に置いてください (数が多ければ sub-module でも可)。

### `create_llm_evaluator` factory

品質次元が domain-specific で built-in evaluator が合わないときに使います。

返り値は **そのまま使える evaluator instance** です。module-level variable に代入してください。`pixie test` はそれを import してそのまま使います (class wrapper は不要です)。

```python
from pixie import create_llm_evaluator

concise_voice_style = create_llm_evaluator(
    name="ConciseVoiceStyle",
    prompt_template="""
    You are evaluating whether this response is concise and phone-friendly.

    Input: {eval_input}
    Response: {eval_output}

    Score 1.0 if the response is concise (under 3 sentences), directly addresses
    the question, and uses conversational language suitable for a phone call.
    Score 0.0 if it's verbose, off-topic, or uses written-style formatting.
    """,
)
```

この evaluator は dataset JSON から `filepath:callable_name` 形式 (例: `"pixie_qa/evaluators.py:concise_voice_style"`) で参照します。

**How template variables work**: placeholder は `{eval_input}`, `{eval_output}`, `{expectation}` の 3 つだけです。それぞれ対応する `Evaluable` field の文字列表現に置き換えられます。

- **Single-item** `eval_input` / `eval_output` → その item の値 (string、JSON-serialized dict/list)
- **Multi-item** `eval_input` / `eval_output` → すべての item に対する `name → value` の JSON dict

LLM judge は、このシリアライズ済み値全体を見ます。

**Rules**:

- **Use only `{eval_input}`, `{eval_output}`, `{expectation}`** — `{eval_input[key]}` のような nested access は不可 (`ValueError` で落ちます)
- **Template は短く直接的に** — system prompt 側がすでに `Score: X.X` を返すよう指示しています。template は data を提示し、採点基準を定義するだけで十分です。
- **LLM に "parse" や "extract" を指示しない** — 値をそのまま提示し、criteria を述べるだけでよいです。LLM は JSON を自然に読めます。

**Non-RAG response relevance** (`AnswerRelevancy` の代わり):

```python
response_relevance = create_llm_evaluator(
    name="ResponseRelevance",
    prompt_template="""
    You are evaluating whether a customer support response is relevant and helpful.

    Input: {eval_input}
    Response: {eval_output}
    Expected: {expectation}

    Score 1.0 if the response directly addresses the question and meets expectations.
    Score 0.5 if partially relevant but misses important aspects.
    Score 0.0 if off-topic, ignores the question, or contradicts expectations.
    """,
)
```

### Manual custom evaluator

custom evaluator は **sync function でも async function でも**かまいません。module-level variable として `pixie_qa/evaluators.py` に置いてください。

```python
from pixie import Evaluation, Evaluable

def my_evaluator(evaluable: Evaluable, *, trace=None) -> Evaluation:
    score = 1.0 if "expected pattern" in str(evaluable.eval_output) else 0.0
    return Evaluation(score=score, reasoning="...")
```

dataset では `filepath:callable_name` で参照します: `"pixie_qa/evaluators.py:my_evaluator"`.

**Accessing `eval_metadata` and captured data**: custom evaluator は、entry ごとの metadata と `wrap()` output を `Evaluable` field 経由で参照します。

- `evaluable.eval_metadata` — entry の `eval_metadata` field 由来の dict (例: `{"expected_tool": "endCall"}`)
- `evaluable.eval_output` — すべての `wrap(purpose="output")` と `wrap(purpose="state")` の値を含む `list[NamedData]`。各 item は `.name` (str) と `.value` (JsonValue) を持ちます。以下の helper を使って name で取り出せます。

```python
def _get_output(evaluable: Evaluable, name: str) -> Any:
    """Look up a wrap value by name from eval_output."""
    for item in evaluable.eval_output:
        if item.name == name:
            return item.value
    return None

def call_ended_check(evaluable: Evaluable, *, trace=None) -> Evaluation:
    expected = evaluable.eval_metadata.get("expected_call_ended") if evaluable.eval_metadata else None
    actual = _get_output(evaluable, "call_ended")
    if expected is None:
        return Evaluation(score=1.0, reasoning="No expected_call_ended in eval_metadata")
    match = bool(actual) == bool(expected)
    return Evaluation(
        score=1.0 if match else 0.0,
        reasoning=f"Expected call_ended={expected}, got {actual}",
    )
```

## 3c. Produce the evaluator mapping artifact

criterion-to-evaluator mapping を `pixie_qa/03-evaluator-mapping.md` に書いてください。この artifact は Step 1b の eval criteria と Step 4 の dataset の橋渡しになります。

**CRITICAL**: `evaluators.md` reference に書かれている evaluator 名をそのまま使ってください。built-in evaluator は短い名前 (例: `Factuality`, `ClosedQA`)、custom evaluator は `filepath:callable_name` 形式 (例: `pixie_qa/evaluators.py:ConciseVoiceStyle`) を使います。

### Template

```markdown
# Evaluator Mapping

## Built-in evaluators used

| Evaluator name | Criterion it covers | Applies to                 |
| -------------- | ------------------- | -------------------------- |
| Factuality     | Factual accuracy    | All items                  |
| ClosedQA       | Answer correctness  | Items with expected_output |

## Custom evaluators

| Evaluator name                           | Criterion it covers | Applies to | Source file            |
| ---------------------------------------- | ------------------- | ---------- | ---------------------- |
| pixie_qa/evaluators.py:ConciseVoiceStyle | Phone-friendly tone | All items  | pixie_qa/evaluators.py |

## Applicability summary

- **Dataset-level defaults** (apply to all items): Factuality, pixie_qa/evaluators.py:ConciseVoiceStyle
- **Item-specific** (apply to subset): ClosedQA (only items with expected_output)
```

## Output

- `pixie_qa/evaluators.py` 内の custom evaluator 実装 (必要な場合)
- `pixie_qa/03-evaluator-mapping.md` — criterion-to-evaluator mapping

---

> **Evaluator selection guide**: evaluator の完全一覧、出力タイプごとの選び方、`create_llm_evaluator` の reference については `evaluators.md` を参照してください。
>
> **If you hit an unexpected error** when implementing evaluators (import 失敗、API mismatch など) は、勘で直す前に `evaluators.md` の authoritative reference と `wrap-api.md` の API 詳細を読んでください。
