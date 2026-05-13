# Step 1b: Eval Criteria

すでに文書化した entry point (`01-entry-point.md`) を踏まえて、このアプリで重要な quality dimension を定義します。

このドキュメントには 2 つの目的があります。

1. **Dataset creation (Step 4)**: use case が、どの種類の item を生成すべきかを決めます。各 use case は dataset 内に代表的 item を持つべきです。
2. **Evaluator selection (Step 3)**: eval criteria が、どの evaluator を選び、どう対応付けるかを決めます。

これは包括仕様書ではなく planning artifact なので、簡潔に保ってください。

---

## What to define

### 1. Use cases

アプリが扱う distinct な scenario を列挙します。各 use case は dataset item のカテゴリになります。**各 use case の説明は、(a) 入力が何か、(b) 期待される振る舞いまたは結果が何か、の両方を伝える簡潔な 1 行でなければなりません。** アプリに不慣れな人でも scenario と成功条件を理解できる程度には具体的である必要があります。

**Good use case descriptions:**

- "Reroute to human agent on account lookup difficulties"
- "Answer billing question using customer's plan details from CRM"
- "Decline to answer questions outside the support domain"
- "Summarize research findings including all queried sub-topics"

**Bad use case descriptions (too vague):**

- "Handle billing questions"
- "Edge case"
- "Error handling"

### 2. Eval criteria

**このアプリ固有の高レベルな eval criteria** を定義します。各 criterion は Step 3 で evaluator に対応付けられます。

**良い criterion はアプリの目的に具体的に結び付いています。** 例:

- Voice customer support agent: "Does the agent verify the caller's identity before transferring?", "Are responses concise enough for phone conversation?"
- Research report generator: "Does the report address all sub-questions?", "Are claims supported by retrieved sources?"
- RAG chatbot: "Are answers grounded in the retrieved context?", "Does it say 'I don't know' when context is missing?"

**悪い criterion は、generic な evaluator 名を要件らしく言い換えただけのものです。** "Factual accuracy" や "Response relevance" ではなく、このアプリにおいて factual accuracy や relevance が何を意味するかを言ってください。

この段階では evaluator class や threshold を選ばないでください。それは Step 3 で行います。

### 3. Check criteria applicability and observability

各 criterion について:

1. **Determine applicability scope** — その criterion は全 use case に適用されるのか、一部だけなのかを決めます。ある criterion が特定 scenario にしか関係しない場合 (例: "identity verification" は account-related request にだけ関係し、general FAQ には関係しない) は、明確に示してください。これは Step 4 (dataset creation) にとって重要です。なぜなら:
   - **Universal criteria** → dataset-level default evaluator になる
   - **Case-specific criteria** → 該当行だけの item-level evaluator になる

2. **Verify observability** — その criterion を評価するために、アプリ内のどの data point を `wrap()` で捕捉する必要があるかを特定します。これが Step 2 の wrap coverage を決めます。
   - criterion が最終応答に関するものなら → `wrap(purpose="output", name="response")` で捕捉
   - routing decision に関するものなら → `wrap(purpose="state", name="routing_decision")` で捕捉
   - アプリが取得・利用した data に関するものなら → `wrap(purpose="input", name="...")` で捕捉

---

## Output: `pixie_qa/02-eval-criteria.md`

調査結果をこのファイルに書いてください。**短く保つこと**。以下の template が最大長です。

### Template

```markdown
# Eval Criteria

## Use cases

1. <Use case name>: <入力 + 期待挙動を伝える 1 行>
2. ...

## Eval criteria

| #   | Criterion | Applies to    | Data to capture |
| --- | --------- | ------------- | --------------- |
| 1   | ...       | All           | wrap name: ...  |
| 2   | ...       | Use case 1, 3 | wrap name: ...  |
```
