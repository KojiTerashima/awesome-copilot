---
name: arize-prompt-optimization
description: "本番トレースデータ、評価、アノテーションを使ってLLMプロンプトを最適化・改善・デバッグする場合はこのスキルを呼び出してください。spanからのプロンプト抽出、パフォーマンスシグナルの収集、ax CLIを使ったデータ駆動の最適化ループをカバーします。"
---

# Arize Prompt Optimization Skill

## 概念

### トレースデータ内でプロンプトが存在する場所

LLMアプリケーションは、OpenInferenceのセマンティック規約に従ってspanを出力します。プロンプトは、spanの種類や計測方法によって異なるspan属性に保存されます。

| Column | 含まれる内容 | 使用するタイミング |
|--------|-----------------|-------------|
| `attributes.llm.input_messages` | 構造化されたチャットメッセージ（system, user, assistant, tool）のロールベース形式 | チャットベースLLMプロンプトの**主要ソース** |
| `attributes.llm.input_messages.roles` | ロール配列: `system`, `user`, `assistant`, `tool` | 各メッセージロールを抽出する |
| `attributes.llm.input_messages.contents` | メッセージ本文文字列の配列 | メッセージテキストを抽出する |
| `attributes.input.value` | シリアライズされたプロンプトまたはユーザー質問（汎用、全span種別） | 構造化メッセージが利用できないときのフォールバック |
| `attributes.llm.prompt_template.template` | `{variable}` プレースホルダー付きテンプレート（例: `"Answer {question} using {context}"`） | アプリがプロンプトテンプレートを使う場合 |
| `attributes.llm.prompt_template.variables` | テンプレート変数の値（JSONオブジェクト） | テンプレートにどの値が代入されたか確認する |
| `attributes.output.value` | モデルの応答テキスト | LLMが何を生成したか確認する |
| `attributes.llm.output_messages` | 構造化されたモデル出力（ツール呼び出しを含む） | ツール呼び出し応答を調査する |

### span種別でプロンプトを見つける

- **LLM span** (`attributes.openinference.span.kind = 'LLM'`): 構造化チャットメッセージは `attributes.llm.input_messages` を確認、またはシリアライズ済みプロンプトは `attributes.input.value` を確認します。テンプレートは `attributes.llm.prompt_template.template` を確認します。
- **Chain/Agent span**: `attributes.input.value` にはユーザー質問が入ります。実際のLLMプロンプトは**子のLLM spans**にあります -- トレースツリーを下にたどってください。
- **Tool span**: `attributes.input.value` にツール入力、`attributes.output.value` にツール結果が入ります。通常、プロンプトがある場所ではありません。

### パフォーマンスシグナル列

これらの列には、最適化に使うフィードバックデータが入ります。

| Column pattern | Source | わかること |
|---------------|--------|-------------------|
| `annotation.<name>.label` | 人間レビューアー | カテゴリ評価（例: `correct`, `incorrect`, `partial`） |
| `annotation.<name>.score` | 人間レビューアー | 数値品質スコア（例: 0.0 - 1.0） |
| `annotation.<name>.text` | 人間レビューアー | 評価理由の自由記述 |
| `eval.<name>.label` | LLM-as-judge evals | 自動カテゴリ評価 |
| `eval.<name>.score` | LLM-as-judge evals | 自動数値スコア |
| `eval.<name>.explanation` | LLM-as-judge evals | そのスコアになった理由 -- **最適化で最も重要** |
| `attributes.input.value` | トレースデータ | LLMへの入力内容 |
| `attributes.output.value` | トレースデータ | LLMの出力内容 |
| `{experiment_name}.output` | Experiment runs | 特定実験の出力 |

## 前提条件

タスクにそのまま着手し、必要な `ax` コマンドを実行してください。事前にバージョン、環境変数、プロファイルを確認しないでください。

`ax` コマンドが失敗した場合は、エラーに基づいて対処します。
- `command not found` またはバージョンエラー → references/ax-setup.md を参照
- `401 Unauthorized` / APIキー未設定 → `ax profiles show` で現在のプロファイルを確認。プロファイル未作成またはAPIキー誤りの場合: `.env` の `ARIZE_API_KEY` を確認し、references/ax-profiles.md に従ってプロファイルを作成/更新。`.env` にもキーがない場合は、ユーザーにArize APIキーを依頼（https://app.arize.com/admin > API Keys）
- Space ID不明 → `.env` の `ARIZE_SPACE_ID` を確認、または `ax spaces list -o json` を実行、またはユーザーに確認
- Projectが不明確 → `.env` の `ARIZE_DEFAULT_PROJECT` を確認、またはユーザーに確認、または `ax projects list -o json --limit 100` を実行して選択肢として提示
- LLMプロバイダー呼び出し失敗（OPENAI_API_KEY / ANTHROPIC_API_KEY 未設定）→ `.env` を確認し、あれば読み込み、なければユーザーに確認

## Phase 1: 現在のプロンプトを抽出する

### プロンプトを含むLLM spansを見つける

```bash
# LLM spansを一覧表示（プロンプトが存在する場所）
ax spans list PROJECT_ID --filter "attributes.openinference.span.kind = 'LLM'" --limit 10

# モデルでフィルタ
ax spans list PROJECT_ID --filter "attributes.llm.model_name = 'gpt-4o'" --limit 10

# span名でフィルタ（例: 特定のLLM呼び出し）
ax spans list PROJECT_ID --filter "name = 'ChatCompletion'" --limit 10
```

### トレースをエクスポートしてプロンプト構造を確認する

```bash
# トレース内のすべてのspanをエクスポート
ax spans export --trace-id TRACE_ID --project PROJECT_ID

# 単一spanをエクスポート
ax spans export --span-id SPAN_ID --project PROJECT_ID
```

### エクスポートしたJSONからプロンプトを抽出する

```bash
# 構造化チャットメッセージを抽出（system + user + assistant）
jq '.[0] | {
  messages: .attributes.llm.input_messages,
  model: .attributes.llm.model_name
}' trace_*/spans.json

# systemプロンプトを特定して抽出
jq '[.[] | select(.attributes.llm.input_messages.roles[]? == "system")] | .[0].attributes.llm.input_messages' trace_*/spans.json

# プロンプトテンプレートと変数を抽出
jq '.[0].attributes.llm.prompt_template' trace_*/spans.json

# input.valueから抽出（非構造化プロンプトのフォールバック）
jq '.[0].attributes.input.value' trace_*/spans.json
```

### プロンプトをmessagesとして再構成する

spanデータを取得したら、プロンプトをmessages配列として再構成します。

```json
[
  {"role": "system", "content": "You are a helpful assistant that..."},
  {"role": "user", "content": "Given {input}, answer the question: {question}"}
]
```

spanに `attributes.llm.prompt_template.template` がある場合、プロンプトは変数を使用しています。これらのプレースホルダー（`{variable}` または `{{variable}}`）は保持してください -- 実行時に置換されます。

## Phase 2: パフォーマンスデータを収集する

### トレースから（本番フィードバック）

```bash
# エラーspanを検索 -- これらはプロンプト失敗を示す
ax spans list PROJECT_ID \
  --filter "status_code = 'ERROR' AND attributes.openinference.span.kind = 'LLM'" \
  --limit 20

# evalスコアが低いspanを検索
ax spans list PROJECT_ID \
  --filter "annotation.correctness.label = 'incorrect'" \
  --limit 20

# 高レイテンシspanを検索（プロンプトが過度に複雑な可能性）
ax spans list PROJECT_ID \
  --filter "attributes.openinference.span.kind = 'LLM' AND latency_ms > 10000" \
  --limit 20

# 詳細調査用にエラートレースをエクスポート
ax spans export --trace-id TRACE_ID --project PROJECT_ID
```

### データセットと実験から

```bash
# データセットをエクスポート（正解データ例）
ax datasets export DATASET_ID
# -> dataset_*/examples.json

# 実験結果をエクスポート（LLMが生成したもの）
ax experiments export EXPERIMENT_ID
# -> experiment_*/runs.json
```

### 分析のために dataset + experiment を結合する

`example_id` で2つのファイルを結合し、入力・出力・評価を並べて確認します。

```bash
# examplesとrunsの件数を確認
jq 'length' dataset_*/examples.json
jq 'length' experiment_*/runs.json

# 結合済みレコードを1件表示
jq -s '
  .[0] as $dataset |
  .[1][0] as $run |
  ($dataset[] | select(.id == $run.example_id)) as $example |
  {
    input: $example,
    output: $run.output,
    evaluations: $run.evaluations
  }
' dataset_*/examples.json experiment_*/runs.json

# 失敗例を抽出（eval score < threshold）
jq '[.[] | select(.evaluations.correctness.score < 0.5)]' experiment_*/runs.json
```

### 最適化対象を特定する

失敗全体でパターンを探します。

1. **出力と正解を比較**: LLM出力は期待値とどこが違うか？
2. **eval explanationを読む**: `eval.*.explanation` で失敗理由を把握
3. **annotation textを確認**: 人手フィードバックから具体的課題を把握
4. **冗長性の不一致を確認**: 出力が正解に対して長すぎる/短すぎるか
5. **フォーマット準拠を確認**: 出力は期待フォーマットか

## Phase 3: プロンプトを最適化する

### 最適化メタプロンプト

このテンプレートを使って改善版プロンプトを生成します。3つのプレースホルダーを埋め、LLM（GPT-4o, Claude など）に送ってください。

````
You are an expert in prompt optimization. Given the original baseline prompt
and the associated performance data (inputs, outputs, evaluation labels, and
explanations), generate a revised version that improves results.

ORIGINAL BASELINE PROMPT
========================

{PASTE_ORIGINAL_PROMPT_HERE}

========================

PERFORMANCE DATA
================

The following records show how the current prompt performed. Each record
includes the input, the LLM output, and evaluation feedback:

{PASTE_RECORDS_HERE}

================

HOW TO USE THIS DATA

1. Compare outputs: Look at what the LLM generated vs what was expected
2. Review eval scores: Check which examples scored poorly and why
3. Examine annotations: Human feedback shows what worked and what didn't
4. Identify patterns: Look for common issues across multiple examples
5. Focus on failures: The rows where the output DIFFERS from the expected
   value are the ones that need fixing

ALIGNMENT STRATEGY

- If outputs have extra text or reasoning not present in the ground truth,
  remove instructions that encourage explanation or verbose reasoning
- If outputs are missing information, add instructions to include it
- If outputs are in the wrong format, add explicit format instructions
- Focus on the rows where the output differs from the target -- these are
  the failures to fix

RULES

Maintain Structure:
- Use the same template variables as the current prompt ({var} or {{var}})
- Don't change sections that are already working
- Preserve the exact return format instructions from the original prompt

Avoid Overfitting:
- DO NOT copy examples verbatim into the prompt
- DO NOT quote specific test data outputs exactly
- INSTEAD: Extract the ESSENCE of what makes good vs bad outputs
- INSTEAD: Add general guidelines and principles
- INSTEAD: If adding few-shot examples, create SYNTHETIC examples that
  demonstrate the principle, not real data from above

Goal: Create a prompt that generalizes well to new inputs, not one that
memorizes the test data.

OUTPUT FORMAT

Return the revised prompt as a JSON array of messages:

[
  {"role": "system", "content": "..."},
  {"role": "user", "content": "..."}
]

Also provide a brief reasoning section (bulleted list) explaining:
- What problems you found
- How the revised prompt addresses each one
````

### パフォーマンスデータの準備

テンプレートに貼り付ける前に、レコードをJSON配列形式に整えます。

```bash
# dataset + experimentから: 結合して必要な列を抽出
jq -s '
  .[0] as $ds |
  [.[1][] | . as $run |
    ($ds[] | select(.id == $run.example_id)) as $ex |
    {
      input: $ex.input,
      expected: $ex.expected_output,
      actual_output: $run.output,
      eval_score: $run.evaluations.correctness.score,
      eval_label: $run.evaluations.correctness.label,
      eval_explanation: $run.evaluations.correctness.explanation
    }
  ]
' dataset_*/examples.json experiment_*/runs.json

# エクスポート済みspanから: annotation付きのinput/outputペアを抽出
jq '[.[] | select(.attributes.openinference.span.kind == "LLM") | {
  input: .attributes.input.value,
  output: .attributes.output.value,
  status: .status_code,
  model: .attributes.llm.model_name
}]' trace_*/spans.json
```

### 改善版プロンプトを適用する

LLMが改善版messages配列を返したら:

1. 元のプロンプトと改善版を並べて比較
2. すべてのテンプレート変数が保持されていることを確認
3. フォーマット指示が維持されていることを確認
4. 本番展開前に数件の例でテスト

## Phase 4: 反復する

### 最適化ループ

```
1. プロンプト抽出     -> Phase 1（一度だけ）
2. 実験を実行         -> ax experiments create ...
3. 結果をエクスポート -> ax experiments export EXPERIMENT_ID
4. 失敗を分析         -> jqで低スコアを抽出
5. メタプロンプト実行 -> 新しい失敗データでPhase 3を実行
6. 改善版プロンプトを適用
7. 手順2から繰り返す
```

### 改善を測定する

```bash
# 実験間でスコアを比較
# Experiment A（baseline）
jq '[.[] | .evaluations.correctness.score] | add / length' experiment_a/runs.json

# Experiment B（optimized）
jq '[.[] | .evaluations.correctness.score] | add / length' experiment_b/runs.json

# failからpassに変わった例を抽出
jq -s '
  [.[0][] | select(.evaluations.correctness.label == "incorrect")] as $fails |
  [.[1][] | select(.evaluations.correctness.label == "correct") |
    select(.example_id as $id | $fails | any(.example_id == $id))
  ] | length
' experiment_a/runs.json experiment_b/runs.json
```

### 2つのプロンプトをA/B比較する

1. 同じデータセットに対して、異なるプロンプト版で2つの実験を作成
2. 両方をエクスポート: `ax experiments export EXP_A` と `ax experiments export EXP_B`
3. 平均スコア、失敗率、具体的な反転例を比較
4. 退行を確認 -- プロンプトAでは通ったがプロンプトBでは失敗する例

## Prompt Engineering Best Practices

プロンプト作成・改訂時には次を適用してください。

| Technique | 適用タイミング | 例 |
|-----------|--------------|---------|
| 明確で詳細な指示 | 出力が曖昧/話題逸れする | "Classify the sentiment as exactly one of: positive, negative, neutral" |
| 指示を先頭に置く | モデルが後半の指示を無視する | 例の前にタスク説明を置く |
| ステップごとの分解 | 複雑な多段処理 | "First extract entities, then classify each, then summarize" |
| 具体的なペルソナ | 文体/トーンの一貫性が必要 | "You are a senior financial analyst writing for institutional investors" |
| 区切りトークン | セクションが混ざる | `---`, `###`, XMLタグで入力と指示を分離 |
| Few-shot例 | 出力形式の明確化が必要 | 合成の入力/出力ペアを2〜3件示す |
| 出力長の指定 | 応答が長すぎる/短すぎる | "Respond in exactly 2-3 sentences" |
| 推論指示 | 精度が重要 | "Think step by step before answering" |
| "I don't know" ガイドライン | 幻覚リスクがある | "If the answer is not in the provided context, say 'I don't have enough information'" |

### 変数保持

テンプレート変数を使うプロンプトを最適化するとき:

- **単一波括弧** (`{variable}`): Python f-string / Jinjaスタイル。Arizeで最も一般的。
- **二重波括弧** (`{{variable}}`): Mustacheスタイル。フレームワーク要件で使用。
- 最適化時に変数プレースホルダーを追加・削除しない
- 変数名を変更しない -- 実行時の置換は厳密な名前に依存
- Few-shot例を追加する場合は、変数プレースホルダーではなくリテラル値を使用

## ワークフロー

### 失敗トレースからプロンプトを最適化する

1. 失敗トレースを見つける:
   ```bash
   ax traces list PROJECT_ID --filter "status_code = 'ERROR'" --limit 5
   ```
2. トレースをエクスポート:
   ```bash
   ax spans export --trace-id TRACE_ID --project PROJECT_ID
   ```
3. LLM spanからプロンプトを抽出:
   ```bash
   jq '[.[] | select(.attributes.openinference.span.kind == "LLM")][0] | {
     messages: .attributes.llm.input_messages,
     template: .attributes.llm.prompt_template,
     output: .attributes.output.value,
     error: .attributes.exception.message
   }' trace_*/spans.json
   ```
4. エラーメッセージまたは出力から失敗箇所を特定
5. プロンプトとエラー文脈を使って最適化メタプロンプト（Phase 3）を埋める
6. 改善版プロンプトを適用

### データセットと実験を使って最適化する

1. データセットと実験を見つける:
   ```bash
   ax datasets list
   ax experiments list --dataset-id DATASET_ID
   ```
2. 両方をエクスポート:
   ```bash
   ax datasets export DATASET_ID
   ax experiments export EXPERIMENT_ID
   ```
3. メタプロンプト用に結合データを準備
4. 最適化メタプロンプトを実行
5. 改善を測定するため、改善版プロンプトで新しい実験を作成

### 出力フォーマットが誤るプロンプトをデバッグする

1. 出力フォーマットが誤っているspanをエクスポート:
   ```bash
   ax spans list PROJECT_ID \
     --filter "attributes.openinference.span.kind = 'LLM' AND annotation.format.label = 'incorrect'" \
     --limit 10 -o json > bad_format.json
   ```
2. LLM出力と期待値の差分を確認
3. 明示的なフォーマット指示をプロンプトに追加（JSON schema、例、区切り）
4. よくある修正: 望ましい出力形式を正確に示すfew-shot例を追加

### RAGプロンプトの幻覚を減らす

1. モデルが幻覚したトレースを見つける:
   ```bash
   ax spans list PROJECT_ID \
     --filter "annotation.faithfulness.label = 'unfaithful'" \
     --limit 20
   ```
2. retriever + LLM spansをまとめてエクスポートして確認:
   ```bash
   ax spans export --trace-id TRACE_ID --project PROJECT_ID
   jq '[.[] | {kind: .attributes.openinference.span.kind, name, input: .attributes.input.value, output: .attributes.output.value}]' trace_*/spans.json
   ```
3. 取得コンテキストに実際に答えが含まれていたか確認
4. systemプロンプトにグラウンディング指示を追加: "Only use information from the provided context. If the answer is not in the context, say so."

## トラブルシューティング

| Problem | Solution |
|---------|----------|
| `ax: command not found` | references/ax-setup.md を参照 |
| `No profile found` | プロファイルが未設定です。作成方法は references/ax-profiles.md を参照してください。 |
| spanに `input_messages` がない | span kindを確認 -- Chain/Agent spansは自分自身ではなく子のLLM spansにプロンプトを保持します |
| Prompt template が `null` | すべての計測が `prompt_template` を出力するわけではありません。代わりに `input_messages` または `input.value` を使用してください |
| 最適化後に変数が失われた | 改善版プロンプトが元のすべての `{var}` プレースホルダーを保持しているか確認 |
| 最適化で悪化した | 過学習を確認 -- メタプロンプトがテストデータを記憶した可能性。few-shot例が合成データであることを確認 |
| eval/annotation列がない | 先に評価を実行（Arize UI または SDK）してから再エクスポート |
| 実験出力列が見つからない | 列名は `{experiment_name}.output` -- `ax experiments get` で正確な実験名を確認 |
| span JSONで `jq` エラー | 正しいファイルパス（例: `trace_*/spans.json`）を対象にしているか確認 |

