---
name: gtm-partnership-architecture
description: 収益とプラットフォーム採用を推進するパートナーエコシステムを構築し、スケールさせる。パートナープログラムをゼロから立ち上げるとき、パートナーのティア設計、共同マーケティングの運用、Build-vs-Partnerの判断、またはcrawl-walk-run型の段階的導入を設計するときに使用。
license: MIT
metadata:
  author: Smit Patel (https://linkedin.com/in/smitkpatel)
  source: https://github.com/beingsmit/technical-product-gtm
---

# Partnership Architecture

収益とプラットフォーム採用を推進するパートナーエコシステムを構築し、スケールさせるための指針です。これは理論ではありません。8桁ARRを生んだパートナープログラムの実践と、実際に経済的コミットメントを伴う提携の観察から得たパターンです。

## When to Use

**Triggers:**
- 「パートナープログラムはどう構築すべき？」
- 「これは自社で作るべきか、提携すべきか？」
- 「パートナー主導と直販、どちらの営業モーションがよいか」
- 「エコシステム戦略」
- 「パートナーをどう獲得し、どうティア分けするか」
- 「パートナーとの共同マーケティング」
- 「提携が本当に意味を持つのはいつか？」

**Context:**
- 提携プログラムをゼロから構築する（0→1）
- 既存プログラムを拡大する（1→100）
- Build vs Partnerの意思決定を評価する
- パートナー契約と経済条件を設計する
- パートナー起点のGTMモーションを計画する

---

## Core Frameworks

### 1. Real Partnerships Require Skin in the Game

**The Pattern:**

多くの「提携」は共同マーケの見せかけです。共同ウェビナー、ロゴ交換、プレスリリース。経済的コミットメントがない。真の当事者意識（skin in the game）がない。

本物の提携は違います:
- 経済的コミットメント（支出、レベニューシェア、共同投資）
- プロダクトロードマップの整合（提携のための機能開発）
- エグゼクティブスポンサーシップ（経営層が四半期ごとに関与）
- 相互リスク（うまくいかなければ双方が失敗し得る）

**How to Tell the Difference:**

こう問います: 「この提携が失敗したら、双方は何を失うか？」

答えが「何もない」なら、それは提携ではありません。単なる握手です。

私が見てきた最良の提携は、双方にとって居心地の悪いコミットメントを伴っていました。複数年のクラウド利用コミット、専任エンジニアリングチーム、売上保証。居心地の悪さこそが要点で、双方に「提携を成功させる」圧力を生みます。

**Framework: Three-Sided Value Proposition**

成功する提携は、常に次の3者に明確な価値を生みます。

**Your Company:**
- 流通・販路（パートナー顧客へのアクセス）
- 信頼性（認知度の高いブランドとの連携）
- 収益（直接または間接）
- プロダクトレバレッジ（自社で作らない機能の獲得）

**The Partner:**
- 売上または利益率の改善
- 顧客維持・スティッキー化
- 競争上の差別化
- サポート負荷の軽減

**Shared Customers:**
- ワークフロー改善
- 連携時の摩擦低減
- 単一ベンダー関係
- コスト効率

**Decision Criteria:**

提携を進める前に、次を答える:

1. 当社の経済的コミットメントは何か？（Engリソース、支出、レベニューシェア？）
2. パートナーの経済的コミットメントは何か？（相手も投資するか？）
3. 失敗したら何が起きるか？（双方が実質的に何かを失うか？）

双方がコストゼロで撤退できるなら、**それは提携ではなく、ただの握手です。**

**Common Mistake:**

「提携」をマーケ発表として扱うこと。連携リリース、共同ウェビナー、共同ブランドコンテンツ。これらは話題は生むが、事業は作りません。本物の提携には居心地の悪いコミットメントが必要です。

---

### 2. Ecosystem Control = Discovery, Not Gatekeeping

**The Developer Marketplace Decision:**

急成長期のプラットフォーム企業でエコシステムを運営していたとき、経営会議で議論になりました。ネットワークを誰にでも開放するか、品質のためにキュレーションするか。

**Quality control camp:** 「ゲートキーピングが必要。でないとSEOスパム、低品質API、ブランド毀損が起きる。」

**Open network camp:** 「開発者はゲートキーパーを迂回する。品質管理よりネットワーク効果の方が重要。」

**The decision:** 開放型を選択。品質懸念は現実的でしたが、こう賭けました: **コントロールは、投稿前の審査ではなく、発見性 + 信頼レイヤーで作る。**

**What We Built Instead of Gatekeeping:**

1. **Search and discovery** - アルゴリズムで高品質APIを上位表示
2. **Trust signals** - 認証バッジ、利用統計、健全性スコア
3. **Community curation** - ユーザー評価、コレクション、推薦
4. **Moderation** - 公開前に止めるのでなく、公開後にスパムを除去

**Result:** ネットワーク効果が勝ちました。数千のAPIが公開され、品質は事前審査ではなく利用実績で浮上しました。

**The Pattern:**

**Curated ecosystem (Gatekeeper Model):**
- Pros: 高品質、ブランド統制
- Cons: 成長が遅い、パートナー摩擦、ボトルネック化

**Open ecosystem (Discovery Model):**
- Pros: ネットワーク効果、急成長、セルフサービス
- Cons: 品質のばらつき、モデレーション負荷

**When to Use Which:**

```
低品質パートナー参加時のブランド毀損リスクは高いか？
├─ Yes（規制産業、セキュリティ重視）→ Curated
└─ No → Continue...
    │
    人手レビューをスケールできるか？
    ├─ No（潜在パートナーが数千）→ Open
    └─ Yes（パートナーが数十）→ Curated
```

**Common Mistake:**

「品質管理が必要だから」とCuratedを初期設定にしてしまうこと。10社なら機能しますが、100社を超えると自分たちがボトルネックになります。代わりに発見性と信頼の仕組みを作るべきです。

---

### 3. Partnership Tactics > Partnership Theater

**The Certification Wedge:**

クラウド提携の初期、チャネルレバレッジを探していました。対象はMSP（マネージドサービスプロバイダー）。

**The insight:** クラウドプロバイダーのパートナープログラム要件に埋もれた一文: 「認定スタックに[当社プロダクトカテゴリ]を含めること。」

**The play:** 提携提案全体を、その一文を軸に構築。MSPは当社製品を「欲しい」のではなく、認定維持のために**必要**でした。

**Result:** 当社は「あるとよい」ではなく「必須」になり、一般的な提携より3倍速くMSP案件を獲得しました。

**Framework: Partnership Leverage Types**

**1. Requirement leverage**（最強）
- 相手が認証/コンプライアンス/提携ステータス維持のためにあなたを必要とする
- 例: クラウド認定があなたのカテゴリ製品を必須化
- 見つけ方: パートナープログラム要件、マーケットプレイス規約を読む

**2. Economic leverage**（強い）
- 相手の収益改善またはコスト削減に直結
- 例: パートナーのサポートコストを30%削減
- 測り方: 相手のP/L観点でROIを算出

**3. Competitive leverage**（中程度）
- 相手に競争優位の差別化を与える
- 例: 6か月の独占連携
- 検証法: 「競合も欲しがるか？」と問う

**4. Customer leverage**（中程度）
- 相手顧客がその連携を強く求める
- 例: 連携要望のサポートチケットが50件超
- 測り方: パートナー側の問い合わせ件数

**5. Co-marketing leverage**（弱い）
- 共同コンテンツ、イベント、ロゴ交換
- 例: 共同ブランドのウェビナー
- 現実: あるとよいが、成約を決めることは稀

**How to Apply:**

**提携提案前に、自社レバレッジを特定する:**

高レバレッジ（要件・経済）→ フル投資
中レバレッジ（競争・顧客）→ 軽量提携で先に検証
低レバレッジ（共同マーケのみ）→ 実施しない（時間の無駄）

**The Qualification Question:**

「この提携をしなかった場合、あなたに何が起こるか？」

- 「クラウド認定を失う」→ 高レバレッジ、推進
- 「一部顧客を失うかもしれない」→ 中程度、慎重に検証
- 「特に変わらない」→ レバレッジなし、撤退

**Common Mistake:**

相手メリットではなく、自社メリット中心で提案すること。「御社顧客にアクセスしたい」は共同マーケの見せかけ。「クラウド認定を維持できます」はレバレッジです。

---

### 4. Partner Tiering: Three-Tier Model

パートナープログラムは、コミットメントと能力に基づき明確なティアで設計します。

**Tier 1: Integration Partner (Self-Serve)**
- パートナーが公開API/docsで自力実装
- 提供するもの: documentation, Slack channel, office hours
- プロモーションはパートナー主導
- Timeline: 2-6 months
- Best for: エンジニアリングリソースを持つ意欲的なパートナー

**Tier 2: Partnership Partner (Joint Development)**
- 共同開発による統合
- 提供するもの: dedicated channel, regular syncs, product input
- プラットフォーム側が共同マーケ支援
- Timeline: 6-12 months
- Best for: 戦略適合が高く、統合品質を加速したいパートナー

**Tier 3: Strategic Partner (Co-Development)**
- プロダクトロードマップまで踏み込んだ深い統合
- 提供するもの: dedicated partner manager, executive relationship
- カスタマイズした共同マーケ、収益目標
- Timeline: Ongoing
- Best for: ポジショニングを変える看板級パートナーシップ

**Decision Criteria:**
- 戦略適合 AND パートナー能力でティアを決める
- 過剰に高ティア化しない（満たせない期待を生む）
- ティア間の昇格パスを明確にする

**Common Mistake:**

全パートナーを同じ扱いにすること。Tier 1はセルフサービスを望み、Tier 3は手厚い支援を望む。ミスマッチは不満を生みます。

---

### 5. Crawl-Walk-Run Partnership Deployment

フルコミット前に段階的検証を行い、提携リスクを下げます。

**Crawl (4-8 weeks):**
- 両ソリューションを使うパイロット顧客1-2社
- 手動または軽量連携（本番品質ではない）
- 特定成果を測定: 時間短縮、採用率、売上インパクト
- Go/no-go: 指定指標で20%以上改善

**Walk (8-12 weeks):**
- 追加で5-10社
- 正式な統合を構築
- Co-marketing: 共同発表、ウェビナー
- Sales enablement: トレーニング、プレイブック
- Go/no-go: 招待顧客の採用率70%以上

**Run (6-12 months ongoing):**
- 本格展開
- 共同エンタープライズ営業、統合カスタマーサクセス
- APIs/native integrations, marketplace listing
- 四半期ビジネスレビュー、経営ステアリング

**The Pattern:**

多くの提携はCrawl段階で失敗します。これは良いことです。最小投資で速く学べます。

**Common Mistakes:**
- Crawlを飛ばす（いきなりフルコミット）
- フェーズを並行実行（混乱し、シグナル分離不可）
- 価値を出さない提携を継続（サンクコストの罠）
- 明確なGo/no-goなしで次フェーズへ進む

**Go/No-Go Criteria:**

**After Crawl:**
- パイロット顧客は20%以上改善したか？
- 同業へ推奨するか？
- この統合をスケールできるか？

**After Walk:**
- 招待顧客の70%以上が採用したか？
- パートナーは積極的に販促しているか？
- サポート負荷は管理可能か？

**Enter Run Only If:**
- CrawlとWalkの基準を両方通過
- 双方が次フェーズにコミット
- ROIモデルがスケール時にも成立

---

### 6. Partnership Value Exchange Clarity

各当事者の獲得価値を説明できなければ、提携は失敗します。

**Partnership Charter (Required Before Launch):**

**Mutual Goals:**
- 当社にとって成功とは何か？
- パートナーにとって成功とは何か？
- 顧客にとって成功とは何か？

**Value Exchange:**
- 当社が提供するもの（engineering time, co-marketing, revenue share）
- パートナーが提供するもの（distribution, credibility, co-investment）
- これは均衡しているか？（相手が離脱しても双方が実行したい内容か？）

**Timeline:**
- Crawl phase（dates, deliverables, metrics）
- Walk phase（dates, deliverables, metrics）
- Run phase（ongoing cadence, QBRs）

**Measurement:**
- 成功の具体指標（revenue, customers, retention）
- 追跡方法（dashboard, reports, reviews）
- レビュー頻度（monthly? quarterly?）

**Governance:**
- 双方で誰が意思決定を担うか？
- 争点のエスカレーション経路
- 終了基準（何をもって提携終了とするか？）

**The Signature Test:**

双方がチャーターに署名すべきです。どちらかが文書化コミットを拒むなら、本物の提携ではありません。

**Common Mistake:**

文書のない口頭合意。状況が厳しくなったとき（必ずなります）、書面での整合が必要です。

---

### 7. Co-Marketing Execution Checklist

**Pre-Launch (4-6 weeks before):**
- [ ] Joint value prop finalized（双方マーケチームレビュー済み）
- [ ] Customer case study identified（理想は2-3候補）
- [ ] Technical integration validated（ローンチ当日の不具合なし）
- [ ] Sales enablement ready（one-pager, deck, demo）
- [ ] Support trained（双方チームがチケット対応方法を把握）
- [ ] Marketplace listings prepared（該当時）

**Launch Week:**
- [ ] Press release（公開タイミングを調整）
- [ ] Blog posts（両社）
- [ ] Joint webinar scheduled（ローンチ後2週間以内）
- [ ] Social media campaign（ハッシュタグを共同設計）
- [ ] Sales teams briefed（ライブ研修）
- [ ] Customer comms sent（該当セグメントへメール）

**Post-Launch (Weeks 2-8):**
- [ ] Customer adoption tracked（週次ダッシュボード）
- [ ] Support issues triaged（共同Slack channel）
- [ ] Case study published（定量成果付き）
- [ ] Pipeline impact measured（影響商談）
- [ ] Quarterly business review scheduled

**Common Mistake:**

ローンチをゴールとみなすこと。本当の仕事はローンチ後に始まります。採用、サポート、改善です。

---

## Decision Trees

### Should We Build or Partner?

```
この機能は自社プロダクト差別化のコアか？
├─ Yes → 自社でBuild
└─ No → Continue...
    │
    これを自社開発するとロードマップが6か月超遅延するか？
    ├─ Yes → Partner
    └─ No → Continue...
        │
        自社を必要としてくれる有力パートナーは存在するか？
        ├─ Yes → Partner
        └─ No → Build
```

### Which Partner Tier?

```
パートナーはセルフサーブ実装できるエンジニアリングリソースを持つか？
├─ Yes → Tier 1から開始し、6か月後にTier 2を評価
└─ No → Continue...
    │
    そのパートナーは自社のポジショニングを変える看板ロゴか？
    ├─ Yes → Tier 3 (Strategic)
    └─ No → Tier 2 (Joint Development)
```

### Should We Continue This Partnership?

```
Crawl phaseは成功基準を満たしたか？
├─ No → 提携終了、失敗から学ぶ
└─ Yes → Continue...
    │
    Walk phaseは成功基準を満たしたか？
    ├─ No → 提携終了、または変更してCrawlを再実施
    └─ Yes → Run phaseへ移行
```

---

## Common Mistakes

1. **提携をプラットフォーム拡張ではなく販売チャネルとして扱う**
   - 提携は「誰が買うか」だけでなく「何ができるか」を拡張すべき

2. **明確な統合導線なしでローンチする**
   - ステップごとのガイドがないと、パートナーは躓いて失敗する

3. **パートナーが自走で販促すると期待する**
   - 共同マーケ用テンプレート、素材、支援の提供が必要

4. **ティアを増やしすぎる**
   - 最適は2-3。増やすほど混乱と期待ミスマッチを招く

5. **ローンチ後に放置する**
   - 関係は継続的な育成が必要。定期接点を予定化する

6. **見栄のための提携を追う**
   - ブランド名や資金コネクションは顧客価値と同義ではない

7. **終了基準がない**
   - 失敗の定義と、優先度を下げるタイミングを事前に定義する

---

## Quick Reference

**Before starting any partnership:**
- [ ] Three-sided value prop articulated
- [ ] Partner tier identified
- [ ] Crawl phase scope defined
- [ ] Success metrics agreed
- [ ] Partnership charter drafted

**Before launching any partnership:**
- [ ] Customer ready criteria met
- [ ] Co-marketing checklist complete
- [ ] Sales team briefed
- [ ] Health management cadence scheduled

**Partnership leverage hierarchy:**
1. Requirement（認証/コンプラのために相手があなたを必要）
2. Economic（相手の収益改善/コスト削減）
3. Competitive（相手の差別化）
4. Customer（相手顧客の需要）
5. Co-marketing（あるとよいが決定打にはなりにくい）

**Go/no-go criteria:**
- Crawl: 顧客成果の20%以上改善
- Walk: 採用率70%以上
- Run: 両フェーズ通過 + ROI検証済み

---

## Related Skills

- **developer-ecosystem**: 開発者向けエコシステムプログラム
- **enterprise-account-planning**: パートナーを含むエンタープライズ案件管理
- **technical-product-pricing**: 提携案件の価格設計

---

*ハイパーグロース期の複数プラットフォーム企業での提携実務に基づく。開放型 vs キュレーション型の開発者マーケットプレイス運営判断や、クラウドプロバイダー認定要件を活用したチャネル成長を含む。理論ではなく、実際に収益とプラットフォーム採用を生んだ提携パターン。*

