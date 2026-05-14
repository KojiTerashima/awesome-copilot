---
name: arize-annotation
description: "Arize で annotation config（categorical、continuous、freeform）を作成・管理・利用する場合や、Python SDK でプロジェクトの span に人手アノテーションを適用する場合は、このスキルを **INVOKE THIS SKILL** してください。Config は、Arize UI 上の span やその他の対象に対する人手フィードバックのラベルスキーマです。トリガー: annotation config, label schema, human feedback schema, bulk annotate spans, update_annotations."
---

# Arize Annotation Skill

このスキルは、**annotation config**（人手フィードバックのスキーマ）と、Python SDK を使った**プロジェクト span へのプログラム的なアノテーション付与**に焦点を当てています。Arize UI での人手レビュー（annotation queue、dataset、experiment を含む）も引き続きこれらの config に依存します。queue 用の `ax` CLI はまだありません。

**方向性:** Arize における人手ラベリングは、config で定義された値をプロダクト UI 上の **span**、**dataset example**、**experiment 関連レコード**、**queue item** に紐づけます。ここでドキュメント化している内容は、`ax annotation-configs` と `ArizeClient.spans.update_annotations` による span の一括更新です。

---

## 前提条件

そのままタスクを進め、必要な `ax` コマンドを実行してください。最初にバージョン、環境変数、プロファイルを確認しないでください。

`ax` コマンドが失敗した場合は、エラーに応じて対処してください。
- `command not found` またはバージョンエラー → references/ax-setup.md を参照
- `401 Unauthorized` / API キー不足 → `ax profiles show` を実行して現在のプロファイルを確認。プロファイルがない、または API キーが誤っている場合: `.env` の `ARIZE_API_KEY` を確認し、references/ax-profiles.md に従ってそのキーでプロファイルを作成/更新してください。`.env` にもキーがない場合は、ユーザーに Arize API キーの提供を依頼してください（https://app.arize.com/admin > API Keys）
- Space ID が不明 → `.env` の `ARIZE_SPACE_ID` を確認するか、`ax spaces list -o json` を実行するか、ユーザーに確認

---

## 概念

### Annotation Config とは？

**annotation config** は、1 種類の人手フィードバックラベルのスキーマを定義します。span、dataset record、experiment output、queue item にアノテーションする前に、そのラベルに対応する config が space 内に存在している必要があります。

| Field | Description |
|-------|-------------|
| **Name** | 説明的な識別子（例: `Correctness`、`Helpfulness`）。space 内で一意である必要があります。 |
| **Type** | `categorical`（リストから選択）、`continuous`（数値範囲）、`freeform`（自由記述テキスト）。 |
| **Values** | categorical の場合: `{"label": str, "score": number}` のペア配列。 |
| **Min/Max Score** | continuous の場合: 数値の下限/上限。 |
| **Optimization Direction** | 高スコアが良い (`maximize`) か悪い (`minimize`) か。UI の傾向表示に使われます。 |

### ラベルが適用される場所（surfaces）

| Surface | Typical path |
|---------|----------------|
| **Project spans** | Python SDK `spans.update_annotations`（下記）および/または Arize UI |
| **Dataset examples** | Arize UI（人手ラベリングフロー）; config は space に存在している必要があります |
| **Experiment outputs** | UI 上で dataset や trace と併せてレビューされることが多い — arize-experiment、arize-dataset を参照 |
| **Annotation queue items** | Arize UI; config は必須 — ここでは `ax` queue コマンドはまだ記載していません |

ラベルを永続化できるようにする前に、関連する **annotation config** が space に存在することを常に確認してください。

---

## 基本 CRUD: Annotation Configs

### 一覧

```bash
ax annotation-configs list --space-id SPACE_ID
ax annotation-configs list --space-id SPACE_ID -o json
ax annotation-configs list --space-id SPACE_ID --limit 20
```

### 作成 — Categorical

Categorical config は、レビュアーが選択する固定ラベル集合を提示します。

```bash
ax annotation-configs create \
  --name "Correctness" \
  --space-id SPACE_ID \
  --type categorical \
  --values '[{"label": "correct", "score": 1}, {"label": "incorrect", "score": 0}]' \
  --optimization-direction maximize
```

よく使う 2 値ラベルの組み合わせ:
- `correct` / `incorrect`
- `helpful` / `unhelpful`
- `safe` / `unsafe`
- `relevant` / `irrelevant`
- `pass` / `fail`

### 作成 — Continuous

Continuous config は、定義された範囲内の数値スコア入力をレビュアーに許可します。

```bash
ax annotation-configs create \
  --name "Quality Score" \
  --space-id SPACE_ID \
  --type continuous \
  --minimum-score 0 \
  --maximum-score 10 \
  --optimization-direction maximize
```

### 作成 — Freeform

Freeform config は自由記述テキストのフィードバックを収集します。name、space、type 以外の追加フラグは不要です。

```bash
ax annotation-configs create \
  --name "Reviewer Notes" \
  --space-id SPACE_ID \
  --type freeform
```

### 取得

```bash
ax annotation-configs get ANNOTATION_CONFIG_ID
ax annotation-configs get ANNOTATION_CONFIG_ID -o json
```

### 削除

```bash
ax annotation-configs delete ANNOTATION_CONFIG_ID
ax annotation-configs delete ANNOTATION_CONFIG_ID --force   # 確認をスキップ
```

**Note:** 削除は取り消せません。この config に紐づく annotation queue の関連付けもプロダクト上で削除されます（queue 自体は残る可能性があります。必要に応じて Arize UI で関連付けを修正してください）。

---

## Spans へのアノテーション適用（Python SDK）

すでにラベルを持っている場合（例: レビュー結果のエクスポートや外部ラベリングツール）、Python SDK を使って**プロジェクト span**へアノテーションを一括適用します。

```python
import pandas as pd
from arize import ArizeClient

import os

client = ArizeClient(api_key=os.environ["ARIZE_API_KEY"])

# アノテーション列を含む DataFrame を作成
# 必須: context.span_id + 少なくとも1つの annotation.<name>.label または annotation.<name>.score
annotations_df = pd.DataFrame([
    {
        "context.span_id": "span_001",
        "annotation.Correctness.label": "correct",
        "annotation.Correctness.updated_by": "reviewer@example.com",
    },
    {
        "context.span_id": "span_002",
        "annotation.Correctness.label": "incorrect",
        "annotation.Correctness.updated_by": "reviewer@example.com",
    },
])

response = client.spans.update_annotations(
    space_id=os.environ["ARIZE_SPACE_ID"],
    project_name="your-project",
    dataframe=annotations_df,
    validate=True,
)
```

**DataFrame 列スキーマ:**

| Column | Required | Description |
|--------|----------|-------------|
| `context.span_id` | yes | アノテーション対象の span |
| `annotation.<name>.label` | one of | Categorical または freeform のラベル |
| `annotation.<name>.score` | one of | 数値スコア |
| `annotation.<name>.updated_by` | no | アノテーター識別子（メールまたは名前） |
| `annotation.<name>.updated_at` | no | Unix epoch からのミリ秒タイムスタンプ |
| `annotation.notes` | no | span に対する自由記述メモ |

**制限:** アノテーションは、送信時点から過去 31 日以内の span にのみ適用されます。

---

## トラブルシューティング

| Problem | Solution |
|---------|----------|
| `ax: command not found` | references/ax-setup.md を参照 |
| `401 Unauthorized` | API キーにこの space へのアクセス権がない可能性があります。https://app.arize.com/admin > API Keys で確認 |
| `Annotation config not found` | `ax annotation-configs list --space-id SPACE_ID` |
| `409 Conflict on create` | 名前が space 内で既に存在します。別名を使うか既存 config ID を取得してください。 |
| UI での人手レビュー / queue | Arize アプリを使用し、config が存在することを確認してください — `ax` の annotation-queue CLI はまだありません |
| Span SDK エラーまたは span 不足 | `project_name`、`space_id`、span ID を確認し、arize-trace で span をエクスポートしてください |

---

## 関連スキル

- **arize-trace**: span ID と時間範囲を特定するために span をエクスポート
- **arize-dataset**: dataset ID と example ID を特定
- **arize-evaluator**: 人手アノテーションと並行する LLM-as-judge の自動評価
- **arize-experiment**: dataset と評価ワークフローに紐づく experiment
- **arize-link**: Arize UI の annotation config や queue へのディープリンク

---

## 今後の利用に向けた認証情報の保存

references/ax-profiles.md の「Save Credentials for Future Use」を参照してください。

