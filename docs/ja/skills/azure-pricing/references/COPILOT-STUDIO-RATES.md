# Copilot Studio — 課金レートと見積もり

> 出典: [課金レートと管理](https://learn.microsoft.com/en-us/microsoft-copilot-studio/requirements-messages-management)
> 見積もりツール: [Microsoft agent usage estimator](https://microsoft.github.io/copilot-studio-estimator/)
> ライセンス ガイド: [Copilot Studio Licensing Guide](https://go.microsoft.com/fwlink/?linkid=2320995)

## Copilot Credit レート

**1 Copilot Credit = 0.01 USD**

## 課金レート（キャッシュされたスナップショット — 最終更新: 2026年3月）

**重要: 常に以下のソースURLから最新レートを取得することを優先してください。Web取得が利用できない場合のみ、この表を代替として使用してください。**

| 機能 | レート | 単位 |
|---|---|---|
| クラシック回答 | 1 | 1回答あたり |
| 生成回答 | 2 | 1回答あたり |
| エージェント アクション | 5 | 1アクションあたり（トリガー、深い推論、トピック遷移、コンピューター利用） |
| テナント グラフ グラウンディング | 10 | 1メッセージあたり |
| Agent flow アクション | 13 | 100 flow アクションあたり |
| Text & gen AI tools（basic） | 1 | 10回答あたり |
| Text & gen AI tools（standard） | 15 | 10回答あたり |
| Text & gen AI tools（premium） | 100 | 10回答あたり |
| コンテンツ処理ツール | 8 | 1ページあたり |

### 注記

- **クラシック回答**: 事前定義された手動作成の回答。静的であり、作成者が更新しない限り変更されません。
- **生成回答**: AIモデル（GPT）を使って動的に生成される回答。コンテキストや知識ソースに応じて適応します。
- **テナント グラフ グラウンディング**: コネクタ経由の外部データを含む、テナント全体の Microsoft Graph に対するRAG。エージェントごとに任意で設定可能です。
- **エージェント アクション**: アクティビティ マップで確認できる、トリガー、深い推論、トピック遷移などのステップ。Computer-Using Agents を含みます。
- **Text & gen AI tools**: エージェントに埋め込まれたプロンプト ツール。基盤となる言語モデルに応じて basic/standard/premium の3階層があります。
- **Agent flow アクション**: 各ステップでエージェントの推論/オーケストレーションを行わず、事前定義されたフロー アクション列を実行します。

### 推論モデルの課金

推論対応モデルを使用する場合:

```
Total cost = feature rate for operation + text & gen AI tools (premium) per 10 responses
```

例: 推論モデルを使用した生成回答のコストは **2 credits**（生成回答）**+ 10 credits**（premium の1回答あたり、100/10を按分）です。

## 見積もり式

### 入力

| パラメーター | 説明 |
|---|---|
| `users` | エンドユーザー数 |
| `interactions_per_month` | 1ユーザーあたりの月間平均インタラクション数 |
| `knowledge_pct` | 知識ソース由来の回答割合（0-100） |
| `tenant_graph_pct` | 知識回答のうち、tenant graph grounding を使う割合（0-100） |
| `tool_prompt` | 1セッションあたりの平均 Prompt ツール呼び出し回数 |
| `tool_agent_flow` | 1セッションあたりの平均 Agent flow 呼び出し回数 |
| `tool_computer_use` | 1セッションあたりの平均 Computer use 呼び出し回数 |
| `tool_custom_connector` | 1セッションあたりの平均 Custom connector 呼び出し回数 |
| `tool_mcp` | 1セッションあたりの平均 MCP（Model Context Protocol）呼び出し回数 |
| `tool_rest_api` | 1セッションあたりの平均 REST API 呼び出し回数 |
| `prompts_basic` | 1セッションあたりの平均 basic AI プロンプト利用回数 |
| `prompts_standard` | 1セッションあたりの平均 standard AI プロンプト利用回数 |
| `prompts_premium` | 1セッションあたりの平均 premium AI プロンプト利用回数 |

### 計算

```
total_sessions = users × interactions_per_month

── Knowledge Credits ──
tenant_graph_credits    = total_sessions × (knowledge_pct/100) × (tenant_graph_pct/100) × 10
generative_answer_credits = total_sessions × (knowledge_pct/100) × (1 - tenant_graph_pct/100) × 2
classic_answer_credits  = total_sessions × (1 - knowledge_pct/100) × 1

── Agent Tools Credits ──
tool_calls = total_sessions × (prompt + computer_use + custom_connector + mcp + rest_api)
tool_credits = tool_calls × 5

── Agent Flow Credits ──
flow_calls = total_sessions × tool_agent_flow
flow_credits = ceil(flow_calls / 100) × 13

── Prompt Modifier Credits ──
basic_credits    = ceil(total_sessions × prompts_basic / 10) × 1
standard_credits = ceil(total_sessions × prompts_standard / 10) × 15
premium_credits  = ceil(total_sessions × prompts_premium / 10) × 100

── Total ──
total_credits = knowledge + tools + flows + prompts
cost_usd = total_credits × 0.01
```

## 課金例（Microsoft Docs より）

### カスタマー サポート エージェント

- 1セッションあたり: クラシック回答4回 + 生成回答2回
- 1日あたり900人の顧客
- **日次**: `[(4×1) + (2×2)] × 900 = 7,200 credits`
- **月次（30日）**: 約216,000 credits = **約$2,160**

### 営業実績エージェント（Tenant Graph Grounded）

- 1セッションあたり: 生成回答4回 + tenant graph grounded response 4回
- ライセンス未保有ユーザー100人
- **日次**: `[(4×2) + (4×10)] × 100 = 4,800 credits`
- **月次（30日）**: 約144,000 credits = **約$1,440**

### 注文処理エージェント

- トリガー1回あたりアクション呼び出し4回（自律実行）
- **トリガーあたり**: `4 × 5 = 20 credits`

## 従業員向け vs 顧客向けエージェント種別

| エージェント種別 | M365 Copilot に含まれるか？ |
|---|---|
| 従業員向け（BtoE） | ユーザーが Microsoft 365 Copilot ライセンスを保有している場合、クラシック回答・生成回答・tenant graph grounding は無料（0コスト）で含まれます |
| 顧客/パートナー向け | すべての利用が通常どおり課金されます |

## 超過時の制御

- 前払い容量の **125%** でトリガー
- カスタム エージェントは無効化（進行中の会話は継続）
- テナント管理者にメール通知を送信
- 解決策: 容量の再割り当て、追加購入、または従量課金の有効化

## ライブソースURL

最新レートについては、以下のページから内容を取得してください:

- [課金レートと管理](https://learn.microsoft.com/en-us/microsoft-copilot-studio/requirements-messages-management)
- [Copilot Studio のライセンス](https://learn.microsoft.com/en-us/microsoft-copilot-studio/billing-licensing)
- [Copilot Studio Licensing Guide (PDF)](https://go.microsoft.com/fwlink/?linkid=2320995)

