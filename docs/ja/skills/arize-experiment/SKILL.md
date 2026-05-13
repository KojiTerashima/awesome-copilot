---
name: arize-experiment
description: "Arize 実験を作成・実行・分析するときはこのスキルを呼び出してください。ax CLI を使った実験の CRUD、実行結果のエクスポート、結果比較、評価ワークフローを扱います。"
---

# Arize Experiment スキル

## 概念

- **Experiment** = 特定のデータセットバージョンに対する名前付き評価実行で、各 example につき 1 つの run を含む
- **Experiment Run** = 1 つのデータセット example を処理した結果 -- モデル出力、任意の評価、任意のメタデータを含む
- **Dataset** = バージョン管理された example のコレクション。すべての実験はデータセットと特定のデータセットバージョンに紐づく
- **Evaluation** = run に付与される名前付きメトリクス（例: `correctness`, `relevance`）。任意でラベル、スコア、説明を含む

典型的な流れ: データセットをエクスポート → 各 example を処理 → 出力と評価を収集 → run を含む実験を作成。

## 前提条件

タスクにそのまま進み、必要な `ax` コマンドを実行してください。事前にバージョン、環境変数、プロファイルを確認しないでください。

`ax` コマンドが失敗した場合は、エラーに基づいて対処してください:
- `command not found` またはバージョンエラー → references/ax-setup.md を参照
- `401 Unauthorized` / API キー不足 → `ax profiles show` を実行して現在のプロファイルを確認。プロファイルがない、または API キーが誤っている場合: `.env` の `ARIZE_API_KEY` を確認し、references/ax-profiles.md を使ってプロファイルを作成/更新。`.env` にもキーがない場合は、ユーザーに Arize API キーを確認（https://app.arize.com/admin > API Keys）
- Space ID が不明 → `.env` の `ARIZE_SPACE_ID` を確認、または `ax spaces list -o json` を実行、またはユーザーに確認
- Project が不明確 → `.env` の `ARIZE_DEFAULT_PROJECT` を確認、またはユーザーに確認、または `ax projects list -o json --limit 100` を実行して選択肢として提示

## 実験一覧: `ax experiments list`

必要に応じてデータセットで絞り込みつつ、実験を参照します。出力は stdout に表示されます。

```bash
ax experiments list
ax experiments list --dataset-id DATASET_ID --limit 20
ax experiments list --cursor CURSOR_TOKEN
ax experiments list -o json
```

### フラグ

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `--dataset-id` | string | none | データセットで絞り込み |
| `--limit, -l` | int | 15 | 最大件数 (1-100) |
| `--cursor` | string | none | 前回レスポンスのページネーションカーソル |
| `-o, --output` | string | table | 出力形式: table, json, csv, parquet, またはファイルパス |
| `-p, --profile` | string | default | 設定プロファイル |

## 実験取得: `ax experiments get`

メタデータを素早く確認します -- 実験名、紐づくデータセット/バージョン、タイムスタンプを返します。

```bash
ax experiments get EXPERIMENT_ID
ax experiments get EXPERIMENT_ID -o json
```

### フラグ

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `EXPERIMENT_ID` | string | required | 位置引数 |
| `-o, --output` | string | table | 出力形式 |
| `-p, --profile` | string | default | 設定プロファイル |

### レスポンスフィールド

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | 実験 ID |
| `name` | string | 実験名 |
| `dataset_id` | string | 紐づくデータセット ID |
| `dataset_version_id` | string | 使用した特定のデータセットバージョン |
| `experiment_traces_project_id` | string | 実験トレースが保存されるプロジェクト |
| `created_at` | datetime | 実験が作成された時刻 |
| `updated_at` | datetime | 最終更新時刻 |

## 実験エクスポート: `ax experiments export`

すべての run をファイルへダウンロードします。デフォルトは REST API を使用し、`--all` を指定すると大量転送向けに Arrow Flight を使用します。

```bash
ax experiments export EXPERIMENT_ID
# -> experiment_abc123_20260305_141500/runs.json

ax experiments export EXPERIMENT_ID --all
ax experiments export EXPERIMENT_ID --output-dir ./results
ax experiments export EXPERIMENT_ID --stdout
ax experiments export EXPERIMENT_ID --stdout | jq '.[0]'
```

### フラグ

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `EXPERIMENT_ID` | string | required | 位置引数 |
| `--all` | bool | false | 大量エクスポートに Arrow Flight を使用（下記参照） |
| `--output-dir` | string | `.` | 出力ディレクトリ |
| `--stdout` | bool | false | ファイルではなく stdout に JSON を出力 |
| `-p, --profile` | string | default | 設定プロファイル |

### REST と Flight (`--all`)

- **REST**（デフォルト）: 手間が少ない -- Arrow/Flight 依存なし、標準 HTTPS ポート、企業プロキシやファイアウォール経由でも動作。1 ページあたり最大 500 run。
- **Flight**（`--all`）: 500 run を超える実験で必須。別ホスト/ポート（`flight.arize.com:443`）で gRPC+TLS を使うため、企業ネットワークによってはブロックされる場合があります。

**エージェント自動昇格ルール:** REST エクスポートがちょうど 500 run を返した場合、結果は切り詰められている可能性が高いです。完全なデータセットを取得するため、`--all` で再実行してください。

出力は run オブジェクトの JSON 配列です:

```json
[
  {
    "id": "run_001",
    "example_id": "ex_001",
    "output": "The answer is 4.",
    "evaluations": {
      "correctness": { "label": "correct", "score": 1.0 },
      "relevance": { "score": 0.95, "explanation": "Directly answers the question" }
    },
    "metadata": { "model": "gpt-4o", "latency_ms": 1234 }
  }
]
```

## 実験作成: `ax experiments create`

データファイル内の run を使って新しい実験を作成します。

```bash
ax experiments create --name "gpt-4o-baseline" --dataset-id DATASET_ID --file runs.json
ax experiments create --name "claude-test" --dataset-id DATASET_ID --file runs.csv
```

### フラグ

| Flag | Type | Required | Description |
|------|------|----------|-------------|
| `--name, -n` | string | yes | 実験名 |
| `--dataset-id` | string | yes | 実験を実行する対象データセット |
| `--file, -f` | path | yes | run を含むデータファイル: CSV, JSON, JSONL, または Parquet |
| `-o, --output` | string | no | 出力形式 |
| `-p, --profile` | string | no | 設定プロファイル |

### stdin 経由でデータを渡す

`--file -` を使うとデータを直接パイプできます -- 一時ファイルは不要です:

```bash
echo '[{"example_id": "ex_001", "output": "Paris"}]' | ax experiments create --name "my-experiment" --dataset-id DATASET_ID --file -

# または heredoc を使用
ax experiments create --name "my-experiment" --dataset-id DATASET_ID --file - << 'EOF'
[{"example_id": "ex_001", "output": "Paris"}]
EOF
```

### runs ファイルの必須カラム

| Column | Type | Required | Description |
|--------|------|----------|-------------|
| `example_id` | string | yes | この run が対応するデータセット example の ID |
| `output` | string | yes | この example に対するモデル/システム出力 |

追加カラムは run の `additionalProperties` としてそのまま渡されます。

## 実験削除: `ax experiments delete`

```bash
ax experiments delete EXPERIMENT_ID
ax experiments delete EXPERIMENT_ID --force   # 確認プロンプトをスキップ
```

### フラグ

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `EXPERIMENT_ID` | string | required | 位置引数 |
| `--force, -f` | bool | false | 確認プロンプトをスキップ |
| `-p, --profile` | string | default | 設定プロファイル |

## Experiment Run スキーマ

各 run は 1 つのデータセット example に対応します:

```json
{
  "example_id": "required -- links to dataset example",
  "output": "required -- the model/system output for this example",
  "evaluations": {
    "metric_name": {
      "label": "optional string label (e.g., 'correct', 'incorrect')",
      "score": "optional numeric score (e.g., 0.95)",
      "explanation": "optional freeform text"
    }
  },
  "metadata": {
    "model": "gpt-4o",
    "temperature": 0.7,
    "latency_ms": 1234
  }
}
```

### 評価フィールド

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `label` | string | no | カテゴリ分類（例: `correct`, `incorrect`, `partial`） |
| `score` | number | no | 数値品質スコア（例: 0.0 - 1.0） |
| `explanation` | string | no | 評価理由の自由記述 |

各評価には `label`、`score`、`explanation` のうち少なくとも 1 つを含めてください。

## ワークフロー

### データセットに対して実験を実行する

1. データセットを見つける、または作成する:
   ```bash
   ax datasets list
   ax datasets export DATASET_ID --stdout | jq 'length'
   ```
2. データセット example をエクスポートする:
   ```bash
   ax datasets export DATASET_ID
   ```
3. 各 example をシステムで処理し、出力と評価を収集する
4. `example_id`、`output`、任意の `evaluations` を含む runs ファイル（JSON 配列）を作成する:
   ```json
   [
     {"example_id": "ex_001", "output": "4", "evaluations": {"correctness": {"label": "correct", "score": 1.0}}},
     {"example_id": "ex_002", "output": "Paris", "evaluations": {"correctness": {"label": "correct", "score": 1.0}}}
   ]
   ```
5. 実験を作成する:
   ```bash
   ax experiments create --name "gpt-4o-baseline" --dataset-id DATASET_ID --file runs.json
   ```
6. 確認: `ax experiments get EXPERIMENT_ID`

### 2 つの実験を比較する

1. 両方の実験をエクスポートする:
   ```bash
   ax experiments export EXPERIMENT_ID_A --stdout > a.json
   ax experiments export EXPERIMENT_ID_B --stdout > b.json
   ```
2. `example_id` ごとに評価スコアを比較する:
   ```bash
   # 実験 A の correctness 平均スコア
   jq '[.[] | .evaluations.correctness.score] | add / length' a.json

   # 実験 B も同様
   jq '[.[] | .evaluations.correctness.score] | add / length' b.json
   ```
3. 結果が異なる example を見つける:
   ```bash
   jq -s '.[0] as $a | .[1][] | . as $run |
     {
       example_id: $run.example_id,
       b_score: $run.evaluations.correctness.score,
       a_score: ($a[] | select(.example_id == $run.example_id) | .evaluations.correctness.score)
     }' a.json b.json
   ```
4. 評価者ごとのスコア分布（pass/fail/partial 件数）:
   ```bash
   # 実験 A のラベル別件数
   jq '[.[] | .evaluations.correctness.label] | group_by(.) | map({label: .[0], count: length})' a.json
   ```
5. 回帰を見つける（A では pass、B では fail の example）:
   ```bash
   jq -s '
     [.[0][] | select(.evaluations.correctness.label == "correct")] as $passed_a |
     [.[1][] | select(.evaluations.correctness.label != "correct") |
       select(.example_id as $id | $passed_a | any(.example_id == $id))
     ]
   ' a.json b.json
   ```

**統計的有意性メモ:** スコア比較の信頼性は、評価者ごとに example が 30 件以上ある場合が最も高くなります。件数が少ない場合は差分を方向性の目安として扱ってください -- n=10 で 5% の差はノイズの可能性があります。スコアと合わせてサンプルサイズも報告してください: `jq 'length' a.json`。

### 分析用に実験結果をダウンロードする

1. `ax experiments list --dataset-id DATASET_ID` -- 実験を探す
2. `ax experiments export EXPERIMENT_ID` -- ファイルにダウンロード
3. 解析: `jq '.[] | {example_id, score: .evaluations.correctness.score}' experiment_*/runs.json`

### エクスポート結果を他ツールにパイプする

```bash
# run 件数をカウント
ax experiments export EXPERIMENT_ID --stdout | jq 'length'

# すべての出力を抽出
ax experiments export EXPERIMENT_ID --stdout | jq '.[].output'

# 低スコアの run を取得
ax experiments export EXPERIMENT_ID --stdout | jq '[.[] | select(.evaluations.correctness.score < 0.5)]'

# CSV に変換
ax experiments export EXPERIMENT_ID --stdout | jq -r '.[] | [.example_id, .output, .evaluations.correctness.score] | @csv'
```

## 関連スキル

- **arize-dataset**: この実験の対象データセットを作成またはエクスポート → まず `arize-dataset` を使用
- **arize-prompt-optimization**: 実験結果を使ってプロンプトを改善 → 次のステップは `arize-prompt-optimization`
- **arize-trace**: 失敗した実験 run の個別スパントレースを調査 → `arize-trace` を使用
- **arize-link**: 実験 run からトレースへのクリック可能な UI リンクを生成 → `arize-link` を使用

## トラブルシューティング

| Problem | Solution |
|---------|----------|
| `ax: command not found` | references/ax-setup.md を参照 |
| `401 Unauthorized` | API キーが不正、期限切れ、またはこの space へのアクセス権がありません。references/ax-profiles.md を使ってプロファイルを修正してください。 |
| `No profile found` | プロファイルが設定されていません。作成方法は references/ax-profiles.md を参照。 |
| `Experiment not found` | `ax experiments list` で experiment ID を確認 |
| `Invalid runs file` | 各 run には `example_id` と `output` フィールドが必要 |
| `example_id mismatch` | `example_id` の値がデータセットの ID と一致していることを確認（検証のためデータセットをエクスポート） |
| `No runs found` | エクスポート結果が空です -- `ax experiments get` で実験に run があるか確認 |
| `Dataset not found` | 紐づいたデータセットが削除されている可能性があります。`ax datasets list` で確認 |

## 将来利用のために認証情報を保存する

references/ax-profiles.md の「§ Save Credentials for Future Use」を参照してください。

