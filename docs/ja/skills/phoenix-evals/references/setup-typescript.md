# セットアップ: TypeScript

Phoenix の評価と実験に必要なパッケージ。

## インストール「」バッシュ
# npm を使用する
npm install @arizeai/phoenix-client @arizeai/phoenix-evals @arizeai/phoenix-otel

# pnpm を使用する
pnpm add @arizeai/phoenix-client @arizeai/phoenix-evals @arizeai/phoenix-otel
「」## LLM プロバイダー

LLM-as-judge 評価者の場合は、Vercel AI SDK プロバイダーをインストールします。「」バッシュ
npm install @ai-sdk/opennai # Vercel AI SDK + OpenAI
npm install @ai-sdk/anthropic # Anthropic
npm install @ai-sdk/google # Google
「」または、直接プロバイダー SDK を使用します。「」バッシュ
npm install openai # OpenAI ダイレクト
npm install @anthropic-ai/sdk # Anthropic direct
「」## クイック検証```タイプスクリプト
import { createClient } から "@arizeai/phoenix-client";
import { createClassificationEvaluator } から "@arizeai/phoenix-evals";
import { registerPhoenix } から "@arizeai/phoenix-otel";

// すべてのインポートが機能するはずです
console.log("Phoenix TypeScript のセットアップが完了しました");
「」
