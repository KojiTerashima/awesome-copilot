# アーキテクチャ ガイダンス ソース（設計方針の意思決定用）

Azure 公式のアーキテクチャ ガイダンスを**設計方針の意思決定にのみ**使用するためのソースレジストリです。

> **このドキュメント内の URL は「どこを参照すべきか」のソース一覧です。**
> これらの URL の内容を固定的な事実としてハードコードしないでください。
> SKU、API バージョン、リージョン、モデル可用性、PE マッピングの判断には使用しないでください。これらは `azure-dynamic-sources.md` でのみ扱います。

---

## 目的の分離

| Purpose | Document to Use | Decidable Items |
|---------|----------------|-----------------|
| **設計方針の意思決定** | このドキュメント（architecture-guidance-sources） | アーキテクチャ パターン、ベストプラクティス、サービス組み合わせの方向性、セキュリティ境界設計 |
| **デプロイ仕様の検証** | `azure-dynamic-sources.md` | API バージョン、SKU、リージョン、モデル可用性、PE groupId、実際のプロパティ値 |

**このドキュメントを使って判断してはならないもの:**
- API バージョン
- SKU 名/価格
- リージョン可用性
- モデル名/バージョン/デプロイ種別
- PE groupId / DNS Zone マッピング
- リソース プロパティの具体値

---

## Primary Sources

設計方針の意思決定に対するターゲット取得先です。

| ID | Document | URL | Purpose |
|----|----------|-----|---------|
| A1 | Azure Architecture Center | https://learn.microsoft.com/en-us/azure/architecture/ | ハブ — ドメイン別ドキュメントを探すための入口 |
| A2 | Well-Architected Framework | https://learn.microsoft.com/en-us/azure/architecture/framework/ | セキュリティ/信頼性/パフォーマンス/コスト/運用の原則 |
| A3 | Cloud Adoption Framework / Landing Zone | https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/landing-zone/ | エンタープライズ ガバナンス、ネットワーク トポロジ、サブスクリプション構造 |
| A4 | Azure AI/ML Architecture | https://learn.microsoft.com/en-us/azure/architecture/ai-ml/ | AI/ML ワークロード参照アーキテクチャのハブ |
| A5 | Basic Foundry Chat Reference Architecture | https://learn.microsoft.com/en-us/azure/architecture/ai-ml/architecture/basic-azure-ai-foundry-chat | 基本的な Foundry ベースのチャットボット構成 |
| A6 | Baseline AI Foundry Chat Reference Architecture | https://learn.microsoft.com/en-us/azure/architecture/ai-ml/architecture/baseline-openai-e2e-chat | Foundry チャットボットのエンタープライズ ベースライン（ネットワーク分離を含む） |
| A7 | RAG Solution Design Guide | https://learn.microsoft.com/en-us/azure/architecture/ai-ml/guide/rag/rag-solution-design-and-evaluation-guide | RAG パターン設計ガイド |
| A8 | Microsoft Fabric Overview | https://learn.microsoft.com/en-us/fabric/get-started/microsoft-fabric-overview | Fabric プラットフォーム概要とワークロード理解 |
| A9 | Fabric Governance / Adoption | https://learn.microsoft.com/en-us/power-bi/guidance/fabric-adoption-roadmap-governance | Fabric ガバナンス、採用ロードマップ |

## Secondary Sources（認知のみ）

直接の取得対象ではなく、変更認知のためにのみ参照します。

| Document | URL | Notes |
|----------|-----|-------|
| Azure Updates | https://azure.microsoft.com/en-us/updates/ | サービス変更/新機能の告知。ターゲット取得先ではない |

---

## Fetch Trigger — クエリを実行するタイミング

アーキテクチャ ガイダンス ドキュメントは**すべてのリクエストでクエリしません。** 以下のトリガーに該当する場合のみ、ターゲット取得を実行します。

### トリガー条件

0. **Phase 1 でユーザーのワークロード種別が特定されたとき（自動）**
   - 関連ワークロードの参照アーキテクチャを事前にクエリし、質問の深さを調整する
   - ユーザーが "best practice" などに言及しなくても自動で発火する
   - 目的: SKU/リージョン仕様の質問を超えて、公式アーキテクチャに基づく設計判断ポイントを質問に反映する
1. **ユーザーが設計方針の根拠提示を求めるとき**
   - "best practice", "reference architecture", "recommended structure", "baseline", "well-architected", "landing zone", "enterprise pattern" などのキーワード
2. **新しいサービス組み合わせのアーキテクチャ境界が曖昧なとき**
   - 既存の reference files/service-gotchas だけでは判定できないサービス間関係
3. **エンタープライズ レベルのセキュリティ/ガバナンス設計が必要なとき**
   - サブスクリプション構造、ネットワーク トポロジ、landing zone パターン

### トリガーが適用されない場合

- 単純なリソース作成（SKU/API バージョン/リージョンの質問）→ `azure-dynamic-sources.md` のみ使用
- domain-packs で既にカバーされているサービス組み合わせ → reference files を優先
- Bicep プロパティ値の検証 → `service-gotchas.md` または MS Docs Bicep reference

---

## Fetch Budget

| Scenario | Max Fetch Count |
|----------|----------------|
| デフォルト（トリガー発火時） | アーキテクチャ ガイダンス ドキュメント **最大 2 件** |
| 追加取得が許可される場合 | ドキュメント間で矛盾がある / 設計の中核的不確実性が残る / ユーザーがより深い根拠提示を明示的に要求 |
| 単純なデプロイ仕様の質問 | **0**（アーキテクチャ ガイダンスのクエリなし） |

---

## 質問タイプ別の判断ルール

| Question Type | Documents to Query | Design Decision Points to Extract | Documents NOT to Query |
|--------------|-------------------|----------------------------------|----------------------|
| RAG / チャットボット / Foundry アプリ | A5 または A6 + A7 | ネットワーク分離レベル、認証方式（managed identity vs key）、インデックス戦略（push vs pull）、監視範囲 | Architecture Center 全体を横断しない |
| エンタープライズ セキュリティ / ガバナンス / landing zone | A2 + A3 | サブスクリプション構造、ネットワーク トポロジ（hub-spoke など）、ID/ガバナンス モデル、セキュリティ境界 | AI/ML ドメイン文書は不要 |
| Fabric データ プラットフォーム | A8 + A9 | 容量モデル（SKU 選定基準）、ガバナンス レベル、データ境界（workspace 分離など） | AI 関連ドキュメントは不要 |
| 曖昧なサービス組み合わせ（パターン不明） | A1（ハブから最も近いドメイン文書を見つける）+ その文書 | 文書から特定される主要な設計判断ポイント | すべてのサブ文書を横断しない |
| 単純なリソース作成値（SKU/API/region） | クエリなし | — | すべてのアーキテクチャ ガイダンス |
| 一般的な AI/ML アーキテクチャ | A4（ハブ）+ 最も近い参照アーキテクチャ | 計算分離、データ境界、モデル提供アプローチ | 全体をクローリングしない |

---

## URL フォールバック ルール

1. デフォルトで `en-us` Learn URL を使用する
2. 特定 URL が 404 / redirect / deprecated の場合 → 親ハブページへフォールバック
   - 例: A5 が失敗した場合 → A4（AI/ML ハブ）で "foundry chat" キーワード検索
3. 親ハブでも見つからない場合 → A1（Architecture Center メイン）でタイトル キーワード検索
4. **URL が存在するという理由だけで、その内容を固定ルールとして扱わない**

---

## Full Traversal Prohibited

- Architecture Center のサブ文書を広範囲に横断（crawl）しない
- 質問タイプ別の判断ルールに従い、関連する 1〜2 文書のみターゲット取得する
- 取得した文書内でも関連セクションのみ参照し、文書全体を読まない
- 無制限の取得、再帰的リンク追跡、サブページ列挙は禁止

