# Testing API Reference

> pixie source code の docstring から自動生成されています。
> 手で編集せず、upstream の [pixie-qa](https://github.com/yiouli/pixie-qa) source repository から再生成してください。

pixie.evals — LLM application 向け evaluation harness。

Public API: - `Evaluation` — 単一 evaluator 実行結果の dataclass - `Evaluator` — evaluation callable の protocol - `evaluate` — 1 evaluator を 1 evaluable に適用 - `run_and_evaluate` — MemoryTraceHandler から得た span を評価 - `assert_pass` — pass/fail criteria 付き batch evaluation - `assert_dataset_pass` — dataset を読み込んで assert_pass を実行 - `EvalAssertionError` — assert_pass 失敗時に送出 - `capture_traces` — in-memory trace capture 用 context manager - `MemoryTraceHandler` — span を集める InstrumentationHandler - `ScoreThreshold` — 設定可能な pass criteria - `last_llm_call` / `root` — trace から evaluable を作る helper - `DatasetEntryResult` — 単一 dataset entry の evaluation result - `DatasetScorecard` — evaluator が非一様でも扱える per-dataset scorecard - `generate_dataset_scorecard_html` — scorecard を HTML として描画 - `save_dataset_scorecard` — scorecard HTML を disk に書き出す

事前定義 evaluator (autoevals adapter): - `AutoevalsAdapter` — 任意の autoevals `Scorer` を包む汎用 wrapper - `LevenshteinMatch` — 編集距離ベースの文字列類似度 - `ExactMatch` — 完全一致比較 - `NumericDiff` — 正規化された数値差分 - `JSONDiff` — JSON 構造比較 - `ValidJSON` — JSON 構文 / schema 検証 - `ListContains` — list overlap - `EmbeddingSimilarity` — embedding cosine similarity - `Factuality` — LLM 事実正確性チェック - `ClosedQA` — closed-book QA evaluation - `Battle` — head-to-head 比較 - `Humor` — humor detection - `Security` — security vulnerability check - `Sql` — SQL 等価性 - `Summary` — 要約品質 - `Translation` — 翻訳品質 - `Possible` — feasibility check - `Moderation` — content moderation - `ContextRelevancy` — RAGAS context relevancy - `Faithfulness` — RAGAS faithfulness - `AnswerRelevancy` — RAGAS answer relevancy - `AnswerCorrectness` — RAGAS answer correctness

## Dataset JSON Format

dataset は次の top-level field を持つ JSON object です。

```json
{
  "name": "customer-faq",
  "runnable": "pixie_qa/scripts/run_app.py:AppRunnable",
  "evaluators": ["Factuality"],
  "entries": [
    {
      "entry_kwargs": { "question": "Hello" },
      "description": "Basic greeting",
      "eval_input": [{ "name": "input", "value": "Hello" }],
      "expectation": "A friendly greeting that offers to help",
      "evaluators": ["...", "ClosedQA"]
    }
  ]
}
```

### Entry structure

各 field は entry の top-level に置きます (flat structure — nesting しない):

```
entry:
  ├── entry_kwargs    (required) — Runnable.run() 用 args
  ├── eval_input      (required) — {"name": ..., "value": ...} object の list
  ├── description     (required) — test case の人間向けラベル
  ├── expectation     (optional) — comparison-based evaluator 用の参照
  ├── eval_metadata   (optional) — custom evaluator 用の追加 per-entry data
  └── evaluators      (optional) — この entry 専用の evaluator 名
```

### Field reference

- `runnable` (required): evaluation 中に app を駆動する `Runnable` subclass への `filepath:ClassName` 参照。
- `evaluators` (dataset-level, optional): default evaluator 名。独自 `evaluators` を持たない全 entry に適用される。
- `entries[].entry_kwargs` (required): `Runnable.run()` に Pydantic model として渡される kwargs。key は `run(args: T)` に使う Pydantic model の field と一致しなければならない。
- `entries[].description` (required): test case を表す人間向けラベル。
- `entries[].eval_input` (required): `{"name": ..., "value": ...}` object の list。wrap input registry を埋めるために使われ、アプリ内の `wrap(purpose="input")` は `name` を key として registry value を返す。
- `entries[].expectation` (optional): comparison-based evaluator 用の簡潔な expectation 説明。**実出力をそのままコピーしてはいけません**。`pixie format` で trace の実 output shape を見て、それをより短く説明してください。
- `entries[].eval_metadata` (optional): custom evaluator 用の追加 per-entry data。例: expected tool 名、boolean flag、threshold。evaluator では `evaluable.eval_metadata` として参照。
- `entries[].evaluators` (optional): row-level evaluator override。ルール:
  - 省略 → entry は dataset-level `evaluators` を継承
  - `["...", "ClosedQA"]` → dataset default **に加えて** ClosedQA
  - `["OnlyThis"]` (`"..."` なし) → **OnlyThis のみ**、default なし

## Evaluator Name Resolution

dataset JSON では evaluator 名は次のように解決されます。

- **Built-in name** (`"Factuality"`、`"ExactMatch"` のような bare name) は自動的に `pixie.{Name}` として解決される
- **Custom evaluator** は `filepath:callable_name` 形式を使う (例: `"pixie_qa/evaluators.py:my_evaluator"`)
- custom evaluator 参照は module-level callable を指す。class (自動 instantiate)、factory function (ゼロ引数なら呼び出し)、evaluator function (そのまま使用)、pre-instantiated callable (`create_llm_evaluator` の結果など) を扱える

## CLI Commands

| Command                                     | Description                           |
| ------------------------------------------- | ------------------------------------- |
| `pixie test [path] [-v] [--no-open]`        | dataset file に対して eval test を実行 |
| `pixie dataset create <name>`               | 新しい空 dataset を作成               |
| `pixie dataset list`                        | すべての dataset を一覧表示           |
| `pixie dataset save <name> [--select MODE]` | span を dataset に保存                |
| `pixie dataset validate [path]`             | dataset JSON file を検証              |
| `pixie analyze <test_run_id>`               | analysis と recommendation を生成     |

---

## Types

### `Evaluable`

```python
class Evaluable(TestCase):
    eval_output: list[NamedData]      # wrap(purpose="output") + wrap(purpose="state") values
    # Inherited from TestCase:
    # eval_input: list[NamedData]     # from eval_input in dataset entry
    # expectation: JsonValue | _Unset # from expectation in dataset entry
    # eval_metadata: dict[str, JsonValue] | None  # from eval_metadata in dataset entry
    # description: str | None
```

evaluator 向けの data carrier。actual output を追加した `TestCase` 拡張です。

- `eval_input` — entry の `eval_input` field から埋まる `list[NamedData]`。**最低 1 件必要** (`min_length=1`)。
- `eval_output` — 実行中に捕捉された、すべての `wrap(purpose="output")` と `wrap(purpose="state")` の値を持つ `list[NamedData]`。各 item は `.name` (str) と `.value` (JsonValue) を持つ。name での参照には `_get_output(evaluable, "name")` を使う。
- `eval_metadata` — entry の `eval_metadata` field 由来の `dict[str, JsonValue] | None`
- `expected_output` — dataset 由来の expectation text (`UNSET` の場合もありうる)

Attributes:
eval_input: dataset 由来の named input data item。非空である必要がある。
eval_output: 実行時の wrap call 由来の named output data item。
各 item は `.name` (str) と `.value` (JsonValue) を持つ。
`wrap(purpose="output")` と `wrap(purpose="state")` の値をすべて含む。
eval_metadata: 補助 metadata (ない場合は `None`)。
expected_output: 評価用の expected/reference output。
既定では `UNSET` (未提供)。明示的に `None` を指定して
"expected output はない" ことを表すこともできる。

### How `wrap()` maps to `Evaluable` fields at test time

`pixie test` が dataset entry を実行するとき、アプリ内の `wrap()` call は evaluator に渡される `Evaluable` を埋めます。

| `wrap()` call in app code                | Evaluable field   | Type              | How to access in evaluator                           |
| ---------------------------------------- | ----------------- | ----------------- | ---------------------------------------------------- |
| `wrap(data, purpose="input", name="X")`  | `eval_input`      | `list[NamedData]` | dataset entry の `eval_input` から事前投入される     |
| `wrap(data, purpose="output", name="X")` | `eval_output`     | `list[NamedData]` | `_get_output(evaluable, "X")` — 下記 helper 参照   |
| `wrap(data, purpose="state", name="X")`  | `eval_output`     | `list[NamedData]` | `_get_output(evaluable, "X")` — output と同じ list |
| (from dataset entry `expectation`)       | `expected_output` | `str \| None`     | `evaluable.expected_output`                          |
| (from dataset entry `eval_metadata`)     | `eval_metadata`   | `dict \| None`    | `evaluable.eval_metadata`                            |

**Key insight**: `purpose="output"` と `purpose="state"` の wrap value は、どちらも `eval_output` 内の `NamedData` item として入ります。別個の `captured_output` や `captured_state` dict はありません。name で値を引くには次の helper を使います。

```python
def _get_output(evaluable: Evaluable, name: str) -> Any:
    """Look up a wrap value by name from eval_output."""
    for item in evaluable.eval_output:
        if item.name == name:
            return item.value
    return None
```

**`eval_metadata`** は、app input/output ではない追加情報を evaluator に渡すためのものです。例: expected tool name、boolean flag、threshold。entry の top-level field として定義し、`evaluable.eval_metadata` として参照します。

**Complete custom evaluator example** (tool call check + dataset entry):

```python
from pixie import Evaluation, Evaluable

def _get_output(evaluable: Evaluable, name: str) -> Any:
    """Look up a wrap value by name from eval_output."""
    for item in evaluable.eval_output:
        if item.name == name:
            return item.value
    return None

def tool_call_check(evaluable: Evaluable, *, trace=None) -> Evaluation:
    expected = evaluable.eval_metadata.get("expected_tool") if evaluable.eval_metadata else None
    actual = _get_output(evaluable, "function_called")
    if expected is None:
        return Evaluation(score=1.0, reasoning="No expected_tool specified")
    match = str(actual) == str(expected)
    return Evaluation(
        score=1.0 if match else 0.0,
        reasoning=f"Expected {expected}, got {actual}",
    )
```

対応する dataset entry:

```json
{
  "entry_kwargs": { "user_message": "I want to end this call" },
  "description": "User requests call end after failed verification",
  "eval_input": [{ "name": "user_input", "value": "I want to end this call" }],
  "expectation": "Agent should call endCall tool",
  "eval_metadata": {
    "expected_tool": "endCall",
    "expected_call_ended": true
  },
  "evaluators": ["...", "pixie_qa/evaluators.py:tool_call_check"]
}
```

### `Evaluation`

```python
Evaluation(score: 'float', reasoning: 'str', details: 'dict[str, Any]' = <factory>) -> None
```

1 つの evaluator を 1 つの test case に適用した結果です。

Attributes:
score: 0.0〜1.0 の evaluation score。
reasoning: 人が読める説明 (必須)。
details: 任意の JSON-serializable metadata。

### `ScoreThreshold`

```python
ScoreThreshold(threshold: 'float' = 0.5, pct: 'float' = 1.0) -> None
```

pass criteria: 入力の _pct_ 割合が、すべての evaluator で _threshold_ 以上を満たす必要があります。

Attributes:
threshold: 個々の evaluation が到達すべき最小 score。
pct: pass すべき test-case input の割合 (0.0–1.0)。

## Eval Functions

### `pixie.run_and_evaluate`

```python
pixie.run_and_evaluate(evaluator: 'Callable[..., Any]', runnable: 'Callable[..., Any]', eval_input: 'Any', *, expected_output: 'Any' = <object object at 0x7788c2ad5c80>, from_trace: 'Callable[[list[ObservationNode]], Evaluable] | None' = None) -> 'Evaluation'
```

`_runnable(eval_input)_` を trace capture しながら実行し、その後評価します。

`_run_and_capture` と `evaluate` をまとめた convenience wrapper です。
runnable は **ちょうど 1 回** 呼ばれます。

Args:
evaluator: evaluator callable (sync/async どちらでも可)。
runnable: テスト対象の application function。
eval_input: _runnable_ に渡す単一 input。
expected_output: 任意の期待値。evaluable にマージされる。
from_trace: 評価対象 span を trace tree から選ぶ任意 callable。

Returns:
`Evaluation` result。

Raises:
ValueError: 実行中に span が 1 つも捕捉されなかった場合。

### `pixie.assert_pass`

```python
pixie.assert_pass(runnable: 'Callable[..., Any]', eval_inputs: 'list[Any]', evaluators: 'list[Callable[..., Any]]', *, evaluables: 'list[Evaluable] | None' = None, pass_criteria: 'Callable[[list[list[Evaluation]]], tuple[bool, str]] | None' = None, from_trace: 'Callable[[list[ObservationNode]], Evaluable] | None' = None) -> 'None'
```

複数 input に対して runnable を実行し、evaluator を適用します。

各 input について、`_run_and_capture` で runnable を 1 回実行し、その後 `asyncio.gather` で全 evaluator を並列評価します。

結果 matrix の shape は `[eval_inputs][evaluators]` です。
pass criteria を満たさない場合は、matrix を保持した :class:`EvalAssertionError` を送出します。

`evaluables` が与えられた場合、各 item に `eval_output` があるかで挙動が変わります。

- **eval_output is None** — `runnable` を `run_and_evaluate` で呼び、trace から output を生成する。`expected_output` は evaluable から取り込む。
- **eval_output is not None** — その evaluable を直接使う (この item では runnable は呼ばれない)。

Args:
runnable: テスト対象 application function。
eval_inputs: それぞれ _runnable_ に渡される input の list。
evaluators: evaluator callable の list。
evaluables: 任意の `Evaluable` list。各 input に 1 つ対応する。
与えた場合、その `expected_output` が `run_and_evaluate` に渡される。
長さは _eval_inputs_ と一致しなければならない。
pass_criteria: 結果 matrix を受け取り、`(passed, message)` を返す。既定は `ScoreThreshold()`。
from_trace: `run_and_evaluate` に渡す任意の span selector。

Raises:
EvalAssertionError: pass criteria を満たさない場合。
ValueError: _evaluables_ の長さが _eval_inputs_ と一致しない場合。

### `pixie.assert_dataset_pass`

```python
pixie.assert_dataset_pass(runnable: 'Callable[..., Any]', dataset_name: 'str', evaluators: 'list[Callable[..., Any]]', *, dataset_dir: 'str | None' = None, pass_criteria: 'Callable[[list[list[Evaluation]]], tuple[bool, str]] | None' = None, from_trace: 'Callable[[list[ObservationNode]], Evaluable] | None' = None) -> 'None'
```

dataset を名前で読み込み、`assert_pass` を実行します。

これは次を行う convenience wrapper です。

1. `DatasetStore` から dataset を読み込む。
2. 各 item の `eval_input` を runnable input として取り出す。
3. `expected_output` を持つ完全な `Evaluable` item を evaluable として使う。
4. `assert_pass` に委譲する。

Args:
runnable: テスト対象 application function。
dataset_name: 読み込む dataset 名。
evaluators: evaluator callable の list。
dataset_dir: dataset store directory の override。
`None` の場合は `PixieConfig.dataset_dir` から読む。
pass_criteria: 結果 matrix を受け取り、`(passed, message)` を返す。
from_trace: `assert_pass` に渡す任意 span selector。

Raises:
FileNotFoundError: _dataset_name_ の dataset が存在しない場合。
EvalAssertionError: pass criteria を満たさない場合。

## Trace Helpers

### `pixie.last_llm_call`

```python
pixie.last_llm_call(trace: 'list[ObservationNode]') -> 'Evaluable'
```

trace tree の中で `ended_at` が最も遅い `LLMSpan` を見つけます。

Args:
trace: trace tree (`ObservationNode` root list)。

Returns:
最も最近終了した `LLMSpan` を包んだ `Evaluable`。

Raises:
ValueError: trace 内に `LLMSpan` が 1 つもない場合。

### `pixie.root`

```python
pixie.root(trace: 'list[ObservationNode]') -> 'Evaluable'
```

最初の root node の span を `Evaluable` として返します。

Args:
trace: trace tree (`ObservationNode` root list)。

Returns:
最初の root node の span を包んだ `Evaluable`。

Raises:
ValueError: trace が空の場合。

### `pixie.capture_traces`

```python
pixie.capture_traces() -> 'Generator[MemoryTraceHandler, None, None]'
```

`MemoryTraceHandler` を install して yield する context manager。

`init()` を呼び出し (すでに初期化されていれば no-op)、その後 `add_handler()` で handler を登録します。終了時には handler を外し、delivery queue を flush して、すべての span が `handler.spans` から参照できるようにします。
