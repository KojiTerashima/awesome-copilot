---
name: flowstudio-power-automate-governance
description: >-
  Govern Power Automate flows and Power Apps at scale using the FlowStudio MCP
  cached store. Classify flows by business impact, detect orphaned resources,
  audit connector usage, enforce compliance standards, manage notification rules,
  and compute governance scores — all without Dataverse or the CoE Starter Kit.
  Load this skill when asked to: tag or classify flows, set business impact,
  assign ownership, detect orphans, audit connectors, check compliance, compute
  archive scores, manage notification rules, run a governance review, generate
  a compliance report, offboard a maker, or any task that involves writing
  governance metadata to flows. Requires a FlowStudio for Teams or MCP Pro+
  subscription — see https://mcp.flowstudio.app
metadata:
  openclaw:
    requires:
      env:
        - FLOWSTUDIO_MCP_TOKEN
    primaryEnv: FLOWSTUDIO_MCP_TOKEN
    homepage: https://mcp.flowstudio.app
---
# FlowStudio MCP を使用してガバナンスを自動化する

FlowStudio を通じて Power Automate フローを大規模に分類、タグ付け、管理します
MCP **キャッシュ ストア** — Dataverse なし、CoE スターター キットなし、および
Power Automate ポータルなし。

このスキルは `update_store_flow` を使用してガバナンス メタデータを書き込み、
監視ツール (`list_store_flows`、`get_store_flow`、`list_store_makers`、
など) テナントの状態を読み取ります。監視およびヘルスチェックのワークフローについては、次を参照してください。
`flowstudio-power-automate-monitoring` スキル。

> **各セッションを `tools/list`** で開始して、ツール名とパラメータを確認します。
> このスキルは、`tools/list` では教えてもらえないワークフローとパターンをカバーします。
> このドキュメントが `tools/list` または実際の API 応答と一致しない場合、API が優先されます。

---

## 重要: フロー ID を抽出する方法

`list_store_flows` は `id` を `<environmentId>.<flowId>` の形式で返します。 **分割する必要があります
最初の `.`** で、他のすべてのツールの `environmentName` と `flowName` を取得します。```
id = "Default-<envGuid>.<flowGuid>"
environmentName = "Default-<envGuid>"    (everything before first ".")
flowName = "<flowGuid>"                  (everything after first ".")
```また、`displayName` が含まれないエントリ、または `state=Deleted` が含まれるエントリをスキップします。
これらは、Power Automate にはもう存在しない、まばらなレコードまたはフローです。
削除されたフローに `monitor=true` がある場合は、モニタリングを無効にすることを提案します
(`update_store_flow` と `monitor=false`) 監視スロットを解放します
(標準プランには 20 が含まれます)。

---

## 書き込みツール: `update_store_flow`

`update_store_flow` はガバナンス メタデータを **Flow Studio キャッシュに書き込みます
のみ** — Power Automate のフローは変更されません。これらのフィールドは、
`get_live_flow` または PA ポータルからは表示されません。それらは、
Flow Studio ストアと Flow Studio のスキャン パイプラインによって使用され、
通知ルール。

これは次のことを意味します。
- `ownerTeam` / `supportEmail` — Flow Studio が誰を考慮するかを設定します
  ガバナンス連絡先。実際の PA フローの所有者は変更されません。
- `rule_notify_email` — Flow Studio の失敗/実行不足を受け取る人を設定します
  通知。 Microsoft の組み込みフロー障害アラートは変更されません。
- `monitor` / `critical` / `businessImpact` — フロー スタジオの分類
  のみ。 Power Automate には同等のフィールドはありません。

マージ セマンティクス — 指定したフィールドのみが更新されます。全額を返します
更新されたレコード (`get_store_flow` と同じ形式)。

必須パラメータ: `environmentName`、`flowName`。他のフィールドはすべてオプションです。

### 設定可能なフィールド

|フィールド |タイプ |目的 |
|---|---|---|
| `monitor` |ブール |実行レベルのスキャンを有効にする (標準プラン: 20 フローが含まれる) |
| `rule_notify_onfail` |ブール |失敗した実行時に電子メール通知を送信する |
| `rule_notify_onmissingdays` |番号 |フローが N 日間実行されなかった場合に通知を送信します (0 = 無効) |
| `rule_notify_email` |文字列 |カンマ区切りの通知受信者 |
| `description` |文字列 |フローの動作 |
| `tags` |文字列 |分類タグ (説明 `#hashtags` からも自動抽出)
| `businessImpact` |文字列 |低 / 中 / 高 / クリティカル |
| `businessJustification` |文字列 |フローが存在する理由、フローによって自動化されるプロセス |
| `businessValue` |文字列 |ビジネス価値ステートメント |
| `ownerTeam` |文字列 |責任あるチーム |
| `ownerBusinessUnit` |文字列 |ビジネスユニット |
| `supportGroup` |文字列 |サポート エスカレーション グループ |
| `supportEmail` |文字列 |サポート連絡先メールアドレス |
| `critical` |ブール |ビジネスクリティカルとして指定 |
| `tier` |文字列 |スタンダードまたはプレミアム |
| `security` |文字列 |セキュリティの分類または注意事項 |

> **`security` に関する注意:** `get_store_flow` の `security` フィールド
> 構造化された JSON (例: `{"triggerRequestAuthenticationType":"All"}`) が含まれています。
> `"reviewed"` のようなプレーンな文字列を書くとこれが上書きされます。マークを付けるには
> セキュリティレビュー済みのフローなので、代わりに `tags` を使用してください。

---

## ガバナンス ワークフロー

### 1. コンプライアンスの詳細レビュー

必要なガバナンス メタデータが欠落しているフローを特定します。これに相当します。
CoE スターター キットの開発者コンプライアンス センター。```
1. Ask the user which compliance fields they require
   (or use their organization's existing governance policy)
2. list_store_flows
3. For each flow (skip entries without displayName or state=Deleted):
   - Split id → environmentName, flowName
   - get_store_flow(environmentName, flowName)
   - Check which required fields are missing or empty
4. Report non-compliant flows with missing fields listed
5. For each non-compliant flow:
   - Ask the user for values
   - update_store_flow(environmentName, flowName, ...provided fields)
```**コンプライアンスチェックに利用可能なフィールド:**

|フィールド |ポリシーの例 |
|---|---|
| `description` |すべてのフローを文書化する必要があります |
| `businessImpact` |低 / 中 / 高 / クリティカルに分類 |
| `businessJustification` |高/重大な影響のフローに必要 |
| `ownerTeam` |すべてのフローには責任あるチームが必要です。
| `supportEmail` |実稼働フローに必要 |
| `monitor` |重要なフローに必要 (注: 標準プランには 20 個の監視対象フローが含まれます) |
| `rule_notify_onfail` |監視対象フローに推奨 |
| `critical` |ビジネスクリティカルなフローを指定する |

> 各組織は独自のコンプライアンス ルールを定義します。上記のフィールドは、
> 一般的な Power Platform ガバナンス パターンに基づく提案 (CoE スターター)
>キット）。フローにフラグを付ける前に、ユーザーに要件を尋ねます。
> 非準拠です。
>
> **ヒント:** MCP 経由で作成または更新されたフローには、すでに `description` が設定されています
> (`update_live_flow` によって自動追加されます)。で手動で作成されたフロー
> Power Automate ポータルは、ガバナンス メタデータが欠落している可能性が最も高いものです。

### 2. 孤立したリソースの検出

削除または無効化された Azure AD アカウントが所有するフローを検索します。```
1. list_store_makers
2. Filter where deleted=true AND ownerFlowCount > 0
   Note: deleted makers have NO displayName/mail — record their id (AAD OID)
3. list_store_flows → collect all flows
4. For each flow (skip entries without displayName or state=Deleted):
   - Split id → environmentName, flowName
   - get_store_flow(environmentName, flowName)
   - Parse owners: json.loads(record["owners"])
   - Check if any owner principalId matches an orphaned maker id
5. Report orphaned flows: maker id, flow name, flow state
6. For each orphaned flow:
   - Reassign governance: update_store_flow(environmentName, flowName,
       ownerTeam="NewTeam", supportEmail="new-owner@contoso.com")
   - Or decommission: set_store_flow_state(environmentName, flowName,
       state="Stopped")
```> `update_store_flow` はキャッシュ内のガバナンス メタデータのみを更新します。に
> 実際の PA 所有権を譲渡するには、管理者は Power Platform 管理者を使用する必要があります
> センターまたは PowerShell。
>
> **注意:** 孤立したフローの多くはシステムによって生成されます (
> `DataverseSystemUser` は、SLA モニタリング、ナレッジ記事、
>など）。これらは人によって構築されたものではありません - タグ付けを検討してください
> 再割り当てではなく。
>
> **対象範囲:** このワークフローは、キャッシュされたストアのみを検索し、ストアは検索しません。
> ライブ PA API。最後のスキャン後に作成されたフローは表示されません。

### 3. アーカイブスコアの計算

安全なクリーンアップを識別するために、フローごとの非アクティブ スコア (0 ～ 7) を計算します。
候補者たち。 CoE スターター キットのアーカイブ スコアと一致します。```
1. list_store_flows
2. For each flow (skip entries without displayName or state=Deleted):
   - Split id → environmentName, flowName
   - get_store_flow(environmentName, flowName)
3. Compute archive score (0-7), add 1 point for each:
   +1  lastModifiedTime within 24 hours of createdTime
   +1  displayName contains "test", "demo", "copy", "temp", or "backup"
       (case-insensitive)
   +1  createdTime is more than 12 months ago
   +1  state is "Stopped" or "Suspended"
   +1  json.loads(owners) is empty array []
   +1  runPeriodTotal = 0 (never ran or no recent runs)
   +1  parse json.loads(complexity) → actions < 5
4. Classify:
   Score 5-7: Recommend archive — report to user for confirmation
   Score 3-4: Flag for review →
     Read existing tags from get_store_flow response, append #archive-review
     update_store_flow(environmentName, flowName, tags="<existing> #archive-review")
   Score 0-2: Active, no action
5. For user-confirmed archives:
   set_store_flow_state(environmentName, flowName, state="Stopped")
   Read existing tags, append #archived
   update_store_flow(environmentName, flowName, tags="<existing> #archived")
```> **「アーカイブ」の意味:** Power Automate にはネイティブのアーカイブ機能がありません。
> MCP 経由でアーカイブすることは、(1) フローを実行できないように停止することを意味します。
> (2) `#archived` タグを付けて、将来のクリーンアップで検出できるようにします。
> 実際の削除には Power Automate ポータルまたは管理者 PowerShell が必要です
> — MCP ツールでは実行できません。

### 4. コネクタの監査

監視対象のフロー全体でどのコネクタが使用されているかを監査します。 DLP に便利
影響分析とプレミアム ライセンスの計画。```
1. list_store_flows(monitor=true)
   (scope to monitored flows — auditing all 1000+ flows is expensive)
2. For each flow (skip entries without displayName or state=Deleted):
   - Split id → environmentName, flowName
   - get_store_flow(environmentName, flowName)
   - Parse connections: json.loads(record["connections"])
     Returns array of objects with apiName, apiId, connectionName
   - Note the flow-level tier field ("Standard" or "Premium")
3. Build connector inventory:
   - Which apiNames are used and by how many flows
   - Which flows have tier="Premium" (premium connector detected)
   - Which flows use HTTP connectors (apiName contains "http")
   - Which flows use custom connectors (non-shared_ prefix apiNames)
4. Report inventory to user
   - For DLP analysis: user provides their DLP policy connector groups,
     agent cross-references against the inventory
```> **監視対象のフローが対象範囲です。** 各フローには `get_store_flow` 呼び出しが必要です
> `connections` JSON を読み取ります。標準プランには最大 20 の監視対象フローがあります —
> 管理可能です。大規模なテナント (1000 以上) 内のすべてのフローを監査するのは非常に困難です
> API呼び出しにコストがかかる。
>
> **`list_store_connections`** は接続インスタンス (作成者) を返します
> どの接続) ですが、フローごとのコネクタ タイプではありません。接続に使用します
> コネクタ監査ではなく、環境ごとの数です。
>
> DLP ポリシー定義は MCP 経由では利用できません。エージェントが構築するのは、
> コネクタの在庫。ユーザーは DLP 分類を提供します
> に対する相互参照。

### 5. 通知ルールの管理

大規模なフローの監視とアラートを構成します。```
Enable failure alerts on all critical flows:
1. list_store_flows(monitor=true)
2. For each flow (skip entries without displayName or state=Deleted):
   - Split id → environmentName, flowName
   - get_store_flow(environmentName, flowName)
   - If critical=true AND rule_notify_onfail is not true:
     update_store_flow(environmentName, flowName,
       rule_notify_onfail=true,
       rule_notify_email="oncall@contoso.com")
   - If NO flows have critical=true: this is a governance finding.
     Recommend the user designate their most important flows as critical
     using update_store_flow(critical=true) before configuring alerts.

Enable missing-run detection for scheduled flows:
1. list_store_flows(monitor=true)
2. For each flow where triggerType="Recurrence" (available on list response):
   - Skip flows with state="Stopped" or "Suspended" (not expected to run)
   - Split id → environmentName, flowName
   - get_store_flow(environmentName, flowName)
   - If rule_notify_onmissingdays is 0 or not set:
     update_store_flow(environmentName, flowName,
       rule_notify_onmissingdays=2)
```> `critical`、`rule_notify_onfail`、`rule_notify_onmissingdays` は
> `list_store_flows` ではなく、`get_store_flow` から利用可能です。リストコール
> 監視対象のフローを事前にフィルタリングします。詳細呼び出しでは、通知フィールドがチェックされます。
>
> **監視制限:** 標準プラン (FlowStudio for Teams / MCP Pro+)
> には 20 の監視対象フローが含まれます。 `monitor=true` を一括有効にする前に確認してください
> すでに監視されているフローの数:
> `len(list_store_flows(monitor=true))`

### 6. 分類とタグ付け

コネクタのタイプ、ビジネス機能、またはリスク レベルごとにフローを一括分類します。```
Auto-tag by connector:
1. list_store_flows
2. For each flow (skip entries without displayName or state=Deleted):
   - Split id → environmentName, flowName
   - get_store_flow(environmentName, flowName)
   - Parse connections: json.loads(record["connections"])
   - Build tags from apiName values:
     shared_sharepointonline → #sharepoint
     shared_teams → #teams
     shared_office365 → #email
     Custom connectors → #custom-connector
     HTTP-related connectors → #http-external
   - Read existing tags from get_store_flow response, append new tags
   - update_store_flow(environmentName, flowName,
       tags="<existing tags> #sharepoint #teams")
```> **2 つのタグ システム:** `list_store_flows` で示されるタグは自動抽出されます
> フローの `description` フィールドから (例: メーカーが `#operations` をフィールドに書き込む)
> PA ポータルの説明)。 `update_store_flow(tags=...)` 経由で設定されたタグ
> Azure テーブル キャッシュの別のフィールドに書き込みます。彼らは独立しています —
> ストアタグの記述は説明には影響せず、
> ポータル内の説明はストア タグに影響しません。
>
> **タグの結合:** `update_store_flow(tags=...)` はストア タグを上書きします
>フィールド。他のワークフローからタグが失われないようにするには、現在のストアを読み取ります。
> `get_store_flow` のタグを最初に追加し、新しいタグを追加してから書き戻します。
>
> `get_store_flow` にはすでに `tier` フィールド (標準/プレミアム) が計算されています
> スキャンパイプラインによって。次の場合にのみ `update_store_flow(tier=...)` を使用してください。
> オーバーライドする必要があります。

### 7. メーカーのオフボーディング

従業員が退職した場合、そのフローとアプリを特定し、再割り当てする
Flow Studio のガバナンス連絡先と通知受信者。```
1. get_store_maker(makerKey="<departing-user-aad-oid>")
   → check ownerFlowCount, ownerAppCount, deleted status
2. list_store_flows → collect all flows
3. For each flow (skip entries without displayName or state=Deleted):
   - Split id → environmentName, flowName
   - get_store_flow(environmentName, flowName)
   - Parse owners: json.loads(record["owners"])
   - If any principalId matches the departing user's OID → flag
4. list_store_power_apps → filter where ownerId matches the OID
5. For each flagged flow:
   - Check runPeriodTotal and runLast — is it still active?
   - If keeping:
     update_store_flow(environmentName, flowName,
       ownerTeam="NewTeam", supportEmail="new-owner@contoso.com")
   - If decommissioning:
     set_store_flow_state(environmentName, flowName, state="Stopped")
     Read existing tags, append #decommissioned
     update_store_flow(environmentName, flowName, tags="<existing> #decommissioned")
6. Report: flows reassigned, flows stopped, apps needing manual reassignment
```> **ここでの「再割り当て」の意味:** `update_store_flow` はフローを変更する人
> Studio はガバナンス担当者と Flow Studio を受け取る人を検討します
> 通知。実際の Power Automate フローは転送されません。
> 所有権 — Power Platform 管理センターまたは PowerShell が必要です。
> また、`rule_notify_email` も更新して、失敗通知が新しいアドレスに送信されるようにします。
> 退職する従業員の電子メールの代わりにチームに連絡します。
>
> Power Apps の所有権は MCP ツールを介して変更できません。報告してください
> Power Apps 管理センターでの手動再割り当て。

### 8. セキュリティのレビュー

キャッシュされたストア データを使用して、潜在的なセキュリティ上の問題がないかフローを確認します。```
1. list_store_flows(monitor=true)
2. For each flow (skip entries without displayName or state=Deleted):
   - Split id → environmentName, flowName
   - get_store_flow(environmentName, flowName)
   - Parse security: json.loads(record["security"])
   - Parse connections: json.loads(record["connections"])
   - Read sharingType directly (top-level field, NOT inside security JSON)
3. Report findings to user for review
4. For reviewed flows:
   Read existing tags, append #security-reviewed
   update_store_flow(environmentName, flowName, tags="<existing> #security-reviewed")
   Do NOT overwrite the security field — it contains structured auth data
```**セキュリティレビューに利用可能なフィールド:**

|フィールド |どこ |それがあなたに伝えること |
|---|---|---|
| `security.triggerRequestAuthenticationType` |セキュリティ JSON | `"All"` = HTTP トリガーは認証されていないリクエストを受け入れます |
| `sharingType` |トップレベル | `"Coauthor"` = 編集のために共著者と共有 |
| `connections` |接続 JSON |フローが使用するコネクタ (HTTP、カスタムを確認) |
| `referencedResources` | JSON文字列 | SharePoint サイト、Teams チャネル、フローがアクセスする外部 URL |
| `tier` |トップレベル | `"Premium"` = プレミアム コネクタを使用 |

> 何がセキュリティ上の懸念事項となるかは、各組織が決定します。たとえば、
> Webhook レシーバーには認証されていない HTTP トリガーが予期されます (ストライプ、
> GitHub) ですが、内部フローにとってはリスクとなる可能性があります。コンテキストに沿って調査結果をレビューする
> フラグを立てる前に。

### 9. 環境ガバナンス

環境のコンプライアンスとスプロールを監査します。```
1. list_store_environments
   Skip entries without displayName (tenant-level metadata rows)
2. Flag:
   - Developer environments (sku="Developer") — should be limited
   - Non-managed environments (isManagedEnvironment=false) — less governance
   - Note: isAdmin=false means the current service account lacks admin
     access to that environment, not that the environment has no admin
3. list_store_flows → group by environmentName
   - Flow count per environment
   - Failure rate analysis: runPeriodFailRate is on the list response —
     no need for per-flow get_store_flow calls
4. list_store_connections → group by environmentName
   - Connection count per environment
```### 10. ガバナンスダッシュボード

テナント全体のガバナンスの概要を生成します。```
Efficient metrics (list calls only):
1. total_flows = len(list_store_flows())
2. monitored = len(list_store_flows(monitor=true))
3. with_onfail = len(list_store_flows(rule_notify_onfail=true))
4. makers = list_store_makers()
   → active = count where deleted=false
   → orphan_count = count where deleted=true AND ownerFlowCount > 0
5. apps = list_store_power_apps()
   → widely_shared = count where sharedUsersCount > 3
6. envs = list_store_environments() → count, group by sku
7. conns = list_store_connections() → count

Compute from list data:
- Monitoring %: monitored / total_flows
- Notification %: with_onfail / monitored
- Orphan count: from step 4
- High-risk count: flows with runPeriodFailRate > 0.2 (on list response)

Detailed metrics (require get_store_flow per flow — expensive for large tenants):
- Compliance %: flows with businessImpact set / total active flows
- Undocumented count: flows without description
- Tier breakdown: group by tier field

For detailed metrics, iterate all flows in a single pass:
  For each flow from list_store_flows (skip sparse entries):
    Split id → environmentName, flowName
    get_store_flow(environmentName, flowName)
    → accumulate businessImpact, description, tier
```---

## フィールド参照: `get_store_flow` ガバナンスで使用されるフィールド

以下のフィールドはすべて `get_store_flow` 応答に存在することが確認されています。
`*` でマークされたフィールドは、`list_store_flows` でも利用できます (安価)。

|フィールド |タイプ |ガバナンス利用 |
|---|---|---|
| `displayName` * |文字列 |アーカイブスコア (テスト/デモ名検出) |
| `state` * |文字列 |アーカイブ スコア、ライフサイクル管理 |
| `tier` |文字列 |ライセンス監査 (Standard と Premium) |
| `monitor` * |ブール |このフローはアクティブに監視されていますか? |
| `critical` |ブール |ビジネスクリティカルな指定 (update_store_flow 経由で設定可能) |
| `businessImpact` |文字列 |コンプライアンスの分類 |
| `businessJustification` |文字列 |コンプライアンス証明書 |
| `ownerTeam` |文字列 |所有者の責任 |
| `supportEmail` |文字列 |エスカレーション連絡先 |
| `rule_notify_onfail` |ブール |障害アラートが設定されていますか? |
| `rule_notify_onmissingdays` |番号 | SLA監視は設定されていますか? |
| `rule_notify_email` |文字列 |アラート受信者 |
| `description` |文字列 |ドキュメントの完全性 |
| `tags` |文字列 |分類 — `list_store_flows` は説明から抽出されたハッシュタグのみを表示します。 `update_store_flow` によって書き込まれたストア タグを読み戻すには `get_store_flow` が必要です。
| `runPeriodTotal` * |番号 |活動レベル |
| `runPeriodFailRate` * |番号 |健康状態 |
| `runLast` | ISO文字列 |最終実行タイムスタンプ |
| `scanned` | ISO文字列 |データの鮮度 |
| `deleted` |ブール |ライフサイクル追跡 |
| `createdTime` * | ISO文字列 |アーカイブスコア (年齢) |
| `lastModifiedTime` * | ISO文字列 |アーカイブスコア (古さ) |
| `owners` | JSON文字列 |孤立検出、所有権監査 — json.loads() で解析 |
| `connections` | JSON文字列 |コネクタ監査、層 — json.loads() で解析 |
| `complexity` | JSON文字列 |アーカイブスコア (シンプルさ) — json.loads() で解析 |
| `security` | JSON文字列 |認証タイプの監査 — json.loads() で解析し、`triggerRequestAuthenticationType` を含みます。
| `sharingType` |文字列 |過剰共有の検出 (トップレベル、セキュリティ内ではない) |
| `referencedResources` | JSON文字列 | URL 監査 - json.loads() で解析 |

---

## 関連スキル

- `flowstudio-power-automate-monitoring` — ヘルスチェック、故障率、インベントリ（読み取り専用）
- `flowstudio-power-automate-mcp` — コア接続セットアップ、ライブツールリファレンス
- `flowstudio-power-automate-debug` — アクションレベルの入出力による詳細な診断
- `flowstudio-power-automate-build` — フロー定義の構築とデプロイ