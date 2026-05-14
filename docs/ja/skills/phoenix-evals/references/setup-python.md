# セットアップ: Python

Phoenix の評価と実験に必要なパッケージ。

## インストール「」バッシュ
# コア Phoenix パッケージ (client、evals、otel を含む)
pip インストール arise-phoenix

# または個別のパッケージをインストールする
pip install arize-phoenix-client # Phoenix クライアントのみ
pip install arize-phoenix-evals # 評価ユーティリティ
pip install arize-phoenix-otel # OpenTelemetry 統合
「」## LLM プロバイダー

LLM-as-judge 評価者の場合は、プロバイダーの SDK をインストールします。「」バッシュ
pip インストール openai # OpenAI
pip install anthropic # Anthropic
pip install google-generativeai # Google
「」## 検証 (オプション)「」バッシュ
pip install scikit-learn # TPR/TNR メトリクスの場合
「」## クイック検証「」パイソン
phoenix.clientインポートクライアントから
phoenix.evals から LLM、ClassificationEvaluator をインポート
phoenix.otelインポートレジスタから

# すべてのインポートが機能するはずです
print("Phoenix Python のセットアップが完了しました")
「」## キーのインポート (Evals 2.0)「」パイソン
phoenix.clientインポートクライアントから
phoenix.evals インポートから (
    ClassificationEvaluator、# LLM 分類エバリュエーター (推奨)
    LLM、# プロバイダーに依存しない LLM ラッパー
    async_evaluate_dataframe, # DataFrame をバッチ評価します (推奨、非同期)
    Evaluate_dataframe, # DataFrame をバッチ評価する (同期)
    create_evaluator, # コードベースのエバリュエーター用のデコレーター
    create_classifier, # LLM 分類評価器のファクトリー
    bind_evaluator, # 列名を評価パラメータにマップする
    スコア、# スコア データクラス
）
from phoenix.evals.utils import to_annotation_dataframe # Phoenix アノテーションの結果をフォーマットする
「」**優先**: `create_classifier` よりも `ClassificationEvaluator` (より多くのパラメーター/カスタマイズ)。
**推奨**: `evaluate_dataframe` よりも `async_evaluate_dataframe` (LLM 評価のスループットが向上します)。

**レガシー 1.0 インポートは使用しないでください**: `OpenAIModel`、`AnthropicModel`、`run_evals`、`llm_classify`。