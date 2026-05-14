# 制作：概要

CI/CD 評価と実稼働モニタリング - 補完的なアプローチ。

## 2 つの評価モード

|側面 | CI/CD 評価 |生産監視 |
| ------ | ----------- | -------------------- |
| **いつ** |導入前 |導入後、進行中 |
| **データ** |固定データセット |サンプリングされたトラフィック |
| **目標** |回帰を防ぐ |ドリフトを検出 |
| **応答** |デプロイをブロックする |アラートと分析 |

## CI/CD の評価「」パイソン
# 高速かつ決定的なチェック
ci_evaluators = [
    has_required_format、
    no_pii_leak、
    安全性チェック、
    regression_test_suite、
】

# 小さいながらも代表的なデータセット (約 100 個の例)
run_experiment(ci_dataset, タスク, ci_evaluators)
「」しきい値を設定します: 回帰 = 0.95、安全性 = 1.0、形式 = 0.98。

## 生産監視

### パイソン「」パイソン
phoenix.clientインポートクライアントから
from datetime import datetime、timedelta

client = クライアント()

# 最近のトレースのサンプル (過去 1 時間)
トレース = client.traces.get_traces(
    project_identifier="私のアプリ",
    start_time=datetime.now() - timedelta(時間=1)、
    include_spans=True、
    制限=100、
）

# サンプリングされたトラフィックに対してエバリュエーターを実行する
トレース内のトレースの場合:
    results = run_evaluators_async(trace,production_evaluators)
    ある場合(r["score"] < 0.5 結果の r ):
        alert_on_failure(トレース、結果)
「」### TypeScript```タイプスクリプト
import { getTraces } から "@arizeai/phoenix-client/traces";
import { getSpans } から "@arizeai/phoenix-client/spans";

// 最近のトレースのサンプル (過去 1 時間)
const { トレース } = await getTraces({
  プロジェクト: { プロジェクト名: "my-app" },
  startTime: 新しい日付(Date.now() - 60 * 60 * 1000)、
  includeSpans: true、
  制限: 100、
});

// または、評価のために直接サンプル スパンを使用することもできます
const { スパン } = await getSpans({
  プロジェクト: { プロジェクト名: "my-app" },
  startTime: 新しい日付(Date.now() - 60 * 60 * 1000)、
  制限: 100、
});

// サンプリングされたトラフィックに対してエバリュエーターを実行します
for (スパンの定数スパン) {
  const results = await runEvaluators(span,productionEvaluators);
  if (results.some((r) => r.score < 0.5)) {
    awaitalertOnFailure(スパン, 結果);
  }
}
「」優先順位付け: エラー → 負のフィードバック → ランダム サンプル。

## フィードバック ループ「」
本番環境で障害が検出される → エラー分析 → CI データセットに追加 → 将来の回帰を防止
「」
