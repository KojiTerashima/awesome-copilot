---
name: phoenix-tracing
description: OpenInference semantic conventions and instrumentation for Phoenix AI observability. Use when implementing LLM tracing, creating custom spans, or deploying to production.
license: Apache-2.0
compatibility: Requires Phoenix server. Python skills need arize-phoenix-otel; TypeScript skills need @arizeai/phoenix-otel.
metadata:
  author: oss@arize.com
  version: "1.0.0"
  languages: "Python, TypeScript"
---
# フェニックス・トレース

Phoenix で OpenInference トレースを使用して LLM アプリケーションをインストルメントするための包括的なガイド。セットアップ、インスツルメンテーション、スパン タイプ、実稼働展開をカバーする参照ファイルが含まれています。

## いつ適用するか

次の場合にこれらのガイドラインを参照してください。

- Phoenix トレースのセットアップ (Python または TypeScript)
- LLM 操作用のカスタム スパンの作成
- OpenInference の規則に従って属性を追加する
- トレースを本番環境に展開する
- トレースデータのクエリと分析

## 参照カテゴリ

|優先順位 |カテゴリー |説明 |プレフィックス |
| -------- | --------------- | ------------------------------ | ------------------------ |
| 1 |セットアップ |インストールと構成 | `setup-*` |
| 2 |計装 |自動および手動トレース | `instrumentation-*` |
| 3 |スパンの種類 | 9 種類のスパンと属性 | `span-*` |
| 4 |組織 |プロジェクトとセッション | `projects-*`、`sessions-*` |
| 5 |エンリッチメント |カスタムメタデータ | `metadata-*` |
| 6 |制作 |バッチ処理、マスキング | `production-*` |
| 7 |フィードバック |注釈と評価 | `annotations-*` |

## クイックリファレンス

### 1. セットアップ (ここから始めてください)

- [setup-python](references/setup-python.md) - arize-phoenix-otelをインストールし、エンドポイントを設定します
- [setup-typescript](references/setup-typescript.md) - @arizeai/phoenix-otel をインストールし、エンドポイントを構成します

### 2. 計測器

- [instrumentation-auto-python](references/instrumentation-auto-python.md) - OpenAI、LangChain などの自動インストゥルメント
- [instrumentation-auto-typescript](references/instrumentation-auto-typescript.md) - 自動インストゥルメントがサポートするフレームワーク
- [instrumentation-manual-python](references/instrumentation-manual-python.md) - デコレータを使用したカスタム スパン
- [instrumentation-manual-typescript](references/instrumentation-manual-typescript.md) - ラッパーを使用したカスタム スパン

### 3. スパン タイプ (完全な属性スキーマを含む)

- [span-llm](references/span-llm.md) - LLM API 呼び出し (モデル、トークン、メッセージ、コスト)
- [span-chain](references/span-chain.md) - マルチステップのワークフローとパイプライン
- [span-retriever](references/span-retriever.md) - 文書検索 (文書、スコア)
- [span-tool](references/span-tool.md) - 関数/API 呼び出し (名前、パラメータ)
- [span-agent](references/span-agent.md) - 複数ステップの推論エージェント
- [span-embedding](references/span-embedding.md) - ベクトル生成
- [span-reranker](references/span-reranker.md) - ドキュメントの再ランキング
- [span-guardrail](references/span-guardrail.md) - 安全チェック
- [span-evaluator](references/span-evaluator.md) - LLM 評価

### 4. 組織- [projects-python](references/projects-python.md) / [projects-typescript](references/projects-typescript.md) - アプリケーションごとにトレースをグループ化
- [sessions-python](references/sessions-python.md) / [sessions-typescript](references/sessions-typescript.md) - 会話を追跡する

### 5. 充実

- [metadata-python](references/metadata-python.md) / [metadata-typescript](references/metadata-typescript.md) - カスタム属性

### 6. 本番環境 (重要)

- [production-python](references/production-python.md) / [production-typescript](references/production-typescript.md) - バッチ処理、PII マスキング

### 7. フィードバック

- [annotations-overview](references/annotations-overview.md) - フィードバックの概念
- [annotations-python](references/annotations-python.md) / [annotations-typescript](references/annotations-typescript.md) - スパンにフィードバックを追加します

### 参照ファイル

- [fundamentals-overview](references/fundamentals-overview.md) - トレース、スパン、属性の基本
- [fundamentals-required-attributes](references/fundamentals-required-attributes.md) - スパン タイプごとの必須フィールド
- [fundamentals-universal-attributes](references/fundamentals-universal-attributes.md) - 共通属性 (user.id、session.id)
- [fundamentals- flattening](references/fundamentals- flattening.md) - JSON フラット化ルール
- [attributes-messages](references/attributes-messages.md) - チャットメッセージのフォーマット
- [attributes-metadata](references/attributes-metadata.md) - カスタム メタデータ スキーマ
- [attributes-graph](references/attributes-graph.md) - エージェント ワークフロー属性
- [属性-例外](references/attributes-Exceptions.md) - エラー追跡

## 一般的なワークフロー

- **クイックスタート**: setup-{lang} →instrumentation-auto-{lang} → Check Phoenix
- **カスタム スパン**: setup-{lang} →instrumentation-manual-{lang} →span-{type}
- **セッション トラッキング**: 会話グループ化パターン用のsessions-{lang}
- **Production**: バッチ処理、マスキング、およびデプロイメント用のproduction-{lang}

## このスキルの使用方法

**ナビゲーション パターン:**「」バッシュ
# カテゴリプレフィックス別
references/setup-* # インストールと設定
References/instrumentation-* # 自動および手動トレース
References/span-* # スパンタイプの仕様
References/sessions-* # セッション追跡
References/production-* # 実稼働デプロイメント
参考/基礎-* # 中心となる概念
References/attributes-* # 属性の仕様

# 言語別
References/*-python.md # Python 実装
References/*-typescript.md # TypeScript の実装
「」**読む順序:**
1. 使用する言語の setup-{lang} から始めます
2.instrumentation-auto-{lang} またはinstrumentation-manual-{lang}を選択します。
3. 特定の操作に必要な、span-{type} ファイルの参照
4. 属性の仕様については、fundamentals-* ファイルを参照してください。

## 参考文献

**Phoenix ドキュメント:**

- [Phoenix ドキュメント](https://docs.arize.com/phoenix)
- [OpenInference 仕様](https://github.com/Arize-ai/openinference/tree/main/spec)

**Python API ドキュメント:**

- [Python OTEL パッケージ](https://arize-phoenix.readthedocs.io/projects/otel/en/latest/) - `arize-phoenix-otel` API リファレンス
- [Python クライアント パッケージ](https://arize-phoenix.readthedocs.io/projects/client/en/latest/) - `arize-phoenix-client` API リファレンス

**TypeScript API ドキュメント:**

- [TypeScript パッケージ](https://arize-ai.github.io/phoenix/) - `@arizeai/phoenix-otel`、`@arizeai/phoenix-client`、およびその他の TypeScript パッケージ