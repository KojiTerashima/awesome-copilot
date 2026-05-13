# CodeQL Alert Management Reference

CodeQL が生成するコードスキャンアラートの理解、トリアージ、dismiss、解消のガイドです。

## Alert Severity Levels

### Standard Severity

| Level | Description |
|---|---|
| `Error` | 信頼度・影響度が高く、修正すべき問題 |
| `Warning` | 信頼度または影響度が中程度の問題 |
| `Note` | 信頼度が低い、または情報提供レベルの指摘 |

### Security Severity

| Level | CVSS Score Range | Description |
|---|---|---|
| `Critical` | > 9.0 | 直ちに対応が必要な重大脆弱性 |
| `High` | 7.0 – 8.9 | 優先度高で対処すべき脆弱性 |
| `Medium` | 4.0 – 6.9 | 通常フローで対処する中程度脆弱性 |
| `Low` | 0.1 – 3.9 | セキュリティ影響が限定的な軽微問題 |

Security severity がある場合、表示/ソートでは standard severity より優先されます。

## Alert Labels

| Label | Description |
|---|---|
| **Generated** | ビルドプロセスで生成されたコード |
| **Test** | テストコード（パスベース判定） |
| **Library** | ライブラリまたはサードパーティコード |
| **Documentation** | ドキュメントファイル |

これらのラベルはファイルパスに基づき自動付与され、手動上書きできません。

## Alert Triage in Pull Requests

- アラートは **Conversation** と **Files changed** に注釈表示
- **Code scanning results** チェックに集約表示
- PR diff に検出行がすべて含まれる場合のみ表示
- 変更行上の新規アラートのみ表示（既存アラートは表示しない）

既定では `error` / `critical` / `high` でチェック失敗します。

## Copilot Autofix

GitHub Copilot Autofix は PR 上の CodeQL アラートに対し修正提案を自動生成します。

- public repository は無料
- private repository は GitHub Code Security ライセンスで利用可能
- Copilot サブスクリプション不要

## Dismissing Alerts

### Dismiss する場面
- 偽陽性
- テストコード限定で許容可能
- 修正コストが便益を上回る

### Dismissal Reasons

| Reason | When to Use |
|---|---|
| **False positive** | 実際には安全で誤検知 |
| **Won't fix** | リスク受容、またはコードが廃止予定 |
| **Used in tests** | 脆弱なパターンがテストコードのみで使用 |

Dismiss コメントは監査目的で保存されます（`alerts/{alert_number}` → `dismissed_comment`）。

## Resolving Alerts

1. 脆弱性を修正
2. commit/push
3. 次回スキャンで修正確認
4. 確認されると自動クローズ

## Alert Data Flow

`path-problem` クエリでは以下を表示:
- **Source**
- **Sink**
- **Path**

注釈の **Show paths** でデータフローを可視化できます。
