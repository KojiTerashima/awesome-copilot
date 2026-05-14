---
name: email-drafter
description: '自分の文体に合ったプロフェッショナルなメールの下書きとレビューを行います。WorkIQ を使って送信済みメールからトーン、挨拶、構成、結びのパターンを分析し、相手に応じた文脈付きドラフトを生成します。USE FOR: draft email、write email、compose email、reply email、follow-up email、analyze email tone、email style。'
---

# Email Drafter

確立された自分の文体とトーンに合うプロフェッショナルなメールを下書きします。WorkIQ を使って送信済みメールや受信者との過去のやり取りを分析し、レビューと調整ができる文脈付きドラフトを生成します。

## When to Use

- "Draft an email to [person] about [topic]"
- "Write a follow-up email to [customer] regarding [project]"
- "Reply to [person]'s email about [subject]"
- "Compose a proposal email for [initiative]"
- "Analyze my email tone with [recipient]"

## Workflow

### Step 1 — Gather Context

下書きの前に、次を収集します。

1. **Recipient(s)** — 誰宛てのメールか?
2. **Purpose** — どのようなメールか? (proposal、follow-up、technical guidance、introduction、status update など)
3. **Key points** — 何を伝える必要があるか?
4. **Relationship context** — 利用可能なら WorkIQ を使って受信者との過去のメール履歴を確認する

ユーザーがこれらを最初からすべて提供している場合は、そのまま進みます。そうでなければ確認質問をします (最大 3 つ)。

### Step 2 — Analyze Tone

受信者向けに下書きするときは、WorkIQ を使ってユーザーに定着しているコミュニケーション パターンを理解します。

1. 同じ受信者、または近い受信者に送った最近の送信済みメールを 3〜5 件取得する
2. 次のパターンを特定する:
   - **Greeting style** — formal ("Dear")、standard ("Hello")、casual ("Hi")、または direct (挨拶なし)
   - **Structure** — 短い段落か、箇条書きか、番号付きステップか
   - **Sign-off** — ユーザーが通常使う締めの言葉と名前の形式
   - **Formality level** — professional、friendly-professional、casual
   - **Language** — この受信者に対して通常どの言語で書いているか
3. それらのパターンをドラフトに適用する

WorkIQ が利用できない、または過去メールが存在しない場合は、妥当な professional default を使い、そのトーンは推定であることを明記します。

### Step 3 — Draft the Email

見つかった (または default の) スタイル ルールを適用します。

**Greeting:**
- Step 2 で見つかった greeting style に合わせる
- Default: 外部向けは "Hello [FirstName],"、内部向けは "Hi [FirstName],"
- 複数受信者の場合: "Hello [Name1], [Name2],"

**Tone:**
- 直接的で簡潔 — 無駄な言い回しは使わない
- フレンドリーだがプロフェッショナル
- すぐ本題に入る
- 適切な場合は主体的に支援を申し出る ("Happy to discuss further"、"Let me know if you need anything")

**Structure:**
- 短いメール (1〜2 点): 単純な段落で十分、箇条書きは不要
- 長めのメール (proposal、複数項目の update): 箇条書きまたは番号付きリストを使う
- relevant な場合は過去会話の文脈を含める ("Following our recent conversation about...")

**Sign-off:**
- Step 2 で見つかったユーザーの sign-off パターンに合わせる
- Default: "Best regards," の次行にユーザーの first name

**Language:**
- ユーザーが別指定しない限り英語を既定とする
- 過去のやり取りが別言語なら、受信者の言語に合わせる

### Step 4 — Output

1. 適用した tone/style の簡単な注記とともに、レビュー用ドラフトを提示する
2. ユーザーの依頼に応じて編集を反映し、満足するまで反復する
3. 最終ドラフトを `outputs/<year>/<month>/` に分かりやすいファイル名で保存する (例: `2026-03-26-email-acme-followup.md`)

## Important Rules

- **Never send emails** — 下書きだけをファイルとして作成し、送信はユーザーが手動で行う
- 受信者との過去文脈が利用できる場合は、必ず WorkIQ を確認する
- ユーザーが "draft email" または "write email" と言ったら、この skill を自動で有効化する
- 下書きは `outputs/<year>/<month>/` のフォルダー規約で保存する
- プライバシーを尊重し、無関係なメール スレッドの機密情報は含めない

## Example Prompts

- "Draft an email to Sarah about the project timeline"
- "Write a follow-up to the customer about their migration questions"
- "Compose a proposal email for the new training initiative"
- "Reply to John's email — agree with his approach but suggest we add monitoring"
- "Analyze my email tone with the Acme team"

## Requirements

- トーン分析と受信者文脈のために **WorkIQ MCP tool** を推奨します (Microsoft 365 / Outlook)
- WorkIQ がなくても skill は動作しますが、個別最適なトーンではなく professional default を使います
- 出力は workspace 内の markdown file として保存されます
