---
name: datanalysis-credit-risk
description: '融資前モデリング向けの信用リスク データのクリーニングと変数スクリーニングのパイプラインです。品質評価、欠損値分析、モデリング前の変数選定が必要な生の信用データを扱うときに使用します。データの読み込みと整形、異常期間の除外、欠損率の計算、高欠損変数の削除、低 IV 変数の除外、高 PSI 変数の削除、Null Importance によるノイズ除去、高相関変数の削除、クリーニング レポートの生成を扱います。適用シナリオは、信用リスク データのクリーニング、変数スクリーニング、融資前モデリング前処理です。'
---

# Data Cleaning and Variable Screening

## Quick Start

```bash
# 完全なデータ クリーニング パイプラインを実行する
python ".github/skills/datanalysis-credit-risk/scripts/example.py"
```

## Complete Process Description

データ クリーニング パイプラインは次の 11 ステップで構成され、各ステップは元データを削除せず独立して実行されます。

1. **Get Data** - 生データを読み込んで整形する
2. **Organization Sample Analysis** - 各機関のサンプル数と不良サンプル率を集計する
3. **Separate OOS Data** - out-of-sample (OOS) サンプルをモデリング用サンプルから分離する
4. **Filter Abnormal Months** - 不良サンプル数または総サンプル数が不足する月を除外する
5. **Calculate Missing Rate** - 各特徴量の全体および機関別の欠損率を計算する
6. **Drop High Missing Rate Features** - 全体欠損率が閾値を超える特徴量を除外する
7. **Drop Low IV Features** - 全体 IV が低すぎる、または多くの機関で IV が低すぎる特徴量を除外する
8. **Drop High PSI Features** - PSI が不安定な特徴量を除外する
9. **Null Importance Denoising** - ラベル置換法を用いてノイズ特徴量を除外する
10. **Drop High Correlation Features** - 元の gain に基づいて高相関の特徴量を除外する
11. **Export Report** - すべてのステップの詳細と統計を含む Excel レポートを生成する

## Core Functions

| Function | Purpose | Module |
|------|------|----------|
| `get_dataset()` | データの読み込みと整形 | references.func |
| `org_analysis()` | 機関別サンプル分析 | references.func |
| `missing_check()` | 欠損率の計算 | references.func |
| `drop_abnormal_ym()` | 異常月の除外 | references.analysis |
| `drop_highmiss_features()` | 高欠損率の特徴量を除外 | references.analysis |
| `drop_lowiv_features()` | 低 IV の特徴量を除外 | references.analysis |
| `drop_highpsi_features()` | 高 PSI の特徴量を除外 | references.analysis |
| `drop_highnoise_features()` | Null Importance によるノイズ除去 | references.analysis |
| `drop_highcorr_features()` | 高相関の特徴量を除外 | references.analysis |
| `iv_distribution_by_org()` | 機関別 IV 分布統計 | references.analysis |
| `psi_distribution_by_org()` | 機関別 PSI 分布統計 | references.analysis |
| `value_ratio_distribution_by_org()` | 機関別 value ratio 分布統計 | references.analysis |
| `export_cleaning_report()` | クリーニング レポートを出力 | references.analysis |

## Parameter Description

### Data Loading Parameters
- `DATA_PATH`: データ ファイル パス (推奨は parquet 形式)
- `DATE_COL`: 日付列名
- `Y_COL`: ラベル列名
- `ORG_COL`: 機関列名
- `KEY_COLS`: 主キー列名のリスト

### OOS Organization Configuration
- `OOS_ORGS`: out-of-sample 機関のリスト

### Abnormal Month Filtering Parameters
- `min_ym_bad_sample`: 月ごとの最小不良サンプル数 (default 10)
- `min_ym_sample`: 月ごとの最小総サンプル数 (default 500)

### Missing Rate Parameters
- `missing_ratio`: 全体欠損率の閾値 (default 0.6)

### IV Parameters
- `overall_iv_threshold`: 全体 IV の閾値 (default 0.1)
- `org_iv_threshold`: 単一機関 IV の閾値 (default 0.1)
- `max_org_threshold`: 許容される低 IV 機関数の上限 (default 2)

### PSI Parameters
- `psi_threshold`: PSI の閾値 (default 0.1)
- `max_months_ratio`: 不安定な月の最大比率 (default 1/3)
- `max_orgs`: 不安定な機関数の上限 (default 6)

### Null Importance Parameters
- `n_estimators`: 木の本数 (default 100)
- `max_depth`: 木の最大深さ (default 5)
- `gain_threshold`: Gain 差分の閾値 (default 50)

### High Correlation Parameters
- `max_corr`: 相関係数の閾値 (default 0.9)
- `top_n_keep`: 元の gain 順位で上位 N 個を保持 (default 20)

## Output Report

生成される Excel レポートには次のシートが含まれます。

1. **汇总** - 全ステップの要約情報。操作結果と条件を含む
2. **机构样本统计** - 各機関のサンプル数と不良サンプル率
3. **分离OOS数据** - OOS サンプル数とモデリング サンプル数
4. **Step4-异常月份处理** - 除外された異常月
5. **缺失率明细** - 各特徴量の全体および機関別欠損率
6. **Step5-有值率分布统计** - 値率レンジごとの特徴量分布
7. **Step6-高缺失率处理** - 除外された高欠損率特徴量
8. **Step7-IV明细** - 各機関および全体における各特徴量の IV 値
9. **Step7-IV处理** - IV 条件を満たさない特徴量と低 IV の機関
10. **Step7-IV分布统计** - IV レンジごとの特徴量分布
11. **Step8-PSI明细** - 各機関・各月における各特徴量の PSI 値
12. **Step8-PSI处理** - PSI 条件を満たさない特徴量と不安定な機関
13. **Step8-PSI分布统计** - PSI レンジごとの特徴量分布
14. **Step9-null importance处理** - 除外されたノイズ特徴量
15. **Step10-高相关性剔除** - 除外された高相関特徴量

## Features

- **Interactive Input**: 各ステップ実行前にパラメーターを入力でき、既定値も利用可能
- **Independent Execution**: 各ステップは元データを削除せず独立して実行され、比較分析を容易にする
- **Complete Report**: 詳細、統計、分布を含む完全な Excel レポートを生成する
- **Multi-process Support**: IV と PSI の計算はマルチプロセスによる高速化をサポートする
- **Organization-level Analysis**: 機関レベルの統計と modeling/OOS の区別をサポートする
