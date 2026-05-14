# Phoenix Tracing: 制作ガイド (TypeScript)

**重要: 本番展開用にバッチ処理、データ マスキング、およびスパン フィルタリングを構成します。**

## メタデータ

|属性 |値 |
|----------|----------|
|優先順位 |重要 - 本番環境の準備 |
|影響 |セキュリティ、パフォーマンス |
|セットアップ時間 | 5～15分 |

## バッチ処理

**バッチ処理を有効にして生産効率を高めます。** バッチ処理では、スパンを個別に送信するのではなくグループで送信することで、ネットワークのオーバーヘッドを削減します。```タイプスクリプト
import { register } from "@arizeai/phoenix-otel";

const プロバイダー = register({
  プロジェクト名: "私のアプリ",
  バッチ: true, // 本番環境のデフォルト
});
「」### シャットダウン処理

**重要:** プロセスの終了時にプロセッサー内でキューに残っている場合、スパンはエクスポートされない可能性があります。 `provider.shutdown()` を呼び出して、終了する前に明示的にフラッシュします。```タイプスクリプト
// キューに入れられたスパンをフラッシュするための明示的なシャットダウン
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
「」**正常な終了シグナル:**```タイプスクリプト
// SIGTERM での正常なシャットダウン
const プロバイダー = register({
  プロジェクト名: "私のサーバー",
  バッチ: true、
});

process.on("SIGTERM", async () => {
  プロバイダーを待ちます.shutdown();
  プロセス終了(0);
});
「」---

## データマスキング (PII 保護)

**環境変数:**「」バッシュ
import OPENINFERENCE_HIDE_INPUTS=true # input.value を非表示にする
import OPENINFERENCE_HIDE_OUTPUTS=true # 出力値を非表示にする
import OPENINFERENCE_HIDE_INPUT_MESSAGES=true # LLM 入力メッセージを非表示にする
import OPENINFERENCE_HIDE_OUTPUT_MESSAGES=true # LLM 出力メッセージを非表示にする
import OPENINFERENCE_HIDE_INPUT_IMAGES=true # 画像コンテンツを非表示にする
import OPENINFERENCE_HIDE_INPUT_TEXT=true # 埋め込みテキストを非表示にする
import OPENINFERENCE_BASE64_IMAGE_MAX_LENGTH=10000 # 画像サイズを制限する
「」**TypeScript TraceConfig:**```タイプスクリプト
import { register } from "@arizeai/phoenix-otel";
import { OpenAIInstrumentation } from "@arizeai/openinference-instrumentation-openai";

const traceConfig = {
  入力を隠す: true、
  出力を隠す: true、
  HideInputMessages: true
};

const計測 = new OpenAIInstrumentation({traceConfig});
「」**優先順位:** コード > 環境変数 > デフォルト

---

## スパンフィルタリング

**特定のコード ブロックを抑制します:**```タイプスクリプト
「@opentelemetry/core」からインポート {suppressTracing};
import { context } から "@opentelemetry/api";

await context.with(suppressTracing(context.active()), async () => {
  内部ログ(); // スパンは生成されません
});
「」**サンプリング：**「」バッシュ
エクスポート OTEL_TRACES_SAMPLER="parentbased_traceidratio"
エクスポート OTEL_TRACES_SAMPLER_ARG="0.1" # サンプル 10%
「」---

## エラー処理```タイプスクリプト
"@opentelemetry/api" から { SpanStatusCode } をインポートします。

{を試してください
  結果 =riskyOperation() を待ちます;
  scan?.setStatus({ コード: SpanStatusCode.OK });
} キャッチ (e) {
  スパン?.recordException(e);
  scan?.setStatus({ コード: SpanStatusCode.ERROR });
  eを投げます。
}
「」---

## 制作チェックリスト

- [ ] バッチ処理が有効です
- [ ] **シャットダウン処理:** キューに入れられたスパンをフラッシュするには、終了する前に `provider.shutdown()` を呼び出します。
- [ ] **正常な終了:** SIGTERM/SIGINT シグナルのフラッシュ スパン
- [ ] データマスキング設定済み (PII の場合は `HIDE_INPUTS`/`HIDE_OUTPUTS`)
- [ ] ヘルスチェック/ノイズの多いパスのスパンフィルタリング
- [ ] エラー処理が実装されました
- [ ] Phoenix が使用できない場合の正常な機能低下
- [ ] パフォーマンステスト済み
- [ ] モニタリングが設定されています (Phoenix UI がチェックされています)