# TypeScript のセットアップ

`@arizeai/phoenix-otel` を使用して、TypeScript/JavaScript で Phoenix トレースをセットアップします。

## メタデータ

|属性 |値 |
|----------|----------|
|優先順位 |クリティカル - すべてのトレースに必要 |
|セットアップ時間 | 5 分未満 |

## クイックスタート「」バッシュ
npm インストール @arizeai/phoenix-otel
「」

```タイプスクリプト
import { register } from "@arizeai/phoenix-otel";
register({ プロジェクト名: "my-app" });
「」デフォルトでは `http://localhost:6006` に接続します。

＃＃ 構成```タイプスクリプト
import { register } from "@arizeai/phoenix-otel";

登録({
  プロジェクト名: "私のアプリ",
  URL: "http://localhost:6006",
  apiKey: process.env.PHOENIX_API_KEY、
  バッチ: true
});
「」**環境変数:**「」バッシュ
エクスポート PHOENIX_API_KEY="あなたの API キー"
エクスポート PHOENIX_COLLECTOR_ENDPOINT="http://localhost:6006"
エクスポート PHOENIX_PROJECT_NAME="my-app"
「」## ESM と CommonJS の比較

**CommonJS (自動):**```JavaScript
const { register } = require("@arizeai/phoenix-otel");
register({ プロジェクト名: "my-app" });

const OpenAI = require("openai");
「」**ESM (手動計測が必要):**```タイプスクリプト
import { register, registerInstrumentations } from "@arizeai/phoenix-otel";
import { OpenAIInstrumentation } from "@arizeai/openinference-instrumentation-openai";
「openai」から OpenAI をインポートします。

register({ プロジェクト名: "my-app" });

const インストルメンテーション = new OpenAIInstrumentation();
インストルメンテーション.manuallyInstrument(OpenAI);
registerInstrumentations({ インストルメンテーション: [インストルメンテーション] });
「」**理由:** ESM インポートはホイストされるため、`manuallyInstrument()` が必要です。

## フレームワークの統合

**Next.js (アプリルーター):**```タイプスクリプト
// インストルメンテーション.ts
非同期関数のエクスポート register() {
  if (process.env.NEXT_RUNTIME === "nodejs") {
    const { register } = await import("@arizeai/phoenix-otel");
    register({ プロジェクト名: "my-nextjs-app" });
  }
}
「」**Express.js:**```タイプスクリプト
import { register } from "@arizeai/phoenix-otel";

register({ プロジェクト名: "my-express-app" });

const app = Express();
「」## 終了前のフラッシュ スパン

**重要:** プロセスの終了時にプロセッサー内でキューに残っている場合、スパンはエクスポートされない可能性があります。 `provider.shutdown()` を呼び出して、終了する前に明示的にフラッシュします。

**標準パターン:**```タイプスクリプト
const プロバイダー = register({
  プロジェクト名: "私のアプリ",
  バッチ: true、
});

非同期関数 main() {
  doWork() を待ちます;
  プロバイダーを待ちます.shutdown();  // 終了前にスパンをフラッシュします
}

main().catch(async (エラー) => {
  コンソール.エラー(エラー);
  プロバイダーを待ちます.shutdown();  // エラー時もフラッシュ
  プロセス終了(1);
});
「」**代替：**```タイプスクリプト
// 即時エクスポートにはバッチ: false を使用します (シャットダウンは必要ありません)
登録({
  プロジェクト名: "私のアプリ",
  バッチ: false、
});
「」正常な終了を含む運用パターンについては、`production-typescript.md` を参照してください。

## 検証

1. Phoenix UI を開きます: `http://localhost:6006`
2. アプリケーションを実行します
3. プロジェクト内のトレースを確認します。

**診断ログを有効にする:**```タイプスクリプト
import { DiagLogLevel, register } from "@arizeai/phoenix-otel";

登録({
  プロジェクト名: "私のアプリ",
  diagLogLevel: DiagLogLevel.DEBUG、
});
「」## トラブルシューティング

**痕跡なし:**
- `PHOENIX_COLLECTOR_ENDPOINT` が正しいことを確認してください
- Phoenix Cloud に `PHOENIX_API_KEY` を設定します
- ESM の場合: `manuallyInstrument()` が呼び出されることを確認します。
- **`batch: true` の場合:** 終了前に `provider.shutdown()` を呼び出して、キューに入れられたスパンをフラッシュします (「スパンのフラッシュ」セクションを参照)

**痕跡がありません:**
- `batch: true` の場合: プロセスが終了する前に `await provider.shutdown()` を呼び出し、キューに入れられたスパンをフラッシュします
- 代替案: `batch: false` を即時エクスポートに設定します (シャットダウンは必要ありません)。

**属性がありません:**
- インストルメンテーションが登録されていることを確認します (ESM は手動セットアップが必要です)
- `instrumentation-auto-typescript.md`を参照

## 関連項目

- **自動インストルメンテーション:** `instrumentation-auto-typescript.md`
- **手動計測:** `instrumentation-manual-typescript.md`
- **API ドキュメント:** https://arize-ai.github.io/phoenix/