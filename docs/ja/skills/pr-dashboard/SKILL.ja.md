---
name: pr-dashboard
description: 'ブラウザで GitHub PR ダッシュボードを開きます。ユーザーがプル リクエストを表示する、PR ダッシュボードを開く、日付範囲の PR を表示する、または PR ステータスを確認することを要求する場合に使用します。トリガー フレーズには、「PR を表示」、「PR ダッシュボードを開く」、「プル リクエスト ダッシュボード」などがあります。'
---

# PRダッシュボード

指定された日付範囲とロール フィルターに基づいて、ブラウザーで GitHub PR ダッシュボードを生成して開きます。

**前提条件:** GitHub CLI (`gh`) がインストールされ、認証されている (`gh auth login`) 必要があります。

## 何をするか

このスキルにバンドルされている CLI スクリプトを見つけて実行します。
```bash
SKILL_SCRIPT=$(find ~/.copilot -name "pr-dashboard-cli.mjs" -path "*/pr-dashboard/scripts/*" 2>/dev/null | head -1)
node "$SKILL_SCRIPT" "<query>" "<role>"
```

- `<query>`: ユーザーが指定した日付範囲 (デフォルト: `last 7 days`)
- `<role>`: `Authored by me`、`Requested reviews`、`Assigned to me`、`All` のいずれか (デフォルト: `Authored by me`)

## ユーザーのリクエストを解析する

ユーザーのメッセージから日付範囲と役割を抽出します。例:

|ユーザーの発言 |クエリ |役割 |
|---|---|---|
|私のPRを表示 | `last 7 days` | `Authored by me` |
|過去 2 週間の自己PRを表示 | `last 2 weeks` | `Authored by me` |
|今月の PR ダッシュボードのレビュー | `this month` | `Requested reviews` |
| PR ダッシュボード 2026 年 3 月割り当て | `march 2026` | `Assigned to me` |
|過去 30 日間のすべての PR を表示 | `last 30 days` | `All` |

**役割キーワードのマッピング:**
- 「私の PR」、「作成者」、「私が書きました」 → `Authored by me`
- 「レビュー」、「レビュー要求」、「レビュー中」 → `Requested reviews`
- 「割り当て済み」 → `Assigned to me`
- 「すべて」「私に関わる」 → `All`

## サポートされている日付範囲形式

スクリプトは自然言語を理解します。それをそのまま渡します。
- `last 7 days`、`last 2 weeks`、`last 30 days`
- `this week`、`last week`、`this month`、`last month`
- `march 2026`、`feb 2025`
- @@コード0@@
- `2025` (通年)

## 走った後

ダッシュボードがブラウザで開いていることをユーザーに伝えます。スクリプトは進行状況を標準出力に出力します。エラーで終了した場合は、エラー出力を表示し、認証の問題であれば `gh auth login` を実行することを提案します。
