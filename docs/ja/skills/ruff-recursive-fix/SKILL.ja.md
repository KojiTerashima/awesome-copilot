---
name: ruff-recursive-fix
description: Run Ruff checks with optional scope and rule overrides, apply safe and unsafe autofixes iteratively, review each change, and resolve remaining findings with targeted edits or user decisions.
---
# Ruff 再帰的修正

## 概要

このスキルを使用して、制御された反復ワークフローで Ruff によるコード品質を強化します。
以下をサポートします。

- オプションの範囲を特定のフォルダーに制限します。
- `pyproject.toml` からのデフォルトのプロジェクト設定。
- 柔軟な Ruff 呼び出し (`uv`、直接 `ruff`、`python -m ruff`、または同等のもの)。
- オプションの実行ごとのルールの上書き (`--select`、`--ignore`、`--extend-select`、`--extend-ignore`)。
- 安全な自動修正と安全でない自動修正。
- 各修正パス後の差分レビュー。
- 結果が解決されるか決定が必要になるまで、再帰的に繰り返します。
- 抑制が正当な場合にのみ、インライン `# noqa` を賢明に使用します。

## 入力

実行する前に次の入力を収集します。

- `target_path` (オプション): チェックするフォルダーまたはファイル。空とはリポジトリ全体を意味します。
- `ruff_runner` (オプション): 明示的な Ruff コマンド プレフィックス (例: `uv run`、`ruff`、`python -m ruff`、`pipx run ruff`)。
- `rules_select` (オプション): 適用するコンマ区切りのルール コード。
- `rules_ignore` (オプション): 無視するコンマ区切りのルール コード。
- `extend_select` (オプション): 設定されたデフォルトを置き換えずに追加する追加のルール。
- `extend_ignore` (オプション): 設定されたデフォルトを置き換えずに無視される追加のルール。
- `allow_unsafe_fixes` (デフォルト: true): Ruff の安全でない修正を実行するかどうか。
- `ask_on_ambiguity` (デフォルト: true): 複数の有効な選択肢が存在する場合は、常にユーザーに質問します。

## コマンドの構築

入力から Ruff コマンドを構築します。

### 0.ラフランナーを解決する

コマンドを構築する前に、再利用可能な `ruff_cmd` プレフィックスを決定してください。

解決順序:

1. `ruff_runner` が提供されている場合は、そのまま使用します。
2. `uv` が利用可能で、Ruff が `uv` を通じて管理されている場合は、`uv run ruff` を使用します。
3. それ以外の場合、`ruff` が `PATH` で利用可能な場合は、`ruff` を使用します。
4. Python が利用可能で、その環境に Ruff がインストールされている場合は、`python -m ruff` を使用します。
5. それ以外の場合は、インストールされた Ruff を呼び出すプロジェクト固有の同等の機能 (`pipx run ruff` など) を使用するか、停止してユーザーに尋ねます。

ワークフロー内のすべての `check` および `format` コマンドには、同じ解決された `ruff_cmd` を使用します。

基本コマンド:```bash
<ruff_cmd> check
```フォーマッタコマンド:```bash
<ruff_cmd> format
```オプションのターゲットを使用する場合:```bash
<ruff_cmd> format <target_path>
```オプションのターゲットを追加します。```bash
<ruff_cmd> check <target_path>
```必要に応じて、オプションのオーバーライドを追加します。```bash
--select <codes>
--ignore <codes>
--extend-select <codes>
--extend-ignore <codes>
```例:```bash
# Full project with defaults from pyproject.toml
ruff check

# One folder with defaults
python -m ruff check src/models

# Override to skip docs and TODO-like rules for this run
uv run ruff check src --extend-ignore D,TD

# Check only selected rules in a folder
ruff check src/data --select F,E9,I
```## ワークフロー

### 1. ベースライン分析

1. 選択したスコープとオプションを使用して `<ruff_cmd> check` を実行します。
2. 検出結果をタイプ別に分類します。
	- 自動修復可能な金庫。
	- 自動修正可能で安全ではありません。
	- 自動修正はできません。
3. 所見が残らない場合は中止します。

### 2. 安全な Autofix パス

1. 同じスコープ/オプションを使用して `--fix` で Ruff を実行します。
2. 結果の diff を注意深く確認して、セマンティックな正確さとスタイルの一貫性を確認します。
3. 同じスコープで `<ruff_cmd> format` を実行します。
4. `<ruff_cmd> check` を再実行して、残りの結果を更新します。

### 3. 安全でない Autofix パス

検出結果が残っており、`allow_unsafe_fixes=true` の場合にのみ実行します。

1. 同じスコープ/オプションを使用して `--fix --unsafe-fixes` で Ruff を実行します。
2. 結果の差分を注意深く確認し、動作に応じた編集を優先します。
3. 同じスコープで `<ruff_cmd> format` を実行します。
4. `<ruff_cmd> check` を再実行します。

### 4. 手動修復パス

残りの調査結果については、次のとおりです。

1. 明確で安全な修正がある場合は、コード内で直接修正します。
2. 編集は最小限かつローカルに保ちます。
3. 同じスコープで `<ruff_cmd> format` を実行します。
4. `<ruff_cmd> check` を再実行します。

### 5. 曖昧さに関するポリシー

いずれかのステップで有効な解決策が複数ある場合は、続行する前に必ずユーザーに質問してください。
同等のオプションの間で黙って選択しないでください。

### 6. 抑制の決定 (`# noqa`)

すべての条件が true の場合にのみ抑制を使用します。

- ルールが、必要な動作、パブリック API、フレームワーク規約、または可読性の目標と矛盾しています。
- リファクタリングはルールの値に不釣り合いになります。
- 抑制は範囲が狭く、特定的です (単一行、可能な場合は明示的なコード)。

ガイドライン:

- 広範囲の `# noqa` よりも `# noqa: <RULE>` を優先します。
- 明らかではない抑制についての簡単な理由のコメントを追加します。
- 有効な結果が 2 つ以上存在する場合は、どのオプションを優先するかを常にユーザーに尋ねます。

### 7. 再帰ループと停止基準

次のいずれかの結果が得られるまで、手順 2 ～ 6 を繰り返します。

- `<ruff_cmd> check` はクリーンな状態を返します。
- 残りの調査結果には、アーキテクチャ/製品の決定が必要です。
- 残りの発見は、文書化された理論的根拠によって意図的に隠蔽されます。
- ループを繰り返しても進みません。

各ループ反復には、次の `<ruff_cmd> check` の前に `<ruff_cmd> format` を含める必要があります。

進行状況が検出されない場合:

1. ブロックされたルールと影響を受けるファイルを要約します。
2. 有効なオプションとトレードオフを提示します。
3. ユーザーに選択を求めます。

## 品質ゲート完了を宣言する前に:

- Ruff は、選択したスコープ/オプションに関して予期しない結果を返しません。
- すべての自動修正の差分が正確であるかどうかがレビューされます。
- 明示的な正当な理由がない限り、抑制は追加されません。
- 動作に影響を与える可能性のある安全でない修正は、ユーザーに対して強調表示されます。
- Ruff フォーマットは反復ごとに実行されます。

## 出力コントラクト

実行の最後に、次のことを報告します。

- スコープとラフのオプションが使用されます。
- 実行された反復の数。
- 修正された調査結果の概要。
- 手動修正のリスト。
- 根拠のある抑制のリスト。
- 残りの調査結果 (存在する場合)、およびユーザーの決定が必要。

## 推奨されるプロンプト スターター

- 「デフォルト設定を使用してリポジトリ全体で ruff-recursive-fix を実行します。」
- 「src/models に対してのみ ruff-recursive-fix を実行し、DOC ルールを無視します。」
- 「F、E9、I を選択し、安全でない修正を含まないテストで ruff-recursive-fix を実行します。」
- 「src/data で ruff-recursive-fix を実行し、noqa を追加する前に私に尋ねてください。」