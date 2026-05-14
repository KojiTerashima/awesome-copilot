---
name: arize-dataset
description: "Arize データセットとサンプルの作成・管理・クエリ時にこのスキルを呼び出してください。データセット CRUD、サンプル追加、データエクスポート、ax CLI を使ったファイルベースのデータセット作成をカバーします。"
---

# Arize Dataset スキル

## 概念

- **Dataset** = 評価や実験に使う、バージョン管理されたサンプルのコレクション
- **Dataset Version** = ある時点のデータセットのスナップショット。更新はインプレースの場合もあれば、新しいバージョンを作成する場合もあります
- **Example** = データセット内の単一レコード。任意のユーザー定義フィールド（例: `question`, `answer`, `context`）を持てます
- **Space** = 組織化のためのコンテナ。データセットはスペースに属します

サンプル上のシステム管理フィールド（`id`, `created_at`, `updated_at`）はサーバーで自動生成されます。作成または追加のペイロードには絶対に含めないでください。

## 前提条件

タスクはそのまま進めてください。必要な `ax` コマンドを実行します。事前にバージョン、環境変数、プロファイルを確認しないでください。

`ax` コマンドが失敗した場合は、エラーに応じてトラブルシュートします:
- `command not found` またはバージョンエラー → references/ax-setup.md を参照
- `401 Unauthorized` / API キー不足 → `ax profiles show` を実行して現在のプロファイルを確認。プロファイルがない、または API キーが誤っている場合: `.env` の `ARIZE_API_KEY` を確認し、references/ax-profiles.md に従ってプロファイルを作成/更新します。.env にもキーがない場合は、ユーザーに Arize API キーを確認してください（https://app.arize.com/admin > API Keys）
- Space ID が不明 → `.env` の `ARIZE_SPACE_ID` を確認、または `ax spaces list -o json` を実行、またはユーザーに確認
- Project が不明確 → `.env` の `ARIZE_DEFAULT_PROJECT` を確認、またはユーザーに確認、または `ax projects list -o json --limit 100` を実行して選択肢として提示

## データセット一覧: `ax datasets list`

スペース内のデータセットを参照します。出力は stdout に出ます。

```bash
ax datasets list
ax datasets list --space-id SPACE_ID --limit 20
ax datasets list --cursor CURSOR_TOKEN
ax datasets list -o json
```

### フラグ

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `--space-id` | string | from profile | スペースで絞り込む |
| `--limit, -l` | int | 15 | 最大件数 (1-100) |
| `--cursor` | string | none | 前回レスポンスのページネーションカーソル |
| `-o, --output` | string | table | 出力形式: table, json, csv, parquet, またはファイルパス |
| `-p, --profile` | string | default | 設定プロファイル |

## データセット取得: `ax datasets get`

メタデータをすばやく確認します。データセット名、スペース、タイムスタンプ、バージョン一覧を返します。

```bash
ax datasets get DATASET_ID
ax datasets get DATASET_ID -o json
```

### フラグ

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `DATASET_ID` | string | required | 位置引数 |
| `-o, --output` | string | table | 出力形式 |
| `-p, --profile` | string | default | 設定プロファイル |

### レスポンスフィールド

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | データセット ID |
| `name` | string | データセット名 |
| `space_id` | string | このデータセットが属するスペース |
| `created_at` | datetime | データセット作成時刻 |
| `updated_at` | datetime | 最終更新時刻 |
| `versions` | array | データセットバージョン一覧（id, name, dataset_id, created_at, updated_at） |

## データセットのエクスポート: `ax datasets export`

すべてのサンプルをファイルにダウンロードします。500 件を超えるデータセットでは `--all` を使用してください（無制限バルクエクスポート）。

```bash
ax datasets export DATASET_ID
# -> dataset_abc123_20260305_141500/examples.json

ax datasets export DATASET_ID --all
ax datasets export DATASET_ID --version-id VERSION_ID
ax datasets export DATASET_ID --output-dir ./data
ax datasets export DATASET_ID --stdout
ax datasets export DATASET_ID --stdout | jq '.[0]'
```

### フラグ

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `DATASET_ID` | string | required | 位置引数 |
| `--version-id` | string | latest | 特定バージョンのデータセットをエクスポート |
| `--all` | bool | false | 無制限バルクエクスポート（500 件超のデータセットで使用） |
| `--output-dir` | string | `.` | 出力ディレクトリ |
| `--stdout` | bool | false | ファイルではなく stdout に JSON を出力 |
| `-p, --profile` | string | default | 設定プロファイル |

**エージェント自動エスカレーションルール:** エクスポート結果がちょうど 500 件の場合、結果が切り詰められている可能性があります。完全なデータセットを取得するために `--all` で再実行してください。

**エクスポート完全性の検証:** エクスポート後、行数がサーバー報告値と一致することを確認します:
```bash
# データセットメタデータからサーバー報告件数を取得
ax datasets get DATASET_ID -o json | jq '.versions[-1] | {version: .id, examples: .example_count}'

# エクスポート結果と比較
jq 'length' dataset_*/examples.json

# 件数が異なる場合は --all で再エクスポート
```

出力はサンプルオブジェクトの JSON 配列です。各サンプルはシステムフィールド（`id`, `created_at`, `updated_at`）に加え、すべてのユーザー定義フィールドを持ちます:

```json
[
  {
    "id": "ex_001",
    "created_at": "2026-01-15T10:00:00Z",
    "updated_at": "2026-01-15T10:00:00Z",
    "question": "What is 2+2?",
    "answer": "4",
    "topic": "math"
  }
]
```

## データセット作成: `ax datasets create`

データファイルから新しいデータセットを作成します。

```bash
ax datasets create --name "My Dataset" --space-id SPACE_ID --file data.csv
ax datasets create --name "My Dataset" --space-id SPACE_ID --file data.json
ax datasets create --name "My Dataset" --space-id SPACE_ID --file data.jsonl
ax datasets create --name "My Dataset" --space-id SPACE_ID --file data.parquet
```

### フラグ

| Flag | Type | Required | Description |
|------|------|----------|-------------|
| `--name, -n` | string | yes | データセット名 |
| `--space-id` | string | yes | データセットを作成するスペース |
| `--file, -f` | path | yes | データファイル: CSV, JSON, JSONL, または Parquet |
| `-o, --output` | string | no | 返却されるデータセットメタデータの出力形式 |
| `-p, --profile` | string | no | 設定プロファイル |

### stdin 経由でデータを渡す

`--file -` を使うと、データを直接パイプできます。テンポラリファイルは不要です:

```bash
echo '[{"question": "What is 2+2?", "answer": "4"}]' | ax datasets create --name "my-dataset" --space-id SPACE_ID --file -

# または heredoc を使用
ax datasets create --name "my-dataset" --space-id SPACE_ID --file - << 'EOF'
[{"question": "What is 2+2?", "answer": "4"}]
EOF
```

既存データセットに行を追加する場合は、代わりに `ax datasets append --json '[...]'` を使ってください。ファイルは不要です。

### 対応ファイル形式

| Format | Extension | Notes |
|--------|-----------|-------|
| CSV | `.csv` | 列ヘッダーがフィールド名になります |
| JSON | `.json` | オブジェクト配列 |
| JSON Lines | `.jsonl` | 1 行に 1 オブジェクト（JSON 配列ではありません） |
| Parquet | `.parquet` | 列名がフィールド名になります。型を保持します |

**形式の注意点:**
- **CSV**: 型情報が失われます。日付は文字列になり、`null` は空文字になります。型を保持するには JSON/Parquet を使ってください。
- **JSONL**: 各行が独立した JSON オブジェクトです。`.jsonl` ファイル内の JSON 配列（`[{...}, {...}]`）は失敗します。代わりに `.json` 拡張子を使用してください。
- **Parquet**: 列型を保持します。ローカルでの読み取りには `pandas`/`pyarrow` が必要です: `pd.read_parquet("examples.parquet")`。

## サンプル追加: `ax datasets append`

既存データセットにサンプルを追加します。入力モードは 2 つあり、用途に合わせて選べます。

### インライン JSON（エージェント向け）

ペイロードを直接生成できます。テンポラリファイルは不要です:

```bash
ax datasets append DATASET_ID --json '[{"question": "What is 2+2?", "answer": "4"}]'

ax datasets append DATASET_ID --json '[
  {"question": "What is gravity?", "answer": "A fundamental force..."},
  {"question": "What is light?", "answer": "Electromagnetic radiation..."}
]'
```

### ファイルから

```bash
ax datasets append DATASET_ID --file new_examples.csv
ax datasets append DATASET_ID --file additions.json
```

### 特定バージョンへ

```bash
ax datasets append DATASET_ID --json '[{"q": "..."}]' --version-id VERSION_ID
```

### フラグ

| Flag | Type | Required | Description |
|------|------|----------|-------------|
| `DATASET_ID` | string | yes | 位置引数 |
| `--json` | string | mutex | サンプルオブジェクトの JSON 配列 |
| `--file, -f` | path | mutex | データファイル（CSV, JSON, JSONL, Parquet） |
| `--version-id` | string | no | 特定バージョンに追加（デフォルト: latest） |
| `-o, --output` | string | no | 返却されるデータセットメタデータの出力形式 |
| `-p, --profile` | string | no | 設定プロファイル |

`--json` または `--file` のどちらか一方を必ず指定する必要があります。

### バリデーション

- 各サンプルは、少なくとも 1 つのユーザー定義フィールドを持つ JSON オブジェクトである必要があります
- 1 リクエストあたり最大 100,000 サンプル

**追加前のスキーマ検証:** データセットに既存サンプルがある場合、フィールド不一致を見逃さないため、追加前にスキーマを確認してください:

```bash
# データセット内の既存フィールド名を確認
ax datasets export DATASET_ID --stdout | jq '.[0] | keys'

# 新規データのフィールド名が一致するか確認
echo '[{"question": "..."}]' | jq '.[0] | keys'

# 両方の出力に同じユーザー定義フィールドが表示されるべき
```

フィールドは自由形式です。新しいサンプルの追加フィールドは列として追加され、欠けたフィールドは null になります。ただし、フィールド名のタイプミス（例: `queston` と `question`）は新しい列を黙って作成してしまうため、追加前にスペルを確認してください。

## データセット削除: `ax datasets delete`

```bash
ax datasets delete DATASET_ID
ax datasets delete DATASET_ID --force   # 確認プロンプトをスキップ
```

### フラグ

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `DATASET_ID` | string | required | 位置引数 |
| `--force, -f` | bool | false | 確認プロンプトをスキップ |
| `-p, --profile` | string | default | 設定プロファイル |

## ワークフロー

### 名前でデータセットを探す

ユーザーは ID ではなく名前でデータセットを指定することがよくあります。ほかのコマンドを実行する前に、名前を ID に解決してください:

```bash
# 名前からデータセット ID を取得
ax datasets list -o json | jq '.[] | select(.name == "eval-set-v1") | .id'

# 一覧がページ分割される場合はさらに取得
ax datasets list -o json --limit 100 | jq '.[] | select(.name | test("eval-set")) | {id, name}'
```

### 評価用にファイルからデータセットを作成

1. 評価用カラム（例: `input`, `expected_output`）を含む CSV/JSON/Parquet ファイルを準備
   - データをインライン生成する場合は、stdin 経由で `--file -` を使ってパイプします（Create Dataset セクション参照）
2. `ax datasets create --name "eval-set-v1" --space-id SPACE_ID --file eval_data.csv`
3. 検証: `ax datasets get DATASET_ID`
4. データセット ID を使って実験を実行

### 既存データセットにサンプルを追加

```bash
# データセットを探す
ax datasets list

# インラインまたはファイルから追加（完全な構文は Append Examples セクション参照）
ax datasets append DATASET_ID --json '[{"question": "...", "answer": "..."}]'
ax datasets append DATASET_ID --file additional_examples.csv
```

### オフライン分析用にデータセットをダウンロード

1. `ax datasets list` -- データセットを見つける
2. `ax datasets export DATASET_ID` -- ファイルにダウンロード
3. JSON を解析: `jq '.[] | .question' dataset_*/examples.json`

### 特定バージョンをエクスポート

```bash
# バージョン一覧
ax datasets get DATASET_ID -o json | jq '.versions'

# そのバージョンをエクスポート
ax datasets export DATASET_ID --version-id VERSION_ID
```

### データセットを反復改善する

1. 現在バージョンをエクスポート: `ax datasets export DATASET_ID`
2. ローカルでサンプルを修正
3. 新しい行を追加: `ax datasets append DATASET_ID --file new_rows.csv`
4. または新しいバージョンを作成: `ax datasets create --name "eval-set-v2" --space-id SPACE_ID --file updated_data.json`

### エクスポートを他ツールにパイプする

```bash
# サンプル数をカウント
ax datasets export DATASET_ID --stdout | jq 'length'

# 単一フィールドを抽出
ax datasets export DATASET_ID --stdout | jq '.[].question'

# jq で CSV に変換
ax datasets export DATASET_ID --stdout | jq -r '.[] | [.question, .answer] | @csv'
```

## Dataset Example スキーマ

サンプルは自由形式の JSON オブジェクトです。固定スキーマはなく、列は提供したフィールドそのものになります。システム管理フィールドはサーバーで追加されます:

| Field | Type | Managed by | Notes |
|-------|------|-----------|-------|
| `id` | string | server | 自動生成 UUID。更新時は必須、作成/追加時は指定禁止 |
| `created_at` | datetime | server | 不変の作成タイムスタンプ |
| `updated_at` | datetime | server | 更新時に自動更新 |
| *(any user field)* | any JSON type | user | 文字列、数値、真偽値、null、ネストオブジェクト、配列 |


## 関連スキル

- **arize-trace**: 本番 span をエクスポートして、データセットに入れるべきデータを把握 → `arize-trace` を使用
- **arize-experiment**: このデータセットに対して評価を実行 → 次のステップは `arize-experiment`
- **arize-prompt-optimization**: データセット + 実験結果を使ってプロンプトを改善 → `arize-prompt-optimization` を使用

## トラブルシューティング

| Problem | Solution |
|---------|----------|
| `ax: command not found` | references/ax-setup.md を参照 |
| `401 Unauthorized` | API キーが誤っている、期限切れ、またはこのスペースへのアクセス権がありません。references/ax-profiles.md を使ってプロファイルを修正してください。 |
| `No profile found` | プロファイルが設定されていません。作成方法は references/ax-profiles.md を参照してください。 |
| `Dataset not found` | `ax datasets list` でデータセット ID を確認 |
| `File format error` | 対応形式: CSV, JSON, JSONL, Parquet。stdin から読むには `--file -` を使用。 |
| `platform-managed column` | 作成/追加ペイロードから `id`, `created_at`, `updated_at` を削除 |
| `reserved column` | `time`, `count`, または `source_record_*` フィールドを削除 |
| `Provide either --json or --file` | append では入力ソースをちょうど 1 つ指定する必要があります |
| `Examples array is empty` | JSON 配列またはファイルに少なくとも 1 件のサンプルがあることを確認 |
| `not a JSON object` | `--json` 配列の各要素は文字列や数値ではなく `{...}` オブジェクトである必要があります |

## 将来利用のために認証情報を保存する

references/ax-profiles.md の「§ Save Credentials for Future Use」を参照してください。

