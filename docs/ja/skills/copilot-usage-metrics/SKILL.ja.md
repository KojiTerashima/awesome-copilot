---
name: copilot-usage-metrics
description: GitHub CLI と REST API を使用して、organization と enterprise の GitHub Copilot usage metrics を取得して表示します。
---

# Copilot Usage Metrics

あなたは GitHub CLI (`gh`) を使って GitHub Copilot usage metrics を取得し表示する skill です。

## この skill を使う場面

次のような質問があるときにこの skill を使います。
- Copilot の usage metrics、adoption、または統計
- org や enterprise で何人が Copilot を使っているか
- Copilot の acceptance rate、suggestion、または chat usage
- ユーザー単位の Copilot usage breakdown
- 特定日の Copilot usage

## この skill の使い方

1. ユーザーが **organization** レベルか **enterprise** レベルの metrics を求めているか判断する。
2. 未提供であれば org 名または enterprise slug を確認する。
3. **aggregated** metrics か **per-user** metrics かを判断する。
4. **specific day**（YYYY-MM-DD 形式）の metrics が必要か、一般的な最近の metrics が必要かを判断する。
5. この skill の directory にある適切な script を実行する。

## 利用可能な script

### Organization metrics

- `get-org-metrics.sh <org> [day]` — organization の aggregated Copilot usage metrics を取得する。必要に応じて YYYY-MM-DD 形式の特定日を渡せる。
- `get-org-user-metrics.sh <org> [day]` — organization の per-user Copilot usage metrics を取得する。必要に応じて特定日を渡せる。

### Enterprise metrics

- `get-enterprise-metrics.sh <enterprise> [day]` — enterprise の aggregated Copilot usage metrics を取得する。必要に応じて特定日を渡せる。
- `get-enterprise-user-metrics.sh <enterprise> [day]` — enterprise の per-user Copilot usage metrics を取得する。必要に応じて特定日を渡せる。

## 出力の整形

結果をユーザーに提示するときは:
- total active users、acceptance rate、total suggestions、total chat interactions などの主要 metric を要約する
- per-user breakdown には table を使う
- 複数日を比較する場合は trend を強調する
- metrics data は 2025-10-10 から利用可能で、最大 1 年分の過去データにアクセスできることを明記する

## 重要な注意点

- これらの API endpoint には **GitHub Enterprise Cloud** が必要です。
- ユーザーには適切な権限（enterprise owner、billing manager、または `manage_billing:copilot` / `read:enterprise` scope を持つ token）が必要です。
- enterprise settings で "Copilot usage metrics" policy が有効になっている必要があります。
- API が 403 を返した場合は、token 権限と enterprise policy settings を確認するよう案内します。
