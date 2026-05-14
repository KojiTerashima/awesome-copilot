# Phoenix トレーシング: プロジェクト (Python)

**プロジェクトを使用してアプリケーションごとにトレースを整理します (Phoenix の最上位グループ)。**

## 概要

プロジェクトは、単一のアプリケーションまたは実験のトレースをグループ化します。

**用途:** 環境 (開発/ステージング/本番)、A/B テスト、バージョン管理

## セットアップ

### 環境変数 (推奨)「」バッシュ
エクスポート PHOENIX_PROJECT_NAME="my-app-prod"
「」

「」パイソン
OSをインポートする
os.environ["PHOENIX_PROJECT_NAME"] = "my-app-prod"
phoenix.otelインポートレジスタから
register() # 「my-app-prod」を使用します
「」### コード「」パイソン
phoenix.otelインポートレジスタから
register(project_name="my-app-prod")
「」## 使用例

**環境:**「」パイソン
# 開発、ステージング、本番
register(project_name="my-app-dev")
register(project_name="my-app-staging")
register(project_name="my-app-prod")
「」**A/B テスト:**「」パイソン
# モデルを比較する
register(プロジェクト名="チャットボット-gpt4")
register(project_name="チャットボット クロード")
「」**バージョン管理:**「」パイソン
# バージョンを追跡する
register(project_name="my-app-v1")
register(project_name="my-app-v2")
「」## プロジェクトの切り替え (Python ノートブックのみ)「」パイソン
openinference.instrumentation から、dangerly_using_project をインポート
phoenix.otelインポートレジスタから

register(project_name="my-app")

# eval のために一時的に切り替える
危険なほど_using_project("my-eval-project") を使用:
    run_evaluations()
「」**⚠️ 運用環境ではなく、ノートブック/スクリプトでのみ使用してください。**