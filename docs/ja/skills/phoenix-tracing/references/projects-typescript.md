# Phoenix Tracing: プロジェクト (TypeScript)

**プロジェクトを使用してアプリケーションごとにトレースを整理します (Phoenix の最上位グループ)。**

## 概要

プロジェクトは、単一のアプリケーションまたは実験のトレースをグループ化します。

**用途:** 環境 (開発/ステージング/本番)、A/B テスト、バージョン管理

## セットアップ

### 環境変数 (推奨)「」バッシュ
エクスポート PHOENIX_PROJECT_NAME="my-app-prod"
「」

```タイプスクリプト
process.env.PHOENIX_PROJECT_NAME = "my-app-prod";
import { register } from "@arizeai/phoenix-otel";
登録する（）;  // 「my-app-prod」を使用します
「」### コード```タイプスクリプト
import { register } from "@arizeai/phoenix-otel";
register({ プロジェクト名: "my-app-prod" });
「」## 使用例

**環境:**```タイプスクリプト
// 開発、ステージング、本番
register({ プロジェクト名: "my-app-dev" });
register({ プロジェクト名: "my-app-staging" });
register({ プロジェクト名: "my-app-prod" });
「」**A/B テスト:**```タイプスクリプト
// モデルを比較する
register({ プロジェクト名: "chatbot-gpt4" });
register({ プロジェクト名: "チャットボット クロード" });
「」**バージョン管理:**```タイプスクリプト
// バージョンを追跡する
register({ プロジェクト名: "my-app-v1" });
register({ プロジェクト名: "my-app-v2" });
「」
