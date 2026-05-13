# Arize Link の例

このドキュメント全体で使うプレースホルダー:
- `{org_id}` — base64 エンコード済み org ID
- `{space_id}` — base64 エンコード済み space ID
- `{project_id}` — base64 エンコード済み project ID
- `{start_ms}` / `{end_ms}` — エポックミリ秒（例: 1741305600000 / 1741392000000）

---

## Trace

```
https://app.arize.com/organizations/{org_id}/spaces/{space_id}/projects/{project_id}?selectedTraceId={trace_id}&queryFilterA=&selectedTab=llmTracing&timeZoneA=America%2FLos_Angeles&startA={start_ms}&endA={end_ms}&envA=tracing&modelType=generative_llm
```

## Span（trace + span をハイライト）

```
https://app.arize.com/organizations/{org_id}/spaces/{space_id}/projects/{project_id}?selectedTraceId={trace_id}&selectedSpanId={span_id}&queryFilterA=&selectedTab=llmTracing&timeZoneA=America%2FLos_Angeles&startA={start_ms}&endA={end_ms}&envA=tracing&modelType=generative_llm
```

## Session

```
https://app.arize.com/organizations/{org_id}/spaces/{space_id}/projects/{project_id}?selectedSessionId={session_id}&queryFilterA=&selectedTab=llmTracing&timeZoneA=America%2FLos_Angeles&startA={start_ms}&endA={end_ms}&envA=tracing&modelType=generative_llm
```

## Dataset（examples タブ）

```
https://app.arize.com/organizations/{org_id}/spaces/{space_id}/datasets/{dataset_id}?selectedTab=examples
```

## Dataset（experiments タブ）

```
https://app.arize.com/organizations/{org_id}/spaces/{space_id}/datasets/{dataset_id}?selectedTab=experiments
```

## Labeling Queue 一覧

```
https://app.arize.com/organizations/{org_id}/spaces/{space_id}/queues
```

## Labeling Queue（特定）

```
https://app.arize.com/organizations/{org_id}/spaces/{space_id}/queues/{queue_id}
```

## Evaluator（最新バージョン）

```
https://app.arize.com/organizations/{org_id}/spaces/{space_id}/evaluators/{evaluator_id}
```

## Evaluator（特定バージョン）

```
https://app.arize.com/organizations/{org_id}/spaces/{space_id}/evaluators/{evaluator_id}?version={version_url_encoded}
```

## Annotation Configs

```
https://app.arize.com/organizations/{org_id}/spaces/{space_id}/annotation-configs
```
