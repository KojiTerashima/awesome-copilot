# Step 4: Build the Dataset

**Why this step**: dataset は、runnable (Step 2)、evaluator (Step 3)、use case (Step 1b) を、具体的な test scenario として結び付けます。test 時には、`pixie test` が `entry_kwargs` を使って runnable を呼び、wrap registry が `eval_input` で埋まり、evaluator が実行中に捕捉された output を採点します。

---

## Understanding `entry_kwargs`, `eval_input`, and `expectation`

dataset を作る前に、これらの意味を理解してください。

- **`entry_kwargs`** = `Runnable.run()` に Pydantic model として渡される kwargs。entry-point input (user message、request body、CLI args) です。key は `run(args: T)` のために定義した Pydantic model の field と一致していなければなりません。

- **`eval_input`** = アプリ内の `wrap(purpose="input")` call に対応する `{"name": ..., "value": ...}` object の list。test 時には、これらが wrap registry により自動注入されるため、アプリ内の `wrap(purpose="input")` call は本物の外部 dependency を呼ばず、registry value を返します。

  **CRITICAL**: `eval_input` には **少なくとも 1 件の item** が必要です (`min_length=1` validation により強制)。もしアプリに `wrap(purpose="input")` call が 1 つもない場合でも、最低 1 件の `eval_input` item を入れてください。その場合は主要な entry-point argument を synthetic input として使います。

  ```json
  "eval_input": [
    { "name": "user_input", "value": "What are your business hours?" }
  ]
  ```

  各 item は `name` (str) と `value` (JSON serializable な任意値) を持つ `NamedData` object です。

- **`expectation`** (optional) = case-specific な評価参照。この scenario で正しい output がどうあるべきかを表します。reference と比較する evaluator (`Factuality`, `ClosedQA` など) で使います。reference を必要としない output-quality evaluator では不要です。

- **eval output** = アプリが実際に生成するもので、`wrap(purpose="output")` と `wrap(purpose="state")` により runtime で捕捉されます。**dataset には保存しません**。`pixie test` がアプリを走らせたときに生成されます。

`pixie_qa/reference-trace.jsonl` の **reference trace** が data shape の主たる情報源です。

- filter して `eval_input` value の正確な serialised format を確認する
- `kwargs` record を読んで `entry_kwargs` の構造を理解する
- `purpose="output"/"state"` event を読んでアプリがどんな output を出すかを理解し、意味のある `expectation` value を書く

---

## 4a. Derive evaluator assignments

`pixie_qa/02-eval-criteria.md` の eval criteria artifact は、各 criterion と use case の対応を示します。`pixie_qa/03-evaluator-mapping.md` の evaluator mapping artifact は、各 criterion と具体的 evaluator 名の対応を示します。これらを組み合わせてください。

1. **Dataset-level default evaluators**: "All" use case に適用される criterion → その evaluator 名を top-level の `"evaluators"` array に入れる
2. **Item-level evaluators**: 一部の use case にだけ適用される criterion → relevant な row の `"evaluators"` に入れ、default も使う場合は `"..."` を含める

## 4b. Inspect data shapes with `pixie format`

reference trace に `pixie format` を実行し、正確な data shape **と**実際の app output を dataset-entry 形式で確認します。

```bash
pixie format --input reference-trace.jsonl --output dataset-sample.json
```

出力は次のようになります。

```json
{
  "entry_kwargs": {
    "user_message": "What are your business hours?"
  },
  "eval_input": [
    {
      "name": "customer_profile",
      "value": { "name": "Alice", "tier": "gold" }
    },
    {
      "name": "conversation_history",
      "value": [{ "role": "user", "content": "What are your hours?" }]
    }
  ],
  "expectation": null,
  "eval_output": {
    "response": "Our business hours are Monday to Friday, 9am to 5pm..."
  }
}
```

**Important**: この template 内の `eval_output` は、実行中の app が生成した **本物の実出力** です。これを dataset entry にコピーしてはいけません。そうすると evaluator に正解そのものを与えることになり、test が自明に通ってしまいます。代わりに:

- `entry_kwargs` と `eval_input` は、data key と format の exact template として使う
- `eval_output` は、アプリが何を返すかを理解するために見る。そのうえで、各 scenario の key quality criteria を表す **簡潔な `expectation` 説明** を書く

**Example**: `eval_output.response` が `"Our business hours are Monday to Friday, 9 AM to 5 PM, and Saturday 10 AM to 2 PM."` なら、`expectation` は `"Should mention weekday hours (Mon–Fri 9am–5pm) and Saturday hours"` のように、人や LLM evaluator が比較できる短い説明にします。

## 4c. Generate dataset items

reference trace と use case を指針に、多様な entry を作成します。

- **`entry_kwargs` key** は、`Runnable.run(args: T)` で使う Pydantic model の field と一致しなければならない
- **`eval_input`** は、アプリ内の `wrap(purpose="input")` の `name` と一致する `{"name": ..., "value": ...}` object の list でなければならない
- **各 use case をカバーすること** — `pixie_qa/02-eval-criteria.md` にあるすべての use case について少なくとも 1 件、かつ意味のある多様性を持つ input を用意する

**ユーザーが prompt で dataset や data source を指定している場合** (例: research question を含む JSON file、conversation scenario file)、その file を読み、各 entry を `entry_kwargs` / `eval_input` shape に合わせて変換し、dataset に取り込んでください。指定された data を無視してはいけません。

## 4d. Build the dataset JSON file

dataset を `pixie_qa/datasets/<name>.json` に作成します。

```json
{
  "name": "qa-golden-set",
  "runnable": "pixie_qa/scripts/run_app.py:AppRunnable",
  "evaluators": ["Factuality", "pixie_qa/evaluators.py:concise_voice_style"],
  "entries": [
    {
      "entry_kwargs": {
        "user_message": "What are your business hours?"
      },
      "description": "Customer asks about business hours with gold tier account",
      "eval_input": [
        {
          "name": "customer_profile",
          "value": { "name": "Alice Johnson", "tier": "gold" }
        }
      ],
      "expectation": "Should mention Mon-Fri 9am-5pm and Sat 10am-2pm"
    },
    {
      "entry_kwargs": {
        "user_message": "I want to change something"
      },
      "description": "Ambiguous change request from basic tier customer",
      "eval_input": [
        {
          "name": "customer_profile",
          "value": { "name": "Bob Smith", "tier": "basic" }
        }
      ],
      "expectation": "Should ask for clarification",
      "evaluators": ["...", "ClosedQA"]
    },
    {
      "entry_kwargs": {
        "user_message": "I want to end this call"
      },
      "description": "User requests call end after failed verification",
      "eval_input": [
        {
          "name": "customer_profile",
          "value": { "name": "Charlie Brown", "tier": "basic" }
        }
      ],
      "expectation": "Agent should call endCall tool and end the conversation",
      "eval_metadata": {
        "expected_tool": "endCall",
        "expected_call_ended": true
      },
      "evaluators": ["...", "pixie_qa/evaluators.py:tool_call_check"]
    }
  ]
}
```

### Key fields

**Entry structure** — 各 field は entry の top-level に置きます (flat structure — nesting しない):

```
entry:
  ├── entry_kwargs    (required) — Runnable.run() 用の args
  ├── eval_input      (required) — {"name": ..., "value": ...} object の list
  ├── description     (required) — test case を表す人間向けラベル
  ├── expectation     (optional) — comparison-based evaluator 用の参照
  ├── eval_metadata   (optional) — custom evaluator 用の追加 per-entry data
  └── evaluators      (optional) — この entry 専用の evaluator 名
```

**Top-level fields:**

- **`runnable`** (required): Step 2 で作成した `Runnable` class への `filepath:ClassName` 参照 (例: `"pixie_qa/scripts/run_app.py:AppRunnable"`)。path は project root からの相対です。
- **`evaluators`** (dataset-level, optional): すべての entry に適用する default evaluator 名。ALL use case に適用される criterion の evaluator をここに置きます。

**Per-entry fields (すべて entry の top-level):**

- **`entry_kwargs`** (required): `Runnable.run(args: T)` の Pydantic model field と一致する key を持つ。アプリの entry-point input です。
- **`eval_input`** (required): `{"name": ..., "value": ...}` object の list。name はアプリ内の `wrap(purpose="input")` name と一致します。
- **`description`** (required): `pixie_qa/02-eval-criteria.md` からの use case one-liner。
- **`expectation`** (optional): reference を必要とする evaluator 用の case-specific expectation text。
- **`eval_metadata`** (optional): custom evaluator のための追加 per-entry data — expected tool name、boolean flag、threshold など。evaluator では `evaluable.eval_metadata` として参照します。
- **`evaluators`** (optional): row-level evaluator override。

### Evaluator assignment rules

1. 全 item に適用する evaluator は top-level の `"evaluators"` array に入れる
2. **追加** evaluator が必要な item では、`"evaluators": ["...", "ExtraEval"]` を使う。`"..."` は default を展開する
3. **完全に別の** evaluator set が必要な item では、`"evaluators": ["OnlyThis"]` のように `"..."` を入れない
4. default だけでよい item では `"evaluators"` field を省略する

---

## Dataset Creation Reference

### Using `eval_input` values

`eval_input` の value は `{"name": ..., "value": ...}` object です。reference trace を template として使い、relevant な `purpose="input"` event の `"data"` field をコピーして値を調整してください。

**Simple dict**:

```json
{ "name": "customer_profile", "value": { "name": "Alice", "tier": "gold" } }
```

**List of dicts** (例: conversation history):

```json
{
  "name": "conversation_history",
  "value": [
    { "role": "user", "content": "Hello" },
    { "role": "assistant", "content": "Hi there!" }
  ]
}
```

**Important**: exact な format は `wrap(purpose="input")` が何を捕捉しているかに依存します。ゼロから組み立てるのではなく、必ず reference trace を土台にしてください。

### Crafting diverse eval scenarios

各 use case のさまざまな側面をカバーします。

- 同じ依頼に対する異なる user phrasing
- edge case (曖昧な input、不足情報、error condition)
- 特定の eval criterion を stress-test する entry
- Step 1b の各 use case につき少なくとも 1 件

---

## Output

`pixie_qa/datasets/<name>.json` — dataset file.
