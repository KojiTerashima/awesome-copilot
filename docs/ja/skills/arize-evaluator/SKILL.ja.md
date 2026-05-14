---
name: arize-evaluator
description: "Arize での LLM-as-judge 評価ワークフロー（evaluator の作成/更新、span や experiment に対する評価実行、task、trigger-run、column mapping、継続的モニタリング）では、このスキルを呼び出してください。ユーザーが次のように言ったときに使います: evaluator を作成したい、LLM judge、hallucination/faithfulness/correctness/relevance、eval を実行、span や experiment を採点、ax tasks、trigger-run、trigger eval、column mapping、continuous monitoring、eval 用 query filter、evaluator version、evaluator prompt を改善したい。"
---

# Arize Evaluator Skill

このスキルは、Arize 上で **LLM-as-judge evaluator** を設計・作成・実行するためのものです。evaluator は判定者の定義であり、**task** はそれを実データに対して実行する方法です。

---

## Prerequisites

そのままタスクを進めて、必要な `ax` コマンドを実行してください。最初にバージョン、環境変数、プロファイル確認は **しないで** ください。

`ax` コマンドが失敗した場合は、エラーに応じて切り分けます:
- `command not found` またはバージョンエラー → references/ax-setup.md を参照
- `401 Unauthorized` / API キー不足 → `ax profiles show` で現在のプロファイルを確認。プロファイルがない、または API キーが誤っている場合: `.env` の `ARIZE_API_KEY` を確認し、references/ax-profiles.md を使ってプロファイルを作成/更新。`.env` にもキーがない場合は、ユーザーに Arize API key を問い合わせる（https://app.arize.com/admin > API Keys）
- Space ID が不明 → `.env` の `ARIZE_SPACE_ID` を確認、または `ax spaces list -o json` を実行、またはユーザーに確認
- LLM provider 呼び出し失敗（OPENAI_API_KEY / ANTHROPIC_API_KEY 不足）→ `.env` を確認し、あれば読み込み、なければユーザーに確認

---

## Concepts

### Evaluator とは？

**evaluator** は LLM-as-judge の定義です。以下を含みます:

| Field | Description |
|-------|-------------|
| **Template** | 判定プロンプト。`{variable}` プレースホルダー（例: `{input}`, `{output}`, `{context}`）を使い、実行時に task の column mappings で値が埋められます。 |
| **Classification choices** | 出力ラベルの許可集合（例: `factual` / `hallucinated`）。デフォルトかつ最も一般的なのは二値です。各 choice は任意で数値スコアを持てます。 |
| **AI Integration** | evaluator が判定モデル呼び出しに使う、保存済み LLM provider 認証情報（OpenAI、Anthropic、Bedrock など）。 |
| **Model** | 使用する判定モデル（例: `gpt-4o`, `claude-sonnet-4-5`）。 |
| **Invocation params** | `{"temperature": 0}` のようなモデル設定 JSON（任意）。再現性のため低 temperature 推奨。 |
| **Optimization direction** | 高スコアが良い（`maximize`）か悪い（`minimize`）か。UI のトレンド表示方法に影響します。 |
| **Data granularity** | evaluator を **span** / **trace** / **session** のどの粒度で実行するか。多くは span レベルです。 |

evaluator は **versioned** です。プロンプトやモデルを変更するたびに新しい immutable version が作られます。最新 version が active です。

### Task とは？

**task** は、1 つ以上の evaluator を実データに対して実行する方法です。task は **project**（ライブ traces/spans）または **dataset**（experiment runs）に紐づきます。task には以下が含まれます:

| Field | Description |
|-------|-------------|
| **Evaluators** | 実行する evaluator の一覧。1 つの task で複数実行可能です。 |
| **Column mappings** | evaluator の template 変数を、span や experiment run 上の実フィールドパスに対応付けます（例: `"input" → "attributes.input.value"`）。これにより evaluator を project/experiment 間で再利用できます。 |
| **Query filter** | 評価対象 spans/runs を選ぶ SQL 風式（例: `"span_kind = 'LLM'"`）。任意ですが精度面で重要です。 |
| **Continuous** | project task で、新しい span 到着時に自動採点するか。 |
| **Sampling rate** | continuous project task で、新規 span のうち評価する割合（0–1）。 |

---

## Data Granularity

`--data-granularity` フラグは、evaluator が採点するデータ単位を制御します。デフォルトは `span` で、**project tasks** にのみ適用されます（dataset/experiment tasks には適用されず、そちらは experiment runs を直接評価します）。

| Level | What it evaluates | Use for | Result column prefix |
|-------|-------------------|---------|---------------------|
| `span` (default) | Individual spans | Q&A correctness, hallucination, relevance | `eval.{name}.label` / `.score` / `.explanation` |
| `trace` | All spans in a trace, grouped by `context.trace_id` | Agent trajectory, task correctness — anything that needs the full call chain | `trace_eval.{name}.label` / `.score` / `.explanation` |
| `session` | All traces in a session, grouped by `attributes.session.id` and ordered by start time | Multi-turn coherence, overall tone, conversation quality | `session_eval.{name}.label` / `.score` / `.explanation` |

### trace / session 集約の仕組み

**trace** 粒度では、同じ `context.trace_id` を共有する spans がグルーピングされます。evaluator template で使うカラム値は、判定モデルに渡す前にカンマ結合された 1 つの文字列にされます（各値は 100K 文字で切り詰め）。

**session** 粒度では、まず同様の trace 単位グルーピングを行い、その後 `start_time` 順に traces を並べ、`attributes.session.id` ごとにグルーピングします。session レベル値は合計 100K 文字までです。

### `{conversation}` template variable

session 粒度では、`{conversation}` は特別な template variable で、session 内すべての traces にまたがる `{input, output}` ターンの JSON 配列として展開されます。入力側は `attributes.input.value` / `attributes.llm.input_messages`、出力側は `attributes.output.value` / `attributes.llm.output_messages` から構築されます。

span または trace 粒度では、`{conversation}` は通常の template variable として扱われ、他と同様に column mappings で解決されます。

### Multi-evaluator tasks

1 つの task に異なる粒度の evaluator を含められます。実行時にはシステムが **最も高い** 粒度（session > trace > span）でデータ取得し、**evaluator ごとに 1 つの child run に自動分割** します。task の evaluators JSON にある evaluator 単位の `query_filter` で、含める spans をさらに絞り込めます（例: session 内で tool-call spans のみ）。

---

## Basic CRUD

### AI Integrations

AI integrations は、evaluator が使う LLM provider 認証情報を保持します。全 CRUD（一覧、全 provider 向け作成（OpenAI、Anthropic、Azure、Bedrock、Vertex、Gemini、NVIDIA NIM、custom）、更新、削除）は **arize-ai-provider-integration** スキルを使ってください。

一般的ケース（OpenAI）のクイックリファレンス:

```bash
# まず既存 integration を確認
ax ai-integrations list --space-id SPACE_ID

# なければ作成
ax ai-integrations create \
  --name "My OpenAI Integration" \
  --provider openAI \
  --api-key $OPENAI_API_KEY
```

返却された integration ID を控えてください。`ax evaluators create --ai-integration-id` で必須です。

### Evaluators

```bash
# List / Get
ax evaluators list --space-id SPACE_ID
ax evaluators get EVALUATOR_ID
ax evaluators list-versions EVALUATOR_ID
ax evaluators get-version VERSION_ID

# Create (evaluator とその初回 version を作成)
ax evaluators create \
  --name "Answer Correctness" \
  --space-id SPACE_ID \
  --description "Judges if the model answer is correct" \
  --template-name "correctness" \
  --commit-message "Initial version" \
  --ai-integration-id INT_ID \
  --model-name "gpt-4o" \
  --include-explanations \
  --use-function-calling \
  --classification-choices '{"correct": 1, "incorrect": 0}' \
  --template 'You are an evaluator. Given the user question and the model response, decide if the response correctly answers the question.

User question: {input}

Model response: {output}

Respond with exactly one of these labels: correct, incorrect'

# 新しい version を作成（prompt/model 変更用 — version は immutable）
ax evaluators create-version EVALUATOR_ID \
  --commit-message "Added context grounding" \
  --template-name "correctness" \
  --ai-integration-id INT_ID \
  --model-name "gpt-4o" \
  --include-explanations \
  --classification-choices '{"correct": 1, "incorrect": 0}' \
  --template 'Updated prompt...

{input} / {output} / {context}'

# メタデータのみ更新（name, description — prompt は不可）
ax evaluators update EVALUATOR_ID \
  --name "New Name" \
  --description "Updated description"

# Delete（恒久的 — 全 version を削除）
ax evaluators delete EVALUATOR_ID
```

**`create` の主要フラグ:**

| Flag | Required | Description |
|------|----------|-------------|
| `--name` | yes | evaluator 名（space 内で一意） |
| `--space-id` | yes | 作成先 Space |
| `--template-name` | yes | eval カラム名（英数字、スペース、ハイフン、アンダースコア） |
| `--commit-message` | yes | この version の説明 |
| `--ai-integration-id` | yes | AI integration ID（上記参照） |
| `--model-name` | yes | 判定モデル（例: `gpt-4o`） |
| `--template` | yes | `{variable}` プレースホルダー付き prompt（bash では単引用符） |
| `--classification-choices` | yes | choice ラベル→数値スコアの JSON（例: `'{"correct": 1, "incorrect": 0}'`） |
| `--description` | no | 人間向け説明 |
| `--include-explanations` | no | ラベルと一緒に理由を含める |
| `--use-function-calling` | no | 構造化された function-call 出力を優先 |
| `--invocation-params` | no | モデルパラメータ JSON（例: `'{"temperature": 0}'`） |
| `--data-granularity` | no | `span`（デフォルト）, `trace`, `session`。project tasks のみ関連し、dataset/experiment tasks には非該当。Data Granularity セクション参照。 |
| `--provider-params` | no | provider 固有パラメータの JSON object |

### Tasks

```bash
# List / Get
ax tasks list --space-id SPACE_ID
ax tasks list --project-id PROJ_ID
ax tasks list --dataset-id DATASET_ID
ax tasks get TASK_ID

# Create (project — continuous)
ax tasks create \
  --name "Correctness Monitor" \
  --task-type template_evaluation \
  --project-id PROJ_ID \
  --evaluators '[{"evaluator_id": "EVAL_ID", "column_mappings": {"input": "attributes.input.value", "output": "attributes.output.value"}}]' \
  --is-continuous \
  --sampling-rate 0.1

# Create (project — one-time / backfill)
ax tasks create \
  --name "Correctness Backfill" \
  --task-type template_evaluation \
  --project-id PROJ_ID \
  --evaluators '[{"evaluator_id": "EVAL_ID", "column_mappings": {"input": "attributes.input.value", "output": "attributes.output.value"}}]' \
  --no-continuous

# Create (experiment / dataset)
ax tasks create \
  --name "Experiment Scoring" \
  --task-type template_evaluation \
  --dataset-id DATASET_ID \
  --experiment-ids "EXP_ID_1,EXP_ID_2" \
  --evaluators '[{"evaluator_id": "EVAL_ID", "column_mappings": {"output": "output"}}]' \
  --no-continuous

# Trigger a run (project task — data window を使用)
ax tasks trigger-run TASK_ID \
  --data-start-time "2026-03-20T00:00:00" \
  --data-end-time "2026-03-21T23:59:59" \
  --wait

# Trigger a run (experiment task — experiment IDs を使用)
ax tasks trigger-run TASK_ID \
  --experiment-ids "EXP_ID_1" \
  --wait

# Monitor
ax tasks list-runs TASK_ID
ax tasks get-run RUN_ID
ax tasks wait-for-run RUN_ID --timeout 300
ax tasks cancel-run RUN_ID --force
```

**trigger-run の時刻形式:** `2026-03-21T09:00:00` — 末尾の `Z` なし。

**追加の trigger-run フラグ:**

| Flag | Description |
|------|-------------|
| `--max-spans` | 処理 span 数の上限（デフォルト 10,000） |
| `--override-evaluations` | 既にラベル済み span も再採点 |
| `--wait` / `-w` | run 完了まで待機 |
| `--timeout` | `--wait` 時の待機秒数（デフォルト 600） |
| `--poll-interval` | 待機時のポーリング間隔（秒、デフォルト 5） |

**Run ステータスガイド:**

| Status | Meaning |
|--------|---------|
| `completed`, 0 spans | その期間の eval index に span がない — 時間範囲を広げる |
| `cancelled` ~1s | integration 認証情報が不正 |
| `cancelled` ~3min | spans は見つかったが LLM 呼び出し失敗 — model 名または key を確認 |
| `completed`, N > 0 | 成功 — UI でスコア確認 |

---

## Workflow A: project 用 evaluator を作成する

ユーザーが *"create an evaluator for my Playground Traces project"* のように言った場合に使います。

### Step 1: project 名を ID に解決

`ax spans export` が必要なのは project **ID** であり、名前ではありません。名前を渡すと validation error になります。必ず先に ID を引きます:

```bash
ax projects list --space-id SPACE_ID -o json
```

`"name"` が一致（大文字小文字無視）するエントリを見つけ、その `"id"`（base64 文字列）を使います。

### Step 2: 何を評価するかを把握

ユーザーが evaluator 種別（hallucination, correctness, relevance など）を指定済みなら Step 3 へ。

未指定なら、直近 spans をサンプルして実データベースで設計します:

```bash
ax spans export PROJECT_ID --space-id SPACE_ID -l 10 --days 30 --stdout
```

`attributes.input`, `attributes.output`, span kind、既存 annotation を確認。失敗モード（例: hallucinated facts、off-topic answers、missing context）を特定し、**具体的な evaluator 案を 1〜3 個** 提案してユーザーに選んでもらいます。

各提案には、evaluator 名（太字）、何を判定するか 1 文説明、二値ラベル対（括弧内）を含め、次形式で提示:

1. **Name** — Description of what is being judged. (`label_a` / `label_b`)

例:
1. **Response Correctness** — Does the agent's response correctly address the user's financial query? (`correct` / `incorrect`)
2. **Hallucination** — Does the response fabricate facts not grounded in retrieved context? (`factual` / `hallucinated`)

### Step 3: AI integration を確認または作成

```bash
ax ai-integrations list --space-id SPACE_ID -o json
```

適切な integration があれば ID を控えます。なければ **arize-ai-provider-integration** スキルで作成。判定に使う provider/model はユーザーに確認します。

### Step 4: evaluator を作成

以下の template 設計ベストプラクティスに従います。evaluator 名と変数は **汎用的** に保ち、project 固有の接続は task（Step 6）の `column_mappings` で行います。

```bash
ax evaluators create \
  --name "Hallucination" \
  --space-id SPACE_ID \
  --template-name "hallucination" \
  --commit-message "Initial version" \
  --ai-integration-id INT_ID \
  --model-name "gpt-4o" \
  --include-explanations \
  --use-function-calling \
  --classification-choices '{"factual": 1, "hallucinated": 0}' \
  --template 'You are an evaluator. Given the user question and the model response, decide if the response is factual or contains unsupported claims.

User question: {input}

Model response: {output}

Respond with exactly one of these labels: hallucinated, factual'
```

### Step 5: 確認 — backfill、continuous、または両方？

task 作成前に次を確認:

> "Would you like to:
> (a) Run a **backfill** on historical spans (one-time)?
> (b) Set up **continuous** evaluation on new spans going forward?
> (c) **Both** — backfill now and keep scoring new spans automatically?"

### Step 6: 実 span データから column mappings を決定

パスを推測しないでください。サンプルを取得し、実際に存在するフィールドを確認します:

```bash
ax spans export PROJECT_ID --space-id SPACE_ID -l 5 --days 7 --stdout
```

各 template 変数（`{input}`, `{output}`, `{context}`）に対応する JSON path を見つけます。よくある起点（**必ず実データで検証してから使用**）:

| Template var | LLM span | CHAIN span |
|---|---|---|
| `input` | `attributes.input.value` | `attributes.input.value` |
| `output` | `attributes.llm.output_messages.0.message.content` | `attributes.output.value` |
| `context` | `attributes.retrieval.documents.contents` | — |
| `tool_output` | `attributes.input.value` (fallback) | `attributes.output.value` |

**span kind の整合性を検証:** evaluator prompt が LLM 最終テキストを前提なのに task が CHAIN spans を対象（またはその逆）だと、run が cancel したり誤ったテキストを採点します。task の `query_filter` が、マッピングした span kind と一致するようにしてください。

**`--evaluators` JSON の完全例:**

```json
[
  {
    "evaluator_id": "EVAL_ID",
    "query_filter": "span_kind = 'LLM'",
    "column_mappings": {
      "input": "attributes.input.value",
      "output": "attributes.llm.output_messages.0.message.content",
      "context": "attributes.retrieval.documents.contents"
    }
  }
]
```

template で参照する **すべて** の変数に mapping を入れてください。1 つでも欠けると run が有効スコアを出せません。

### Step 7: task を作成

**backfill のみ (a):**
```bash
ax tasks create \
  --name "Hallucination Backfill" \
  --task-type template_evaluation \
  --project-id PROJECT_ID \
  --evaluators '[{"evaluator_id": "EVAL_ID", "column_mappings": {"input": "attributes.input.value", "output": "attributes.output.value"}}]' \
  --no-continuous
```

**continuous のみ (b):**
```bash
ax tasks create \
  --name "Hallucination Monitor" \
  --task-type template_evaluation \
  --project-id PROJECT_ID \
  --evaluators '[{"evaluator_id": "EVAL_ID", "column_mappings": {"input": "attributes.input.value", "output": "attributes.output.value"}}]' \
  --is-continuous \
  --sampling-rate 0.1
```

**両方 (c):** 作成時に `--is-continuous` を使い、さらに Step 8 で backfill run を起動します。

### Step 8: backfill run を起動（要望がある場合）

まずデータがある時間範囲を確認:
```bash
ax spans export PROJECT_ID --space-id SPACE_ID -l 100 --days 1 --stdout   # まず直近24時間
ax spans export PROJECT_ID --space-id SPACE_ID -l 100 --days 7 --stdout   # 空なら拡大
```

実 span の `start_time` / `end_time` を使ってウィンドウを設定します。初回テスト run は最新データを使います。

```bash
ax tasks trigger-run TASK_ID \
  --data-start-time "2026-03-20T00:00:00" \
  --data-end-time "2026-03-21T23:59:59" \
  --wait
```

---

## Workflow B: experiment 用 evaluator を作成する

ユーザーが *"create an evaluator for my experiment"* または *"evaluate my dataset runs"* のように言った場合に使います。

**ユーザーが「dataset」と言うが experiment がない場合:** task は experiment（dataset 単体では不可）を対象にする必要があります。次を確認:
> "Evaluation tasks run against experiment runs, not datasets directly. Would you like help creating an experiment on that dataset first?"

yes の場合は **arize-experiment** スキルで作成後、ここに戻ります。

### Step 1: dataset と experiment を解決

```bash
ax datasets list --space-id SPACE_ID -o json
ax experiments list --dataset-id DATASET_ID -o json
```

採点対象の dataset ID と experiment ID（複数可）を控えます。

### Step 2: 何を評価するかを把握

ユーザーが evaluator 種別を指定済みなら Step 3 へ。

未指定なら、直近 experiment run を確認して実データベースで設計します:

```bash
ax experiments export EXPERIMENT_ID --stdout | python3 -c "import sys,json; runs=json.load(sys.stdin); print(json.dumps(runs[0], indent=2))"
```

`output`, `input`, `evaluations`, `metadata` を確認。欠けている指標（ユーザーが重視していて未計測）を特定し、**evaluator 案を 1〜3 個** 提案。各提案は Workflow A Step 2 と同形式（太字名、1 文説明、二値ラベル対）で提示。

### Step 3: AI integration を確認または作成

Workflow A Step 3 と同じです。

### Step 4: evaluator を作成

Workflow A Step 4 と同じです。変数は汎用的に保ちます。

### Step 5: 実 run データから column mappings を決定

run データ形状は span データと異なります。確認:

```bash
ax experiments export EXPERIMENT_ID --stdout | python3 -c "import sys,json; runs=json.load(sys.stdin); print(json.dumps(runs[0], indent=2))"
```

experiment runs でよくある mapping:
- `output` → `"output"`（各 run のトップレベル）
- `input` → run 上にあるか、紐づく dataset examples に埋め込まれているかを確認

`input` が run JSON にない場合、dataset examples を export してパスを確認:
```bash
ax datasets export DATASET_ID --stdout | python3 -c "import sys,json; ex=json.load(sys.stdin); print(json.dumps(ex[0], indent=2))"
```

### Step 6: task を作成

```bash
ax tasks create \
  --name "Experiment Correctness" \
  --task-type template_evaluation \
  --dataset-id DATASET_ID \
  --experiment-ids "EXP_ID" \
  --evaluators '[{"evaluator_id": "EVAL_ID", "column_mappings": {"output": "output"}}]' \
  --no-continuous
```

### Step 7: 起動と監視

```bash
ax tasks trigger-run TASK_ID \
  --experiment-ids "EXP_ID" \
  --wait

ax tasks list-runs TASK_ID
ax tasks get-run RUN_ID
```

---

## Template Design のベストプラクティス

### 1. 汎用で再利用可能な変数名を使う

`{input}`, `{output}`, `{context}` を使い、特定 project/span 属性に依存する名前は避けます（例: `{attributes_input_value}` は使わない）。evaluator 自体は抽象のまま保ち、特定 project/experiment の実フィールドへの接続は **task の `column_mappings`** で行います。これにより同一 evaluator を複数 project/experiment で修正なしに再利用できます。

### 2. デフォルトは二値ラベル

明確な文字列ラベルをちょうど 2 つ使います（例: `hallucinated` / `factual`, `correct` / `incorrect`, `pass` / `fail`）。二値ラベルは:
- 判定モデルが一貫して出しやすい
- 業界で最も一般的
- ダッシュボードで解釈しやすい

ユーザーが 3 値以上を希望するなら対応可ですが、まずは二値を推奨し、トレードオフ（ラベル増加 → 曖昧性増加 → 評価者間一致の低下）を説明します。

### 3. モデルの返答形式を明示する

template では、判定モデルに **ラベル文字列のみ** を返すよう明示してください。prompt 中のラベル文字列は `--classification-choices` のラベルと **完全一致**（スペル・大文字小文字）する必要があります。

Good:
```
Respond with exactly one of these labels: hallucinated, factual
```

Bad（曖昧すぎる）:
```
Is this hallucinated? Answer yes or no.
```

### 4. temperature は低く保つ

再現性ある採点のため `--invocation-params '{"temperature": 0}'` を渡します。temperature が高いと評価結果にノイズが入ります。

### 5. デバッグには `--include-explanations` を使う

初期設定時は常に explanations を含め、スケール運用前に判定ロジックが妥当か確認してください。

### 6. bash では template を単引用符で渡す

単引用符により shell が `{variable}` を展開しません。二重引用符だと問題が起きます:

```bash
# Correct
--template 'Judge this: {input} → {output}'

# Wrong — shell may interpret { } or fail
--template "Judge this: {input} → {output}"
```

### 7. `--classification-choices` は template ラベルに必ず一致させる

`--classification-choices` のラベルは `--template` で参照するラベルと完全一致（スペル・大文字小文字）である必要があります。`--classification-choices` を省略すると task run は "missing rails and classification choices." で失敗します。

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| `ax: command not found` | references/ax-setup.md を参照 |
| `401 Unauthorized` | API key がこの space にアクセス権を持たない可能性。https://app.arize.com/admin > API Keys で確認 |
| `Evaluator not found` | `ax evaluators list --space-id SPACE_ID` |
| `Integration not found` | `ax ai-integrations list --space-id SPACE_ID` |
| `Task not found` | `ax tasks list --space-id SPACE_ID` |
| `project-id and dataset-id are mutually exclusive` | task 作成時はどちらか一方のみ使用 |
| `experiment-ids required for dataset tasks` | `create` と `trigger-run` に `--experiment-ids` を追加 |
| `sampling-rate only valid for project tasks` | dataset tasks では `--sampling-rate` を外す |
| `ax spans export` の validation error | project 名ではなく project ID（base64）を渡す。`ax projects list` で取得 |
| Template validation errors | bash で `--template '...'` を単引用符で指定。`{{var}}` ではなく `{var}` を使用 |
| Run stuck in `pending` | `ax tasks get-run RUN_ID`; その後 `ax tasks cancel-run RUN_ID` |
| Run `cancelled` ~1s | integration 認証情報が無効 — AI integration を確認 |
| Run `cancelled` ~3min | spans は見つかったが LLM 呼び出し失敗 — model 名または key が誤り |
| Run `completed`, 0 spans | 時間範囲を拡大。eval index が古いデータをカバーしていない可能性 |
| UI にスコアが出ない | spans/runs の実パスに合うよう `column_mappings` を修正 |
| スコアが不自然 | `--include-explanations` を追加し、数サンプルで判定理由を確認 |
| Evaluator が誤った span kind で cancel | `query_filter` と `column_mappings` を LLM/CHAIN spans に合わせる |
| `trigger-run` の時刻形式エラー | `2026-03-21T09:00:00` を使用 — 末尾 `Z` なし |
| Run failed: "missing rails and classification choices" | `ax evaluators create` に `--classification-choices '{"label_a": 1, "label_b": 0}'` を追加 — ラベルは template と一致必須 |
| Run `completed`, all spans skipped | query filter は一致しているが column mappings 不正、または template 変数が解決できていない — sample span を export してパス確認 |

---

## Related Skills

- **arize-ai-provider-integration**: LLM provider integrations の完全 CRUD（認証情報の作成/更新/削除）
- **arize-trace**: spans を export して column path と time range を特定
- **arize-experiment**: experiment を作成し、run を export して experiment の column mappings を決定
- **arize-dataset**: runs に input がない場合、dataset examples を export して input フィールドを特定
- **arize-link**: Arize UI の evaluator/task へのディープリンク

---

## Save Credentials for Future Use

references/ax-profiles.md の「§ Save Credentials for Future Use」を参照。

