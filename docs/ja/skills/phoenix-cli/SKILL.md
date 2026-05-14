---
name: phoenix-cli
description: Debug LLM applications using the Phoenix CLI. Fetch traces, analyze errors, review experiments, inspect datasets, and query the GraphQL API. Use when debugging AI/LLM applications, analyzing trace data, working with Phoenix observability, or investigating LLM performance issues.
license: Apache-2.0
compatibility: Requires Node.js (for npx) or global install of @arizeai/phoenix-cli. Optionally requires jq for JSON processing.
metadata:
  author: arize-ai
  version: "2.0.0"
---
# フェニックス CLI

## 呼び出し「」バッシュ
px <resource> <action> # グローバルにインストールされている場合
npx @arizeai/phoenix-cli <リソース> <アクション> # インストールは必要ありません
「」CLI は、`list` や `get` のようなサブコマンドを含む単一のリソース コマンドを使用します。「」バッシュ
pxトレースリスト
px トレース取得 <トレース ID>
ピクセルスパンリスト
pxデータセットリスト
px データセットを取得 <名前>
「」＃＃ 設定「」バッシュ
エクスポート PHOENIX_HOST=http://localhost:6006
import PHOENIX_PROJECT=私のプロジェクト
import PHOENIX_API_KEY=your-api-key # 認証が有効な場合
「」`jq` にパイプする場合は、常に `--format raw --no-progress` を使用してください。

## トレース「」バッシュ
px トレース リスト --limit 20 --format raw --no-progress | jq 。
px トレース リスト --last-n- minutes 60 --limit 20 --format raw --no-progress | jq '.[] | select(.status == "エラー")'
px トレース リスト --format raw --no-progress | jq 'sort_by(-.duration) | .[0:5]'
px トレース get <trace-id> --format raw | jq 。
px トレース get <trace-id> --format raw | jq '.spans[] | select(.status_code != "OK")'
「」## スパン「」バッシュ
px スパン リスト --limit 20 # 最近のスパン (テーブル ビュー)
px スパン リスト --last-n- minutes 60 --limit 50 # 過去 1 時間からのスパン
px スパンリスト --span-kind LLM --limit 10 # LLM スパンのみ
px スパンリスト --status-code ERROR --limit 20 # エラーのあるスパンのみ
px スパンリスト --name chat_completion --limit 10 # スパン名でフィルタリング
px スパン リスト --trace-id <id> --format raw --no-progress | jq 。   # トレースのすべてのスパン
px スパンリスト --include-annotations --limit 10 # 注釈スコアを含める
pxスパンリストoutput.json --limit 100 # JSONファイルに保存
px スパンリスト --format raw --no-progress | jq '.[] | select(.status_code == "エラー")'
「」### スパン JSON 形状「」
スパン
  name、span_kind ("LLM"|"CHAIN"|"TOOL"|"RETRIEVER"|"EMBEDDING"|"AGENT"|"RERANKER"|"GUARDRAIL"|"EVALUATOR"|"UNKNOWN")
  status_code ("OK"|"ERROR"|"UNSET")、status_message
  context.span_id、context.trace_id、parent_id
  開始時刻、終了時刻
  属性 (上記のトレース スパン属性と同じ)
  annotations[] (--include-annotations 付き)
    名前、結果 { スコア、ラベル、説明 }
「」### JSON 形状をトレースする「」
トレース
  トレース ID、ステータス ("OK"|"ERROR")、期間 (ミリ秒)、開始時刻、終了時刻
  rootSpan — 最上位のスパン (parent_id: null)
  スパン[]
    名前、span_kind ("LLM"|"CHAIN"|"TOOL"|"RETRIEVER"|"EMBEDDING"|"AGENT")
    status_code ("OK"|"ERROR")、parent_id、context.span_id
    属性
      input.value、output.value — 生の入力/出力
      llm.model_name、llm.provider
      llm.token_count.prompt/完了/合計
      llm.token_count.prompt_details.cache_read
      llm.token_count.completion_details.reasoning
      llm.input_messages.{N}.message.role/content
      llm.output_messages.{N}.message.role/content
      llm.invocation_parameters — JSON 文字列 (温度など)
      Exception.message — スパンエラーが発生した場合に設定されます
「」## セッション「」バッシュ
px セッション リスト --limit 10 --format raw --no-progress | jq 。
px セッション リスト --order asc --format raw --no-progress | jq '.[].session_id'
px セッション get <セッション ID> --format raw | jq 。
px セッション get <セッション ID> --include-annotations --format raw | jq '.annotations'
「」### セッション JSON 形式「」
セッションデータ
  ID、セッションID、プロジェクトID
  開始時刻、終了時刻
  痕跡[]
    id、trace_id、start_time、end_time

SessionAnnotation (--include-annotations 付き)
  id、名前、annotator_kind ("LLM"|"CODE"|"HUMAN")、session_id
  結果 { ラベル、スコア、説明 }
  メタデータ、識別子、ソース、created_at、updated_at
「」## データセット / 実験 / プロンプト「」バッシュ
px データセット リスト --format raw --no-progress | jq '.[].name'
px データセット get <名前> --format raw | jq '.examples[] | {入力、出力: .expected_output}'
px 実験リスト --dataset <名前> --format raw --no-progress | jq '.[] | {id、名前、failed_run_count}'
px 実験 get <id> --format raw --no-progress | jq '.[] | select(.error != null) | {入力、エラー}'
px プロンプト リスト --format raw --no-progress | jq '.[].name'
px プロンプト get <name> --format text --no-progress # プレーン テキスト、AI へのパイプに最適
「」## グラフQL

上記のコマンドでカバーされないアドホック クエリの場合。出力は `{"data": {...}}` です。「」バッシュ
px apigraphql '{ プロジェクト数 データセット数 プロンプト数 エバリュエーター数 }'
px apigraphql '{ プロジェクト { エッジ { ノード { 名前 トレースカウント tokenCountTotal } } } }' | jq '.data.projects.edges[].node'
px apigraphql '{ データセット { エッジ { ノード { 名前 exampleCount ExperimentCount } } }' | jq '.data.datasets.edges[].node'
px apigraphql '{ 評価者 { エッジ { ノード { 名前の種類 } } }' | jq '.data.evaluators.edges[].node'

# 任意の型をイントロスペクトする
px apigraphql '{ __type(name: "プロジェクト") { フィールド { 名前タイプ { 名前 } } } }' | jq '.data.__type.fields[]'
「」主要なルート フィールド: `projects`、`datasets`、`prompts`、`evaluators`、`projectCount`、`datasetCount`、`promptCount`、`evaluatorCount`、`viewer`。

## ドキュメント

コーディングエージェントがローカルで使用できるように、Phoenix ドキュメントのマークダウンをダウンロードします。「」バッシュ
px docs fetch # デフォルトのワークフロードキュメントを .px/docs にフェッチします
px docs fetch --workflow tracing # トレースドキュメントのみを取得します
px docs fetch --ワークフロー トレース --ワークフロー評価
px docs fetch --dry-run # ダウンロードされる内容をプレビューする
px docs fetch --refresh # .px/docs をクリアして再ダウンロード
px docs fetch --output-dir ./my-docs # カスタム出力ディレクトリ
「」キーオプション: `--workflow` (繰り返し可能、値: `tracing`、`evaluation`、`datasets`、`prompts`、`integrations`、`sdk`、`self-hosting`、`all`)、`--dry-run`、`--refresh`、`--output-dir` (デフォルト) `.px/docs`)、`--workers` (デフォルトは 10)。