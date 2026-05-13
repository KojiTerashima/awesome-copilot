---
name: arize-link
description: Arize UI へのディープリンクを生成します。特定の trace、span、session、dataset、labeling queue、evaluator、annotation config を開くクリック可能な URL が必要なときに使用します。
---

# Arize Link

trace、span、session、dataset、labeling queue、evaluator、annotation config 向けに、Arize UI へのディープリンクを生成します。

## 使うタイミング

- trace、span、session、dataset、labeling queue、evaluator、annotation config へのリンクをユーザーが求めている
- エクスポート済みデータやログの ID を使って UI に戻るリンクを作る必要がある
- 上記リソースを Arize で「open」または「view」したいとユーザーが依頼している

## 必須入力

ユーザーまたはコンテキスト（エクスポート済み trace データ、解析済み URL）から収集します。

| 常に必須 | リソース固有 |
|---|---|
| `org_id` (base64) | `project_id` + `trace_id` [+ `span_id`] — trace/span |
| `space_id` (base64) | `project_id` + `session_id` — session |
| | `dataset_id` — dataset |
| | `queue_id` — 特定の queue（一覧の場合は省略） |
| | `evaluator_id` [+ `version`] — evaluator |

**すべてのパス ID は base64 エンコード済みである必要があります**（文字: `A-Za-z0-9+/=`）。生の数値 ID を使うと見た目は正しい URL でも 404 になります。ユーザーが数値を渡してきた場合は、Arize のブラウザー URL（`https://app.arize.com/organizations/{org_id}/spaces/{space_id}/…`）から ID を直接コピーしてもらってください。生の内部 ID（例: `Organization:1:abC1`）しかない場合は、URL に挿入する前に base64 エンコードしてください。

## URL テンプレート

ベース URL: `https://app.arize.com`（オンプレミス環境では上書き）

**Trace**（特定の span をハイライトするには `&selectedSpanId={span_id}` を追加）:
```
{base_url}/organizations/{org_id}/spaces/{space_id}/projects/{project_id}?selectedTraceId={trace_id}&queryFilterA=&selectedTab=llmTracing&timeZoneA=America%2FLos_Angeles&startA={start_ms}&endA={end_ms}&envA=tracing&modelType=generative_llm
```

**Session:**
```
{base_url}/organizations/{org_id}/spaces/{space_id}/projects/{project_id}?selectedSessionId={session_id}&queryFilterA=&selectedTab=llmTracing&timeZoneA=America%2FLos_Angeles&startA={start_ms}&endA={end_ms}&envA=tracing&modelType=generative_llm
```

**Dataset**（`selectedTab`: `examples` または `experiments`）:
```
{base_url}/organizations/{org_id}/spaces/{space_id}/datasets/{dataset_id}?selectedTab=examples
```

**Queue 一覧 / 特定 queue:**
```
{base_url}/organizations/{org_id}/spaces/{space_id}/queues
{base_url}/organizations/{org_id}/spaces/{space_id}/queues/{queue_id}
```

**Evaluator**（最新を使う場合は `?version=…` を省略）:
```
{base_url}/organizations/{org_id}/spaces/{space_id}/evaluators/{evaluator_id}
{base_url}/organizations/{org_id}/spaces/{space_id}/evaluators/{evaluator_id}?version={version_url_encoded}
```
`version` の値は URL エンコードされている必要があります（例: 末尾の `=` → `%3D`）。

**Annotation configs:**
```
{base_url}/organizations/{org_id}/spaces/{space_id}/annotation-configs
```

## 時間範囲

重要: trace/span/session リンクでは `startA` と `endA`（エポックミリ秒）が**必須**です。省略すると直近 7 日間が既定になり、その範囲外の trace は "no recent data" と表示されます。

**優先順:**
1. **ユーザー提供の URL** — `startA`/`endA` を直接抽出して再利用する。
2. **Span の `start_time`** — ±1 日（より狭くするなら ±1 時間）を付与する。
3. **フォールバック** — 直近 90 日（`now - 90d` から `now`）。

できるだけ狭い時間範囲を優先してください。90 日ウィンドウは読み込みが遅くなります。

## 手順

1. ユーザー、エクスポート済みデータ、または URL コンテキストから ID を収集する。
2. すべてのパス ID が base64 エンコード済みであることを確認する。
3. 上記の優先順で `startA`/`endA` を決定する。
4. 適切なテンプレートに値を代入し、クリック可能な markdown リンクとして提示する。

## トラブルシューティング

| 問題 | 解決策 |
|---|---|
| "No data" / 空ビュー | Trace が時間範囲外です。`startA`/`endA` を広げてください（±1h → ±1d → 90d）。 |
| 404 | ID が誤っているか base64 ではありません。ブラウザー URL から `org_id`、`space_id`、`project_id` を再確認してください。 |
| Span がハイライトされない | `span_id` が別の trace に属している可能性があります。エクスポート済み span データと照合してください。 |
| `org_id` が不明 | `ax` CLI では公開されません。`https://app.arize.com/organizations/{org_id}/spaces/{space_id}/…` からコピーしてもらってください。 |

## 関連スキル

- **arize-trace**: `trace_id`、`span_id`、`start_time` を取得するために spans をエクスポートします。

## 例

すべてのリンク種別の具体的な URL 一式は references/EXAMPLES.md を参照してください。
