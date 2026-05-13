---
description: 'R 言語とドキュメント形式 (R、Rmd、Quarto) に関する、慣用的で安全かつ一貫したコード生成のためのコーディング標準と Copilot ガイダンス。'
applyTo: '**/*.R, **/*.r, **/*.Rmd, **/*.rmd, **/*.qmd'
---

# R プログラミング言語 instruction

## 目的

GitHub Copilot が project 全体で慣用的、安全、かつ保守しやすい R code を生成できるよう支援します。

## コア規約

- **project の style に合わせる。** file に好み (tidyverse か base R か、`%>%` か `|>` か) が見えるなら、それに従います。
- **明確で vectorized な code を優先する。** 関数は小さく保ち、隠れた副作用を避けます。
- **例や snippet では base 以外の関数を修飾する**。たとえば `dplyr::mutate()`、`stringr::str_detect()`。project code では、それが repo の標準なら `library()` の使用も許容されます。
- **命名:** object / file には `lower_snake_case` を使い、名前に dot は避けます。
- **副作用:** `setwd()` は決して呼ばず、project 相対 path (例: `here::here()`) を優先します。
- **再現性:** 確率的操作の周辺では、`withr::with_seed()` を使って局所的に seed を設定します。
- **検証:** ユーザー入力は検証して制約し、可能なら型付きチェックや allowlist を使います。
- **安全性:** `eval(parse())`、未検証の shell call、parameter 化されていない SQL を避けます。

### Pipe Operators

- **Native pipe `|>` (R ≥ 4.1.0):** R ≥ 4.1 ではこれを優先します (追加依存なし)。
- **Magrittr pipe `%>%`:** すでに magrittr に寄せている project、または `.`、`%T>%`、`%$%` のような機能が必要な場合は使い続けます。
- **一貫性を保つ:** 明確な技術的理由がない限り、同じ script 内で `|>` と `%>%` を混在させません。

## パフォーマンスに関する考慮事項

- **大規模データセット:** `data.table` を検討し、実際の workload で benchmark を取ります。
- **dplyr 互換性:** `dtplyr` を使うと、dplyr 構文を data.table 操作へ自動変換し、パフォーマンスを向上できます。
- **Profiling:** `profvis::profvis()` を使って code のパフォーマンス bottleneck を特定します。最適化の前に profile を取ってください。
- **Caching:** 高コストな関数結果の cache には `memoise::memoise()` を使います。API の繰り返し呼び出しや複雑な計算に特に有効です。
- **Vectorization:** loop より vectorized operation を優先します。残る反復処理には、`purrr::map_*()` 系または `apply()` 系を使います。

## Tooling と品質

- **Formatting:** `styler` (tidyverse style)、2-space indent、約 100 文字行を使います。
- **Linting:** `.lintr` 経由で設定した `lintr` を使います。
- **Pre-commit:** 自動 lint / format のために `precommit` hook を検討します。
- **Docs:** export する関数には roxygen2 (`@param`、`@return`、`@examples`) を使います。
- **Tests:** unit test しやすい、小さく pure で composable な関数を優先します。
- **Dependencies:** `renv` で管理し、package 追加後は snapshot を取ります。
- **Paths:** 可搬性のために `fs` と `here` を優先します。

## データ加工と I/O

- **Data frames:** tidyverse が中心の file では tibble を優先し、それ以外では base の `data.frame()` でも問題ありません。
- **Iteration:** tidyverse code では `purrr` を使います。base style の code では、明確さやパフォーマンスが向上するなら、明示的な `for` loop より `vapply()`
   (atomic output 向け) や `Map()` (要素ごとの操作向け) のような型安定・vectorized なパターンを優先します。
- **Strings & Dates:** すでに存在するなら `stringr` / `lubridate` を使い、そうでなければ明確な base helper (`nchar()`、`substr()`、明示的 format を伴う `as.Date()` など) を使います。
- **I/O:** 明示的で型付きの reader (例: `readr::read_csv()`) を優先し、parse の前提を明確にします。

## Plotting

- 出版品質の plot には `ggplot2` を優先します。layer は読みやすく保ち、axis と unit に label を付けます。

## エラー処理

- tidyverse 文脈では structured condition のため `rlang::abort()` / `rlang::warn()` を使い、base のみの code では `stop()` / `warning()` を使います。
- 回復可能な操作について:
- 型が同じ fallback 値を使いたい場合は `purrr::possibly()` を使う (より単純)。
- 後で確認や logging を行うため result と error の両方を取得したい場合は `purrr::safely()` を使う。
- 細かな制御や、tidyverse 以外の code との互換性が必要なら base R の `tryCatch()` を使う。
- 一貫した return structure を優先します。通常フローでは型付き output、error 詳細が必要なときだけ structured list を使います。

## セキュリティのベスト プラクティス

- **Command execution:** `system()` より `processx::run()` または `sys::exec_wait()` を優先し、すべての引数を検証・サニタイズします。
- **Database queries:** SQL injection 防止のため parameter 化された `DBI` query を使います。
- **File paths:** ユーザー提供 path は正規化してサニタイズし (例: `fs::path_sanitize()`)、allowlist で検証します。
- **Credentials:** secret をハードコードしません。env var (`Sys.getenv()`)、VCS 外の config、または `keyring` を使います。

## Shiny

- 規模のある app では UI と server logic を module 化します。依存関係を明示するため `eventReactive()` / `observeEvent()` を使います。
- `req()` と分かりやすいユーザー向けメッセージで入力を検証します。
- database には connection pooling (`pool`) を使い、寿命の長い global object は避けます。
- 高コストな計算は isolate し、小さな state には `reactiveVal()` / `reactiveValues()` を優先します。

## R Markdown / Quarto

- chunk は焦点を絞り、`echo`、`message`、`warning` など chunk option を明示します。
- global state は避け、局所 helper を優先します。決定的な chunk には `withr::with_seed()` を使います。

## Copilot 固有ガイダンス

- 現在の file が tidyverse を使っているなら、**tidyverse-first なパターンを提案する** (`dplyr::across()` を superseded verb より優先するなど)。base-R style が見えるなら、**base の idiom を使う**。
- 提案では base 以外の呼び出しを修飾する (例: `dplyr::mutate()`)。
- idiomatic であれば、loop より vectorized 解法や tidy な解法を提案する。
- 長い pipeline より小さな helper 関数を優先する。
- 複数の手法が同等なら、可読性と型安定性を優先し、trade-off を説明する。

---

## 最小例

```r
# Base R variant
scores <- data.frame(id = 1:5, x = c(1, 3, 2, 5, 4))
safe_log <- function(x) tryCatch(log(x), error = function(e) NA_real_)
scores$z <- vapply(scores$x, safe_log, numeric(1))

# Tidyverse variant (if this file uses tidyverse)
result <- tibble::tibble(id = 1:5, x = c(1, 3, 2, 5, 4)) |>
dplyr::mutate(z = purrr::map_dbl(x, purrr::possibly(log, otherwise = NA_real_))) |>
dplyr::filter(z > 0)

# Example reusable helper with roxygen2 doc
#' Compute the z-score of a numeric vector
#' @param x A numeric vector
#' @return Numeric vector of z-scores
#' @examples z_score(c(1, 2, 3))
z_score <- function(x) (x - mean(x, na.rm = TRUE)) / stats::sd(x, na.rm = TRUE)
```
