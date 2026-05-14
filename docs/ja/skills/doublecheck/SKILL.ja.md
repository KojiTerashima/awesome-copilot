---
name: doublecheck
description: 'AI 出力のための 3 層検証パイプラインです。検証可能な主張を抽出し、Web 検索で裏付けまたは矛盾する情報源を探し、幻覚パターンに対する敵対的レビューを実行し、人手レビュー用にソース リンク付きの構造化検証レポートを生成します。'
---

# Doublecheck

AI が生成した出力に対して 3 層の検証パイプラインを実行します。目的はユーザーに何が真実かを断言することではなく、検証可能な主張をすべて抽出し、ユーザーが独立して確認できる情報源を見つけ、幻覚パターンに見えるものをフラグ化することです。

## Activation

Doublecheck は 2 つのモードで動作します。**active mode** (継続適用) と **one-shot mode** (必要時のみ) です。

### Active Mode

ユーザーが検証対象の具体的テキストを示さずにこの skill を呼び出した場合は、持続的な doublecheck mode を有効化します。次のように応答してください。

> **Doublecheck is now active.** I'll verify factual claims in my responses before presenting them. You'll see an inline verification summary after each substantive response. Say "full report" on any response to get the complete three-layer verification with detailed sourcing. Turn it off anytime by saying "turn off doublecheck."

その後、この会話の残りでは以下のルールすべてに従ってください。

**Rule: Classify every response before sending it.**

実質的な応答を生成する前に、それが検証可能な主張を含むかを判断してください。応答を次のように分類します。

| Response type | Contains verifiable claims? | Action |
|--------------|---------------------------|--------|
| Factual analysis, legal guidance, regulatory interpretation, compliance guidance, or content with case citations or statutory references | Yes -- high density | 完全な検証レポートを実行する (下記 high-stakes content rule を参照) |
| Summary of a document, research, or data | Yes -- moderate density | 主要な主張に対して inline verification を行う |
| Code generation, creative writing, brainstorming | Rarely | 検証は省略し、この種類の内容には doublecheck mode が適用されないことを注記する |
| Casual conversation, clarifying questions, status updates | No | 黙って検証を省略する |

**Rule: Inline verification for active mode.**

active mode が適用される場合、毎回 separate な完全検証レポートを生成してはいけません。代わりに、次のパターンで検証を応答内に埋め込んでください。

1. 通常どおり応答を生成する。
2. 応答の後に `Verification` セクションを追加する。
3. そのセクションで、検証可能な主張ごとに confidence rating と、あれば source link を列挙する。

Format:

```
---
**Verification (N claims checked)**

- [VERIFIED] "Claim text" -- Source: [URL]
- [VERIFIED] "Claim text" -- Source: [URL]
- [PLAUSIBLE] "Claim text" -- no specific source found
- [FABRICATION RISK] "Claim text" -- could not find this citation; verify before relying on it
```

active mode では速度を優先してください。citation、具体的な統計、または自信の低い主張について Web 検索を実行します。一般常識レベル、または高い自信がある主張まで検索する必要はありません。その場合は PLAUSIBLE と評価して進んでください。

いずれかの主張が DISPUTED または FABRICATION RISK と評価された場合は、verification セクションの前にそれを目立つ形で示してください。auto-escalation が適用される場合 (下記参照)、この呼びかけは summary table の前、full report の先頭に次の形式で置きます。

```
**Heads up:** I'm not confident about [specific claim]. I couldn't find a supporting source. You should verify this independently before relying on it.
```

**Rule: Auto-escalate to full report for high-risk findings.**

inline verification で DISPUTED または FABRICATION RISK の主張が **1 つでも**見つかった場合、inline verification を出力してはいけません。代わりに、応答先頭に "Heads up" の呼びかけを置き、その後 `assets/verification-report-template.md` のテンプレートを使った完全な 3 層検証レポートを生成してください。明らかな問題がある場合、ユーザーが詳細レポートを要求しなくても済むようにします。

**Rule: Full report for high-stakes content.**

応答に legal analysis、regulatory interpretation、compliance guidance、case citation、または statutory reference が含まれる場合は、常に `assets/verification-report-template.md` のテンプレートを使った完全検証レポートを生成してください。これらの内容では inline verification を使ってはいけません。省略版ではリスクが高すぎます。

**Rule: Discoverability footer for inline verification.**

inline verification (full report ではない) を出力するときは、verification セクションの末尾に必ず次の 1 行を追加してください。

```
_Say "full report" for detailed three-layer verification with sources._
```

**Rule: Offer full verification on request.**

ユーザーが "full report"、"run full verification"、"verify that"、"doublecheck that" などと言った場合は、完全な 3 層パイプラインを実行し、`assets/verification-report-template.md` を使って full report を生成してください。

### One-Shot Mode

ユーザーがこの skill を呼び出し、検証対象の具体的テキストを提供した場合 (または前の出力を参照した場合) は、完全な 3 層パイプラインを実行し、`assets/verification-report-template.md` を使った full verification report を生成してください。

### Deactivation

ユーザーが "turn off doublecheck"、"stop doublecheck" などと言った場合は、次のように応答してください。

> **Doublecheck is now off.** I'll respond normally without inline verification. You can reactivate it anytime.

---

## Layer 1: Self-Audit

対象テキストを批判的な視点で読み直します。この層の仕事は抽出と内部分析です。まだ Web 検索は行いません。

### Step 1: Extract Claims

対象テキストを文ごとに見て、検証可能な主張をすべて抽出します。各主張を次のように分類してください。

| Category | What to look for | Examples |
|----------|-----------------|---------|
| **Factual** | 事実や過去の出来事に関する断定 | "Python was created in 1991", "The GPL requires derivative works to be open-sourced" |
| **Statistical** | 数値、割合、数量 | "95% of enterprises use cloud services", "The contract has a 30-day termination clause" |
| **Citation** | 文書、判例、法律、論文、標準などの具体的参照 | "Under Section 230 of the CDA...", "In *Mayo v. Prometheus* (2012)..." |
| **Entity** | 人物、組織、製品、場所に関する主張 | "OpenAI was founded by Sam Altman and Elon Musk", "GDPR applies to EU residents" |
| **Causal** | X が Y を引き起こした、または X が Y につながるという主張 | "This vulnerability allows remote code execution", "The regulation was passed in response to the 2008 financial crisis" |
| **Temporal** | 日付、タイムライン、出来事の順序 | "The deadline is March 15", "Version 2.0 was released before the security patch" |

後続レイヤーで追跡できるよう、各主張に一時 ID (C1、C2、C3...) を割り当てます。

### Step 2: Check Internal Consistency

抽出した主張同士を見比べます。
- テキスト内で自己矛盾していないか? (例: 同じ出来事に対して 2 つの異なる日付が書かれている)
- 論理的に両立しない主張がないか?
- ある箇所で置いた前提を別の箇所で否定していないか?

内部矛盾があればすぐにフラグ化してください。これらは外部検証なしでも問題として特定できます。

### Step 3: Initial Confidence Assessment

各主張について、自分の知識だけに基づいて初期評価を行います。
- これは正しいと記憶しているか?
- モデルがよく hallucinate する種類の主張か? (具体的な citation、厳密な統計、正確な日付は高リスクです)
- 検証可能なほど具体的か、それとも反証不能なほど曖昧か?

初期 confidence は記録しますが、まだ finding として報告してはいけません。これは Layer 2 への入力であり、出力ではありません。

---

## Layer 2: Source Verification

抽出した各主張について外部証拠を探します。この層の目的は、ユーザーが自分で検証するために訪問できる URL を見つけることです。

### Search Strategy

各主張ごとに:

1. **Formulate a search query**: primary source が見つかる検索クエリを作ります。citation なら正確な title や case name、統計なら具体的な数値と topic、事実主張なら主要 entity と関係性を検索します。

2. **Run the search** using `web_search`: 最初の検索で relevant な結果が出ない場合は、表現を変えて 1 回だけ再試行します。

3. **Evaluate what you find:**
   - その主張を直接扱う primary または authoritative source が見つかったか?
   - credible な source から矛盾する情報が見つかったか?
   - 関連するものが何も見つからなかったか? (これ自体がシグナルです。本当に存在するものは通常 Web 上に痕跡があります)

4. **Record the result** with the source URL: source の内容を要約する場合でも、必ず URL を添えてください。

### What Counts as a Source

primary かつ authoritative な source を優先します。
- 公式ドキュメント、仕様書、標準
- 裁判記録、法令本文、規制文書
- 査読済み論文
- 公式組織サイトやプレス リリース
- 信頼できる参照資料 (百科事典、法情報 DB など)

source が secondary (ニュース記事、ブログ、wiki など) なのか、primary なのかを明記してください。ユーザーが重み付けを判断できます。

### Handling Citations Specifically

citation は hallucination の中でも最も高リスクなカテゴリです。特定の case、statute、paper、standard、document を引用している主張については:

1. 正確な citation (case name、title、section number) を検索する。
2. 見つかったら、引用された内容が本当にその主張どおりの意味を持つか確認する。
3. まったく見つからない場合は FABRICATION RISK としてフラグ化する。モデルは存在しない citation をもっともらしく生成しがちです。

---

## Layer 3: Adversarial Review

姿勢を完全に切り替えます。Layer 1 と 2 では出力を理解し検証しようとしていました。この層では **出力に誤りが含まれている前提で**、積極的に問題を探します。

### Hallucination Pattern Checklist

次の一般的パターンを確認してください。

1. **Fabricated citations** -- Layer 2 で見つけられなかった具体的な case、paper、statute を引用している。これは権威があるように見えるため、最も危険な hallucination パターンです。

2. **Precise numbers without sources** -- どこから来た数値か示さずに具体的な統計 (例: "78% of companies...") を述べている。モデルはもっともらしい統計を作りがちです。

3. **Confident specificity on uncertain topics** -- 実際には不確実または争いのある話題に対して、非常に具体的な断定をしている。専門家の間で意見が割れる分野の正確な日付、金額、帰属に注意してください。

4. **Plausible-but-wrong associations** -- 概念、判断、出来事を誤った entity に結び付けている。たとえば判決を誤った裁判所に帰属させたり、引用を誤った人物に帰したり、法律名は正しいのに条文内容を誤って説明していたりするケースです。

5. **Temporal confusion** -- 古い情報を current だと述べたり、出来事の順序を取り違えたりしている。

6. **Overgeneralization** -- 本来は特定の jurisdiction、context、time period にしか当てはまらないことを普遍的に真であるように述べている。legal や regulatory の内容でよく起こります。

7. **Missing qualifiers** -- 例外、制限、反論がある複雑な話題を、確定済みで単純なものとして提示している。

### Adversarial Questions

Layer 1 と 2 を通過した主要な主張ごとに、次を問います。
- この主張が間違いになる条件は何か?
- この分野でモデルが拾いやすい一般的誤解はあるか?
- 自分が subject matter expert なら、この表現に異議を唱えるか?
- これは学習データ cutoff の前後どちらの主張で、古くなっている可能性はあるか?

### Red Flags to Escalate

次のいずれかを見つけたら、レポートで目立つように示してください。
- どこにも見つからない具体的 citation
- 出典を特定できない統計
- authoritative source と矛盾する legal または regulatory claim
- 高い自信で述べられているが、実際には disputed または uncertain な主張

---

## Producing the Verification Report

3 つのレイヤーを終えたら、`assets/verification-report-template.md` を使ってレポートを作成します。

### Confidence Ratings

各主張に最終 rating を割り当てます。

| Rating | Meaning | What the user should do |
|--------|---------|------------------------|
| **VERIFIED** | 裏付け source が見つかりリンクできた | その主張が重要なら source link を spot-check する |
| **PLAUSIBLE** | 一般知識とは整合するが、specific source は見つからない | 妥当だが未確認として扱い、判断に使うなら独立に検証する |
| **UNVERIFIED** | 裏付けも反証も見つからなかった | 独立検証なしでこの主張に依存しない |
| **DISPUTED** | credible source から矛盾する証拠が見つかった | 矛盾する source を確認する。この主張は誤っている可能性がある |
| **FABRICATION RISK** | hallucination pattern に一致する (例: 見つからない citation、出典不明の精密統計) | primary source で確認できるまで誤りとみなす |

### Report Principles

- verdict ではなく link を提供する。何が真かを決めるのはユーザーであって、あなたではない。
- 矛盾する情報が見つかった場合は、両方を source 付きで提示する。勝者を決めてはいけない。
- 主張が反証不能 (曖昧すぎる、主観的すぎる) なら、そう明示すること。"Unfalsifiable" も有用な情報です。
- 何を検証できなかったかを明確にする。"I could not verify this" と "this is wrong" は違います。
- finding は重大度でグループ化する。最も注意が必要な項目から始める。

### Limitations Disclosure

レポートの最後には必ず次を含めてください。

> **Limitations of this verification:**
> - This tool accelerates human verification; it does not replace it.
> - Web search results may not include the most recent information or paywalled sources.
> - The adversarial review uses the same underlying model that may have produced the original output. It catches many issues but cannot catch all of them.
> - A claim rated VERIFIED means a supporting source was found, not that the claim is definitely correct. Sources can be wrong too.
> - Claims rated PLAUSIBLE may still be wrong. The absence of contradicting evidence is not proof of accuracy.

---

## Domain-Specific Guidance

### Legal Content

legal content は hallucination risk が高まります。理由は次のとおりです。
- case name、citation、holding はモデルにより頻繁に捏造される
- jurisdiction ごとの差異が平坦化または省略されやすい
- statute の文言が法的意味を変える形で言い換えられがち
- "majority rule" と "minority rule" の区別が失われやすい

legal content では、case citation、statutory reference、regulatory interpretation、jurisdiction に関する主張を特に厳しく確認してください。可能なら legal database を検索します。

### Medical and Scientific Content

- 引用された study が実在するか、結果が正確に説明されているか確認する
- 古い guideline を current として提示していないか注意する
- dosage、treatment protocol、diagnostic criteria はフラグ化する。これらは変化し、誤りが危険になり得る

### Financial and Regulatory Content

- 具体的な金額、日付、閾値を検証する
- regulatory requirement が正しい jurisdiction に帰属しており、現在有効か確認する
- 最近の法改正で古くなっている tax law claim に注意する

### Technical and Security Content

- CVE 番号、脆弱性の説明、影響バージョンを検証する
- API 仕様や構成手順が current documentation と一致するか確認する
- 古くなっている可能性のある version-specific な情報に注意する
