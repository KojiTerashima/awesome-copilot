# Step 5: Run Evaluation-Based Tests

**Why this step**: `pixie test` を実行し、すべての entry で実 evaluator score が出るまで dataset quality の問題 — `WrapRegistryMissError`、`WrapTypeMismatchError`、不正な `eval_input` data、import failure など — を修正します。

---

## 5a. Run tests

```bash
pixie test
```

各ケースの score と evaluator reasoning を含む verbose 出力が必要なら:

```bash
pixie test -v
```

`pixie test` は test 実行前に `.env` file を自動読み込みします。

現在の test runner は次のように動作します。

1. dataset の `runnable` field から `Runnable` class を解決する
2. `Runnable.create()` を呼んで instance を作り、`setup()` を 1 回呼ぶ
3. すべての dataset entry を **並列実行** する (最大 4 並列):
   a. entry から `entry_kwargs` と `eval_input` を読む
   b. wrap input registry を `eval_input` data で埋める
   c. capture registry を初期化する
   d. `entry_kwargs` を Pydantic model へ検証して `Runnable.run(args)` を呼ぶ
   e. アプリ内の `wrap(purpose="input")` は外部 service を呼ばず、registry value を返す
   f. `wrap(purpose="output"/"state")` は評価用 data を捕捉する
   g. 捕捉 data から `Evaluable` を構築する
   h. evaluator を実行する
4. `Runnable.teardown()` を 1 回呼ぶ

entry は並列実行されるため、Runnable の `run()` method は concurrency-safe でなければなりません。`sqlite3.OperationalError`、`"database is locked"` などの error が出た場合は、Runnable に `Semaphore(1)` を追加してください (Step 2 reference の concurrency section を参照)。

## 5b. Fix dataset/harness issues

**Data validation error** (registry miss、type mismatch、deserialization failure) は、対象の `wrap` name と dataset field を示す明確な message とともに entry 単位で報告されます。このステップは **Step 4 で自分が誤って作ったもの** を直すフェーズです。つまり bad data、wrong format、missing field の修正であり、アプリ品質そのものを評価する段階ではありません。

| Error                                 | Cause                                                                                                                   | Fix                                                                                          |
| ------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| `WrapRegistryMissError: name='<key>'` | アプリ内の `wrap(purpose="input", name="<key>")` が期待する `name` を持つ `eval_input` item が dataset entry にない | 影響するすべての entry の `eval_input` に `{"name": "<key>", "value": ...}` を追加する |
| `WrapTypeMismatchError`               | Deserialize した型がアプリ期待と一致しない                                                                             | dataset 内の value を修正する                                                                |
| Runnable resolution failure           | `runnable` path/class name が誤っている、または class が `Runnable` protocol を実装していない                         | dataset の `filepath:ClassName` を修正し、class に `create()` と `run()` を実装する         |
| Import error                          | runnable/evaluator に module path または syntax error がある                                                           | 参照先 file を修正する                                                                       |
| `ModuleNotFoundError: pixie_qa`       | `pixie_qa/` directory に `__init__.py` がない                                                                          | `pixie init` を実行して再作成する                                                            |
| `TypeError: ... is not callable`      | evaluator 名が non-callable attribute を指している                                                                     | evaluator は function、class、または callable instance でなければならない                    |
| `sqlite3.OperationalError`            | 並列な `run()` 呼び出しが同じ SQLite connection を共有している                                                         | Runnable に `asyncio.Semaphore(1)` を追加する (Step 2 の concurrency section 参照)          |

反復してください。error を直し、再実行し、次の error を直す。この繰り返しで、すべての entry に対して `pixie test` が実 evaluator score を出すまで進めます。

### When to stop iterating on evaluator results

dataset が error なく実行され、実 score が出たら結果を評価します。

- **Custom function evaluator** (決定的な check): 失敗した場合、問題は dataset data か evaluator logic にあります。修正して再実行してください。これらは通常すぐ収束します。
- **LLM-as-judge evaluator** (`Factuality`、`ClosedQA`、custom LLM evaluator など): 実行ごとに揺らぎがあります。コード変更なしで score がぶれるなら、問題は app behavior ではなく evaluator prompt quality です。**LLM evaluator prompt に 1 サイクル以上の修正時間を使ってはいけません。** 2〜3 回実行して variance を見て、方向性として妥当なら受け入れてください。
- **General rule**: すべての custom function evaluator が安定して通り、LLM evaluator が概ね妥当な score (大半が pass) を返す時点で反復を止めます。LLM evaluator の完全スコアが目標ではありません。目標は、実 regression を検出できる動く QA pipeline です。

## 5c. Run analysis

setup error なしで test が完了し、実 score が出たら analysis を実行します。

```bash
pixie analyze <test_id>
```

`<test_id>` は `pixie test` が出力する test run identifier です (例: `20250615-120000`)。これにより dataset ごとの LLM-powered な markdown analysis が生成され、成功・失敗パターンが分かります。

## Output

- Test result: `{PIXIE_ROOT}/results/<test_id>/result.json`
- Analysis file: `{PIXIE_ROOT}/results/<test_id>/dataset-<index>.md` (`pixie analyze` 後)

---

> **If you hit an unexpected error** when running tests (parameter 名の誤り、import failure、API mismatch など) は、勘で直す前に `wrap-api.md`、`evaluators.md`、`testing-api.md` の authoritative reference を読んでください。
