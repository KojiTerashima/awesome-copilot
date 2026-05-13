# Investigation and Iteration

この reference は eval-driven-dev process の Step 6 を扱います。つまり、test failure を調査し、root cause を特定し、fix の反復を行う段階です。

---

## STOP — check before proceeding

**調査や反復作業を始める前に、続けるか止まってユーザーに確認するかを決めなければなりません。**

**すぐ続行してよい** のは、ユーザーの元の prompt が明示的に iteration を求めている場合です。"fix"、"improve"、"debug"、"iterate"、"investigate failures"、"make tests pass" といった語を探してください。この場合は、以下の investigation step へ進みます。

**それ以外は、ここで STOP してください。** ユーザーには test result を次のように報告します。

> "QA setup is complete. Tests show N/M passing. [brief summary of failures if any]. Want me to investigate the failures and iterate?"

**ユーザーが確認するまで調査を進めてはいけません。** これが既定の動作です。"set up evals"、"add tests"、"set up QA"、"add evaluations" のような prompt の大半は setup だけを求めており、iteration ではありません。

---

## Step-by-step investigation

ユーザーが確認した場合 (または元の prompt が明示的に iteration を求めていた場合) は、次へ進みます。

### 1. Read the analysis

まず Step 5 で生成した analysis を読みます。analysis file は `{PIXIE_ROOT}/results/<test_id>/dataset-<index>.md` にあります。ここには test run 全体の成功・失敗パターンについて、LLM が生成した insight が入っています。どの失敗から優先的に調査すべきか、どのような systemic issue があるかを理解するために使ってください。

### 2. Get detailed test output

```bash
pixie test -v    # case ごとの score と reasoning を表示
```

完全な verbose output を取得します。各 failing case について、次を記録してください。

- `entry_kwargs` (何を送ったか)
- `the captured output` (アプリが何を生成したか)
- `expected_output` (適用される場合は何が期待されていたか)
- evaluator score と reasoning

### 3. Inspect the trace data

各 failing case について、app 内で何が起きたかを確認するため、full trace を調べます。

```python
from pixie import DatasetStore

store = DatasetStore()
ds = store.get("<dataset-name>")
for i, item in enumerate(ds.items):
    print(i, item.eval_metadata)   # trace_id is here
```

続いて full span tree を確認します。

```python
import asyncio
from pixie import ObservationStore

async def inspect(trace_id: str):
    store = ObservationStore()
    roots = await store.get_trace(trace_id)
    for root in roots:
        print(root.to_text())   # full span tree: inputs, outputs, LLM messages

asyncio.run(inspect("the-trace-id-here"))
```

### 4. Root-cause analysis

trace を追って、failure が正確にどこで始まっているかを特定します。よくあるパターンは次のとおりです。

**LLM-related failures** (prompt/model/eval の修正で対応):

| Symptom                                                | Likely cause                                                  |
| ------------------------------------------------------ | ------------------------------------------------------------- |
| Tool 結果は正しいのに出力が事実誤認している           | Prompt が tool output を忠実に使うよう LLM に指示していない   |
| Agent が誤った tool/handoff に route する             | Routing prompt または handoff description が曖昧              |
| 出力 format が誤っている                              | Prompt に format instruction が足りない                       |
| Tool を使うべきなのに LLM が hallucinate した         | Prompt が tool usage を強制していない                         |

**Non-LLM failures** (従来どおりの code 修正が必要。eval の範囲外):

| Symptom                                           | Likely cause                                            |
| ------------------------------------------------- | ------------------------------------------------------- |
| Tool が誤った data を返した                       | Tool 実装の bug — eval ではなく tool を修正する         |
| Keyword mismatch で tool が呼ばれなかった        | Tool-selection logic が壊れている — code を修正する     |
| Database が古い/誤った record を返した           | Data issue — 独立に修正する                             |
| API call が error で失敗した                      | Infrastructure issue                                    |

non-LLM failure については、investigation log に記録し、必要な code fix を推奨してください。ただし **non-LLM code の bug に合わせて eval expectation や threshold を緩めてはいけません**。eval test は、システムの残りが正しく動く前提で LLM quality を測るものです。

### 5. Document findings

**すべての failure investigation は fix と一緒に文書化する必要があります。** 次を含めてください。

````markdown
### <date> — failure investigation

**Dataset**: `qa-golden-set`
**Result**: 3/5 cases passed (60%)

#### Failing case 1: "What rows have extra legroom?"

- **entry_kwargs**: `{"user_message": "What rows have extra legroom?"}`
- **the captured output**: "I'm sorry, I don't have the exact row numbers for extra legroom..."
- **expected_output**: "rows 5-8 Economy Plus with extra legroom"
- **Evaluator score**: 0.1 (Factuality)
- **Evaluator reasoning**: "The output claims not to know the answer while the reference clearly states rows 5-8..."

**Trace analysis**:
Inspected trace `abc123`. The span tree shows:

1. Triage Agent routed to FAQ Agent ✓
2. FAQ Agent called `faq_lookup_tool("What rows have extra legroom?")` ✓
3. `faq_lookup_tool` returned "I'm sorry, I don't know..." ← **root cause**

**Root cause**: `faq_lookup_tool` (customer_service.py:112) uses keyword matching.
The seat FAQ entry is triggered by keywords `["seat", "seats", "seating", "plane"]`.
The question "What rows have extra legroom?" contains none of these keywords, so it
falls through to the default "I don't know" response.

**Classification**: Non-LLM failure — the keyword-matching tool is broken.
The LLM agent correctly routed to the FAQ agent and used the tool; the tool
itself returned wrong data.

**Fix**: Add `"row"`, `"rows"`, `"legroom"` to the seating keyword list in
`faq_lookup_tool` (customer_service.py:130). This is a traditional code fix,
not an eval/prompt change.

**Verification**: After fix, re-run:

```bash
pixie test -v      # verify
```
````

### 6. Fix and re-run

対象を絞った変更を加え、必要なら dataset も更新し、再実行します。

```bash
pixie test -v
```

fix が安定したら analysis も再実行し、パターンが変わったか確認します。

```bash
pixie analyze <new_test_id>
```

---

## The iteration cycle

1. Step 6 の analysis を読む → failure の優先順位を付ける
2. verbose で test を実行 → 具体的な failure を特定する
3. 各 failure を調査 → LLM 由来か non-LLM 由来かを分類する
4. LLM failure なら prompt、model、eval criteria を調整する
5. non-LLM failure なら code fix を推奨または適用する
6. fix により app behavior が変わったなら dataset を更新する
7. test と analysis を再実行する
8. pass するか、ユーザーが満足するまで繰り返す
