# Built-in Evaluators

> pixie source code の docstring から自動生成されています。
> 手で編集せず、upstream の [pixie-qa](https://github.com/yiouli/pixie-qa) source repository から再生成してください。

autoevals adapter — `autoevals` の scorer を包んだ、事前定義済み evaluator 群です。

この module は、autoevals の `Scorer` interface を pixie の `Evaluator` protocol に橋渡しする :class:`AutoevalsAdapter` と、一般的な評価タスク向けの factory function 群を提供します。

Public API (すべて `pixie.evals` からも再 export されます):

**Core adapter:** - :class:`AutoevalsAdapter` — 任意の autoevals `Scorer` を包む汎用 wrapper。

**Heuristic scorer (LLM 不要):** - :func:`LevenshteinMatch` — 編集距離ベースの文字列類似度。 - :func:`ExactMatch` — 完全一致比較。 - :func:`NumericDiff` — 正規化された数値差分。 - :func:`JSONDiff` — JSON 構造比較。 - :func:`ValidJSON` — JSON 構文 / schema 検証。 - :func:`ListContains` — 2 つの文字列 list の重なり。

**Embedding scorer:** - :func:`EmbeddingSimilarity` — embedding による cosine similarity。

**LLM-as-judge scorer:** - :func:`Factuality`, :func:`ClosedQA`, :func:`Battle`,
:func:`Humor`, :func:`Security`, :func:`Sql`,
:func:`Summary`, :func:`Translation`, :func:`Possible`.

**Moderation:** - :func:`Moderation` — OpenAI content-moderation check。

**RAGAS metric:** - :func:`ContextRelevancy`, :func:`Faithfulness`,
:func:`AnswerRelevancy`, :func:`AnswerCorrectness`.

## Evaluator Selection Guide

**出力タイプ** と eval criteria に基づいて evaluator を選んでください。

| Output type                                  | Evaluator category                                          | Examples                              |
| -------------------------------------------- | ----------------------------------------------------------- | ------------------------------------- |
| Deterministic (labels, yes/no, fixed-format) | Heuristic: `ExactMatch`, `JSONDiff`, `ValidJSON`            | Label classification, JSON extraction |
| Open-ended text with a reference answer      | LLM-as-judge: `Factuality`, `ClosedQA`, `AnswerCorrectness` | Chatbot responses, QA, summaries      |
| Text with expected context/grounding         | RAG: `Faithfulness`, `ContextRelevancy`                     | RAG pipelines                         |
| Text with style/format requirements          | Custom via `create_llm_evaluator`                           | Voice-friendly responses, tone checks |
| Multi-aspect quality                         | Multiple evaluators combined                                | Factuality + relevance + tone         |

重要なルール:

- open-ended な LLM text には **絶対に** `ExactMatch` を使わないでください。LLM 出力は非決定的です。
- `AnswerRelevancy` は **RAG 専用** です。trace に `context` が必要で、ない場合は 0.0 を返します。一般的な relevance には `create_llm_evaluator` を使ってください。
- `expected_output` のない item に比較系 evaluator (`Factuality`, `ClosedQA`, `ExactMatch`) を使ってはいけません。意味のない score になります。

---

## Evaluator Reference

### `AnswerCorrectness`

```python
AnswerCorrectness(*, client: 'Any' = None) -> 'AutoevalsAdapter'
```

回答正確性 evaluator (RAGAS)。

`eval_output` が `expected_output` に対してどれだけ正しいかを、事実的類似性と意味的類似性を組み合わせて判定します。

**When to use**: reference answer がある RAG pipeline の QA scenario で、包括的な correctness score が欲しい場合。

**Requires `expected_output`**: Yes.
**Requires `eval_metadata["context"]`**: Optional (あると精度向上)。

Args:
client: OpenAI client instance.

### `AnswerRelevancy`

```python
AnswerRelevancy(*, client: 'Any' = None) -> 'AutoevalsAdapter'
```

回答関連性 evaluator (RAGAS)。

`eval_output` が `eval_input` 内の質問に直接答えているかを判定します。

**When to use**: RAG pipeline のみ。trace に `context` が必要です。ない場合は 0.0 を返します。一般的な (non-RAG) relevance には custom prompt を伴う `create_llm_evaluator` を使ってください。

**Requires `expected_output`**: No.
**Requires `eval_metadata["context"]`**: Yes — **RAG pipeline only**.

Args:
client: OpenAI client instance.

### `Battle`

```python
Battle(*, model: 'str | None' = None, client: 'Any' = None) -> 'AutoevalsAdapter'
```

head-to-head 比較 evaluator (LLM-as-judge)。

LLM を使って `eval_output` と `expected_output` を比較し、`eval_input` の instructions を踏まえてどちらが優れているかを判定します。

**When to use**: A/B test、model output の比較、代替応答の順位付け。

**Requires `expected_output`**: Yes.

Args:
model: LLM model name.
client: OpenAI client instance.

### `ClosedQA`

```python
ClosedQA(*, model: 'str | None' = None, client: 'Any' = None) -> 'AutoevalsAdapter'
```

closed-book question-answering evaluator (LLM-as-judge)。

LLM を使って、`eval_output` が `eval_input` の質問に対し `expected_output` と比べて正しく答えているかを判定します。必要に応じて `eval_metadata["criteria"]` を custom grading criteria として渡します。

**When to use**: reference と照合すべき QA scenario。例: customer support answer、knowledge-base query。

**Requires `expected_output`**: Yes — `expected_output` のない item に使ってはいけません。意味のない score になります。

Args:
model: LLM model name.
client: OpenAI client instance.

### `ContextRelevancy`

```python
ContextRelevancy(*, client: 'Any' = None) -> 'AutoevalsAdapter'
```

context 関連性 evaluator (RAGAS)。

取得された context が query に relevant かを判定します。`eval_metadata["context"]` を内部 scorer へ渡します。

**When to use**: RAG pipeline で retrieval quality を評価するとき。

**Requires `expected_output`**: Yes.
**Requires `eval_metadata["context"]`**: Yes (RAG pipelines only).

Args:
client: OpenAI client instance.

### `EmbeddingSimilarity`

```python
EmbeddingSimilarity(*, prefix: 'str | None' = None, model: 'str | None' = None, client: 'Any' = None) -> 'AutoevalsAdapter'
```

embedding ベースの意味類似度 evaluator。

`eval_output` と `expected_output` の embedding vector 間 cosine similarity を計算します。

**When to use**: 正確な wording より意味の近さが重要な場合。言い換えに対しては Levenshtein より頑健ですが、LLM-as-judge よりはニュアンスが少ないです。

**Requires `expected_output`**: Yes.

Args:
prefix: domain context のために先頭へ付ける任意の text。
model: Embedding model name.
client: OpenAI client instance.

### `ExactMatch`

```python
ExactMatch() -> 'AutoevalsAdapter'
```

完全一致比較 evaluator。

`eval_output` が `expected_output` と完全に一致すれば 1.0、そうでなければ 0.0 を返します。

**When to use**: 決定的で構造化された output (classification label、yes/no、fixed-format string)。open-ended な LLM text に使ってはいけません。ほぼ失敗します。

**Requires `expected_output`**: Yes.

### `Factuality`

```python
Factuality(*, model: 'str | None' = None, client: 'Any' = None) -> 'AutoevalsAdapter'
```

事実正確性 evaluator (LLM-as-judge)。

LLM を使って、`eval_output` が `eval_input` 文脈のもとで `expected_output` と事実的に整合しているかを判定します。

**When to use**: 事実正確性が重要な open-ended text (chatbot response、QA answer、summary)。LLM 生成 text では `ExactMatch` より適切です。

**Requires `expected_output`**: Yes — `expected_output` のない item には使ってはいけません。

Args:
model: LLM model name.
client: OpenAI client instance.

### `Faithfulness`

```python
Faithfulness(*, client: 'Any' = None) -> 'AutoevalsAdapter'
```

faithfulness evaluator (RAGAS)。

`eval_output` が与えられた context に忠実か (つまり context に支えられているか) を判定します。`eval_metadata["context"]` を渡します。

**When to use**: RAG pipeline で、取得 context を超えた hallucination がないことを確認したい場合。

**Requires `expected_output`**: No.
**Requires `eval_metadata["context"]`**: Yes (RAG pipelines only).

Args:
client: OpenAI client instance.

### `Humor`

```python
Humor(*, model: 'str | None' = None, client: 'Any' = None) -> 'AutoevalsAdapter'
```

ユーモア品質 evaluator (LLM-as-judge)。

LLM を使って、`eval_output` のユーモア品質を `expected_output` と比較して判定します。

**When to use**: creative writing、chatbot personality、entertainment application における humor の評価。

**Requires `expected_output`**: Yes.

Args:
model: LLM model name.
client: OpenAI client instance.

### `JSONDiff`

```python
JSONDiff(*, string_scorer: 'Any' = None) -> 'AutoevalsAdapter'
```

JSON 構造比較 evaluator。

2 つの JSON 構造を再帰的に比較し、類似度 score を生成します。nested object、array、混在型を扱えます。

**When to use**: field 単位の比較が必要な構造化 JSON output (抽出 data、API response schema、tool call argument など)。

**Requires `expected_output`**: Yes.

Args:
string_scorer: string field に使う任意の pairwise scorer。

### `LevenshteinMatch`

```python
LevenshteinMatch() -> 'AutoevalsAdapter'
```

編集距離ベースの文字列類似度 evaluator。

`eval_output` と `expected_output` の正規化 Levenshtein distance を計算します。同一文字列なら 1.0 で、編集距離が大きいほど score が下がります。

**When to use**: 決定的、またはほぼ決定的な output で、小さな文字差異だけ許容したい場合。open-ended な LLM text には不向きです。

**Requires `expected_output`**: Yes.

### `ListContains`

```python
ListContains(*, pairwise_scorer: 'Any' = None, allow_extra_entities: 'bool' = False) -> 'AutoevalsAdapter'
```

list 重なり evaluator。

`eval_output` が `expected_output` のすべての item を含むかを判定し、重なり率に基づいて score を付けます。

**When to use**: list 形式 output で完全性が重要な場合 (抽出 entity、search result、recommendation など)。

**Requires `expected_output`**: Yes.

Args:
pairwise_scorer: 要素同士を比較する任意 scorer。
allow_extra_entities: True の場合、output 側の余分な item を減点しない。

### `Moderation`

```python
Moderation(*, threshold: 'float | None' = None, client: 'Any' = None) -> 'AutoevalsAdapter'
```

content moderation evaluator。

OpenAI moderation API を使って、`eval_output` に unsafe content (hate speech、violence、self-harm など) がないかを判定します。

**When to use**: 出力安全性が重要な任意の application。chatbot、content generation、user-facing AI など。

**Requires `expected_output`**: No.

Args:
threshold: custom flagging threshold。
client: OpenAI client instance.

### `NumericDiff`

```python
NumericDiff() -> 'AutoevalsAdapter'
```

正規化された数値差分 evaluator。

`eval_output` と `expected_output` の数値距離を正規化して計算します。同一数値なら 1.0 で、差が大きいほど score が下がります。

**When to use**: おおよその一致を許容できる数値 output (価格計算、score、測定値など)。

**Requires `expected_output`**: Yes.

### `Possible`

```python
Possible(*, model: 'str | None' = None, client: 'Any' = None) -> 'AutoevalsAdapter'
```

実現可能性 / 妥当性 evaluator (LLM-as-judge)。

LLM を使って、`eval_output` が plausible か、feasible かを判定します。

**When to use**: 具体的な reference answer なしで、output が reasonable かを一般的に確認したい場合。

**Requires `expected_output`**: No.

Args:
model: LLM model name.
client: OpenAI client instance.

### `Security`

```python
Security(*, model: 'str | None' = None, client: 'Any' = None) -> 'AutoevalsAdapter'
```

セキュリティ脆弱性 evaluator (LLM-as-judge)。

LLM を使って、`eval_input` の instructions に照らして `eval_output` に脆弱性がないかを確認します。

**When to use**: code generation、SQL output、または injection/vulnerability risk を確認すべき scenario。

**Requires `expected_output`**: No.

Args:
model: LLM model name.
client: OpenAI client instance.

### `Sql`

```python
Sql(*, model: 'str | None' = None, client: 'Any' = None) -> 'AutoevalsAdapter'
```

SQL 等価性 evaluator (LLM-as-judge)。

LLM を使って、`eval_output` SQL が `expected_output` SQL と意味的に等価かを判定します。

**When to use**: text-to-SQL application で、生成 SQL が reference query と機能的に同等であるべき場合。

**Requires `expected_output`**: Yes.

Args:
model: LLM model name.
client: OpenAI client instance.

### `Summary`

```python
Summary(*, model: 'str | None' = None, client: 'Any' = None) -> 'AutoevalsAdapter'
```

要約品質 evaluator (LLM-as-judge)。

LLM を使って、`eval_output` が `expected_output` の reference summary と比べてどれだけ良い summary かを判定します。

**When to use**: 出力が source material の key information を捉える必要がある summarisation task。

**Requires `expected_output`**: Yes.

Args:
model: LLM model name.
client: OpenAI client instance.

### `Translation`

```python
Translation(*, language: 'str | None' = None, model: 'str | None' = None, client: 'Any' = None) -> 'AutoevalsAdapter'
```

翻訳品質 evaluator (LLM-as-judge)。

LLM を使って、target language における `eval_output` の翻訳品質を `expected_output` と比較して判定します。

**When to use**: machine translation や multilingual output の scenario。

**Requires `expected_output`**: Yes.

Args:
language: target language (例: `"Spanish"`)。
model: LLM model name.
client: OpenAI client instance.

### `ValidJSON`

```python
ValidJSON(*, schema: 'Any' = None) -> 'AutoevalsAdapter'
```

JSON 構文および schema 検証 evaluator。

`eval_output` が妥当な JSON であり (任意で指定 schema に一致する) 場合は 1.0、そうでなければ 0.0 を返します。

**When to use**: valid JSON である必要がある output。必要に応じて specific schema への適合も確認できます (tool call response、structured extraction など)。

**Requires `expected_output`**: No.

Args:
schema: 検証に使う任意の JSON Schema。

---

## Custom Evaluators: `create_llm_evaluator`

prompt template から custom LLM-as-judge evaluator を作る factory です。

Usage::

    from pixie import create_llm_evaluator

    concise_voice_style = create_llm_evaluator(
        name="ConciseVoiceStyle",
        prompt_template="""
        You are evaluating whether a voice agent response is concise and
        phone-friendly.

        User said: {eval_input}
        Agent responded: {eval_output}
        Expected behavior: {expectation}

        Score 1.0 if the response is concise (under 3 sentences), directly
        addresses the question, and uses conversational language suitable for
        a phone call. Score 0.0 if it's verbose, off-topic, or uses
        written-style formatting.
        """,
    )

### `create_llm_evaluator`

```python
create_llm_evaluator(name: 'str', prompt_template: 'str', *, model: 'str' = 'gpt-4o-mini', client: 'Any | None' = None) -> '_LLMEvaluator'
```

prompt template から custom LLM-as-judge evaluator を作成します。

template では次の変数が使えます (:class:`~pixie.storage.evaluable.Evaluable` field から埋められます):

- `{eval_input}` — evaluable の input data。single-item list の場合はその value、multi-item list の場合は `name → value` の JSON dict に展開されます。
- `{eval_output}` — evaluable の output data (`eval_input` と同じ規則)。
- `{expectation}` — evaluable の expected output。

Args:
name: evaluator の表示名 (scorecard に表示)。
prompt_template: `{eval_input}`、`{eval_output}`、`{expectation}` placeholder を持つ string template。
model: OpenAI model name (default: `gpt-4o-mini`)。
client: 任意の pre-configured OpenAI client instance。

Returns:
`Evaluator` protocol を満たす evaluator callable。

Raises:
ValueError: `{eval_input[key]}` のような nested field access を template に使った場合 (サポートされるのは top-level placeholder のみ)。
