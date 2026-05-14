---
name: phoenix-evals
description: Build and run evaluators for AI/LLM applications using Phoenix.
license: Apache-2.0
compatibility: Requires Phoenix server. Python skills need phoenix and openai packages; TypeScript skills need @arizeai/phoenix-client.
metadata:
  author: oss@arize.com
  version: "1.0.0"
  languages: "Python, TypeScript"
---
#フェニックス・エヴァルス

AI/LLM アプリケーション用のエバリュエーターを構築します。最初にコードを作成し、LLM でニュアンスを確認し、人間に対して検証します。

## クイックリファレンス

|タスク |ファイル |
| ---- | ----- |
|セットアップ | [setup-python](references/setup-python.md)、[setup-typescript](references/setup-typescript.md) |
|何を評価するかを決定する | [評価者-概要](参考/評価者-概要.md) |
|審査員モデルを選択する | [基礎モデルの選択](参考文献/基礎モデルの選択.md) |
|事前に構築されたエバリュエーターを使用する | [evaluators-pre-built](references/evaluators-pre-built.md) |
|ビルドコード評価ツール | [評価者コード-Python](参照/評価者コード-python.md)、[評価者コード-タイプスクリプト](参照/評価者コード-タイプスクリプト.md) |
| LLM エバリュエーターを構築する | [evaluators-llm-python](references/evaluators-llm-python.md)、[evaluators-llm-typescript](references/evaluators-llm-typescript.md)、[evaluators-custom-templates](references/evaluators-custom-templates.md) |
| DataFrame をバッチ評価する | [評価-データフレーム-python](参考/評価-データフレーム-python.md) |
|実験を実行する | [experiments-running-python](references/experiments-running-python.md)、[experiments-running-typescript](references/experiments-running-typescript.md) |
|データセットの作成 | [実験-データセット-python](参照/実験-データセット-python.md)、[実験-データセット-typescript](参照/実験-データセット-typescript.md) |
|合成データの生成 | [実験-合成-python](参考/実験-合成-python.md)、[実験-合成-typescript](参考/実験-合成-typescript.md) |
|評価者の精度を検証する | [検証](参照/validation.md)、[検証-評価者-python](参照/検証-評価者-python.md)、[検証-評価者-typescript](参照/検証-評価者-typescript.md) |
|レビュー用のサンプル トレース | [observe-sampling-python](references/observe-sampling-python.md)、[observe-sampling-typescript](references/observe-sampling-typescript.md) |
|エラーを分析する | [エラー分析](参照/エラー分析.md)、[エラー分析-マルチターン](参照/エラー分析-マルチターン.md)、[軸コーディング](参照/軸コーディング.md) |
| RAG 評価 | [evaluators-rag](references/evaluators-rag.md) |
|よくある間違いを避ける | [一般的な間違い-Python](参考/一般的な間違い-python.md)、[基本的なアンチパターン](参考/基本的なアンチパターン.md) |
|制作 | [production-overview](references/production-overview.md)、[production-guardrails](references/production-guardrails.md)、[production-continuous](references/production-continuous.md) |

## ワークフロー

**新たに始める:**
[observe-tracing-setup](references/observe-tracing-setup.md) → [error-analysis](references/error-analysis.md) → [axis-coding](references/axis-coding.md) → [evaluators-overview](references/evaluators-overview.md)**建物評価者:**
[fundamentals](references/fundamentals.md) → [common-missing-python](references/common-missing-python.md) → evaluators-{code|llm}-{python|typescript} → validation-evaluators-{python|typescript}

**RAG システム:**
[evaluators-rag](references/evaluators-rag.md) → evaluators-code-* (取得) → evaluators-llm-* (忠実度)

**制作:**
[production-overview](references/production-overview.md) → [production-guardrails](references/production-guardrails.md) → [production-continuous](references/production-continuous.md)

## 参照カテゴリ

|プレフィックス |説明 |
| ------ | ----------- |
| `fundamentals-*` |タイプ、スコア、アンチパターン |
| `observe-*` |トレース、サンプリング |
| `error-analysis-*` |失敗を見つける |
| `axial-coding-*` |失敗の分類 |
| `evaluators-*` |コード、LLM、RAG 評価者 |
| `experiments-*` |データセット、実験の実行 |
| `validation-*` |人間のラベルに対して評価者の精度を検証する |
| `production-*` | CI/CD、モニタリング |

## 重要な原則

|原則 |アクション |
| --------- | ------ |
|まずはエラー分析 |観察していないものは自動化できません |
|カスタム > 汎用 |失敗から構築する |
|最初にコードを作成 | LLM 前の決定論 |
|裁判官を検証する | >80% TPR/TNR |
|バイナリ > リッカート | 1 ～ 5 ではなく合否 |