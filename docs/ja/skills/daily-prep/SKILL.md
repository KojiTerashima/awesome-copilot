---
name: daily-prep
description: '翌日の会議とタスクに備えます。WorkIQ 経由で Outlook の予定表を取得し、未完了タスクやワークスペース文脈と突き合わせ、会議を分類し、競合やその日の適合性の問題を検出し、学習時間と深い集中作業の枠を見つけ、生産性向上の提案付きで構造化された HTML の準備ファイルを生成します。'
---

# Daily Prep

次の稼働日に向けて、会議詳細、準備用の箇条書き、関連タスク、生産性向上の提案を含む構造化された prep ファイルを生成してください。

## When to Use

- 終業時: "prepare me for tomorrow"
- 任意のタイミング: "prep me for Friday" または "what does March 25 look like?"
- 週間計画: 複数日分に対して実行する

## Procedure

### 1. Determine Target Date

ユーザーが日付を指定した場合はそれを使います。そうでなければ翌日 (current date + 1 day) を既定値とします。
翌日が土曜日なら月曜日、日曜日でも月曜日を既定値とします。
出力パスを計算します: `outputs/YYYY/MM/YYYY-MM-DD-prep.html`

### 2. Pull Calendar via WorkIQ

WorkIQ MCP ツールを使って予定表を取得します。WorkIQ には次のように尋ねます。

> "What meetings do I have on {target date}? For each meeting, include: subject, start time, end time, organizer, all attendees with their email addresses, location, whether it's online, and whether I've accepted or declined."

応答が不十分であれば、追加で次の問い合わせを行います。

> "For the meetings on {target date}, which ones are marked as optional or tentative? Which ones are recurring?"

### 3. Classify Each Meeting

参加者ドメインと件名に基づいて、次のラベルを適用します。

| Label | Criteria |
|-------|----------|
| `[Customer · HIGH]` | 顧客/パートナーのドメインを持つ外部参加者がいる、または件名が既知の顧客名に一致する |
| `[Internal]` | 社内ドメインの参加者のみ |
| `[Community]` | CoP、community、guild、learning session |
| `[Upskilling]` | training、workshop、certification、learning |
| `[Optional · skip]` | tentative、重要度が低い、または既知の定例任意参加 (例: "Office Hours", "Open Q&A") |
| `[Personal]` | 非公開イベント、業務外 |

#### Zone Markers

各会議について organizer フィールドを確認し、次の追加マーカーを適用します。

| Condition | Marker | Action |
|-----------|--------|--------|
| Starts ≥ 15:30 and < 16:00 (any organizer) | `⚠️ After-hours` | decline を推奨 |
| Starts ≥ 16:00 and **not** self-organized | `⚠️ After-hours` | decline を推奨 |
| Starts ≥ 16:00 and self-organized | _(no flag)_ | OK — 自分で予定したため問題なし |
| Before 09:00 and **not** self-organized | `⚠️ Early` | decline を推奨 — 学習時間帯を侵食する |
| Before 09:00 and self-organized | _(no flag)_ | OK — 自分で予定したため問題なし |
| Overlaps 12:00–13:00 | `🍽️ Lunch conflict` | Calendar Notes に記載 |

"Self-organized" とは、**you** がその会議の organizer であることを意味します (WorkIQ の organizer フィールドを確認してください)。

### 4. Ideal Day Structure

すべての分析ステップで、これを判断フレームワークとして使います。すべての会議はこれらのゾーンに照らして評価しなければなりません。ユーザーは自分のルーティンに合わせて時間帯や目標を調整してかまいません。

| Zone | Time | Purpose | Rules |
|------|------|---------|-------|
| Morning Focus | Before 09:00 | 事務、学習、個人作業 | 他者の会議から保護する。外部イベントはフラグを付ける。 |
| Customer Zone | 09:00–12:00 | 顧客/外部会議 | 顧客会議は最大 2 件。外部通話は午前を優先。 |
| Lunch | 12:00–13:00 | 休憩 | 保護対象。重複はフラグ化する。 |
| Deep Work | 13:00–15:30 | 成果物作成、集中したコーディング/執筆 | 会議を最小化する。不要不急の会議は deep work の妨害としてフラグ化する。 |
| Protected (strict) | 15:30–16:00 | 終業前のクールダウン | organizer に関係なくすべての会議にフラグを付ける。 |
| Protected (flex) | 16:00+ | 終業時間帯 | 他者の会議のみフラグ化。self-organized は許容。 |

**Targets per day:**
- Learning hours: **1.5h** (morning focus + gap time から確保)
- Deep work hours: **2.5h** (13:00–15:30 zone)
- Customer meetings: **max 2** (できれば 09:00–12:00)

### 5. Detect Conflicts & Day Fit Issues

イベントの時間帯を比較し、Conflicts テーブルに重複をフラグ化して、それぞれに推奨対応を付けます。優先順位は customer meeting を internal/optional より上にします。

さらに次の **day fit issues** も検出し、別の "Day Fit Issues" テーブルに出力します。

| Check | Condition | Flag |
|-------|-----------|------|
| **Customer overload** | >2 `[Customer · HIGH]` meetings | 3 件目以降に "Consider rescheduling to another day" |
| **Deep work disruption** | 13:00–15:30 zone の不要不急会議 | "Disrupts deep work — consider moving to morning" |
| **Non-ideal placement** | 09:00–12:00 以外の customer meeting | "Customer meeting outside preferred morning zone" |
| **Early intrusion** | 09:00 前の他者主催会議 | "Intrudes on learning window — recommend decline" |
| **Lunch conflict** | 12:00–13:00 と会議が重複 | "Conflicts with lunch break" |

### 6. Gather Context from Workspace

1. 翌日の会議に含まれる顧客名や参加者に関連する open task file を読む
2. それらの顧客やトピックに関連する最近のファイルを workspace folder から検索する
3. 関連する準備文脈として、最近の meeting summary や plan を確認する
4. これらを使って各会議向けの実行可能な prep bullet を生成する

### 7. Generate Prep per Meeting

各会議について (時系列順):
- 時刻、subject、organizer
- 参加者一覧 (名、外部なら company)
- open task、最近の summary、meeting subject に基づく 3〜5 個の実行可能な prep bullet
- 文脈がなければ、会議で確認/質問すべき点を記載する

### 8. Find Learning & Focus Slots

会議ごとの prep を生成した後、その日の予定を分析して空き枠を見つけます。

1. **Morning Focus confirmation** — morning focus の時間帯が空いているか確認する。そこに self-organized ではないイベントがあればフラグ化する。

2. **Learning Slots** — morning window 内の 30 分以上の空き、および upskilling に適した他の空き枠を見つける。目標: **1.5h/day**。各枠について time range、duration、推奨 activity を示す。

3. **Deep Work Blocks** — 13:00–15:30 zone 内の連続した空き時間を見つけ、deliverable 作業に充てる。各ブロックについて time range、duration、open tasks からの推奨 task を示す。

4. **Report totals:**
   - 見つかった learning hours と 1.5h 目標の比較 (例: "1.0h / 1.5h target — 0.5h short")
   - 13:00–15:30 に確保できる deep work 時間 (例: "2.0h / 2.5h available")

### 9. Productivity Recommendations

1 日全体を分析し、次を提供します。

| Section | What to Include |
|---------|------------------|
| **Day Fit Score** | その日が Ideal Day Structure にどれだけ合っているかを 0–100% で評価する。基準: (1) morning focus が空いている (+20%)、(2) 09:00–12:00 に customer meeting が 2 件以下 (+20%)、(3) lunch 12:00–13:00 が保護されている (+15%)、(4) deep work 13:00–15:30 が維持されている (+20%)、(5) 15:30 以降に何もない、または 16:00 以降は self-organized のみ (+15%)、(6) 1h 以上の learning slot を確保 (+10%)。表示形式: 🟢 ≥80%、🟡 50–79%、🔴 <50%。 |
| **Day Shape** | 総会議時間、利用可能な focus time、learning hours、deep work hours、heavy/moderate/light 判定 |
| **Decline Candidates** | 自動対象: (1) 15:30–16:00 の全会議、(2) 16:00 以降の他者会議、(3) 09:00 前の他者会議、(4) 3 件目以降の customer meeting、(5) deep work zone 中の optional meeting。"Reclaim" 列には回収できる分数を示す。09:00 前または 16:00 以降の self-organized meeting は自動 decline 対象から **除外**。 |
| **Conflict Resolution** | 各重複に対する具体的な推奨対応 |
| **Learning Slots** | upskilling 用の空き枠 — Step 8 より。テーブル: Window、Duration、Suggested Activity。合計と 1.5h 目標も示す。 |
| **Deep Work Blocks** | deliverable 向けの 13:00–15:30 の空き枠 — Step 8 より。テーブル: Window、Duration、Suggested Task。 |
| **Energy Management** | 休憩なしで 3h を超える連続 customer meeting がある場合はフラグ化 |
| **Top 3 Priorities** | 達成すべき最も重要な 3 項目 (会議 + タスク) |

### 10. Write the File

`outputs/YYYY/MM/YYYY-MM-DD-prep.html` に、埋め込み CSS を持つ自己完結型 HTML ファイルとして出力してください (dark theme、color-coded timeline、responsive layout)。

その日付のファイルが既に存在する場合は、上書きではなく先に読み込んで更新してください。ユーザーが手動メモを追加している可能性があります。

## Example Prompts

- "Prepare me for tomorrow"
- "What does Friday look like?"
- "Daily prep for March 28"
- "Prep me for next Monday — focus on customer meetings"

## Requirements

- カレンダーアクセスには **WorkIQ MCP tool** が必要です (Microsoft 365 / Outlook)
- 文脈補強のために task file と customer/project folder を含む workspace が必要です
- 出力は自己完結型 HTML です — 外部依存はありません
