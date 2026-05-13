# Verification Report

## Summary

**検証対象テキスト:** [何を確認したかの簡潔な説明]
**抽出した主張数:** [合計 N 件]
**内訳:**

| Rating | Count |
|--------|-------|
| VERIFIED | |
| PLAUSIBLE | |
| UNVERIFIED | |
| DISPUTED | |
| FABRICATION RISK | |

**要注意項目:** [DISPUTED または FABRICATION RISK と評価された件数]

---

## Flagged Items (Review These First)

DISPUTED または FABRICATION RISK と評価された項目です。source material に依存する前に確認が必要です。

### [C#] -- [主張の簡潔な説明]

- **Claim:** [対象テキスト中の具体的な断定]
- **Rating:** [DISPUTED または FABRICATION RISK]
- **Finding:** [検証で何が見つかったか。どこが誤っているか、または疑わしいか]
- **Source:** [矛盾する、または relevant な source の URL]
- **Recommendation:** [ユーザーが取るべき行動。例: "Verify this citation in Westlaw"、"Remove this statistic unless you can find a primary source"]

---

## All Claims

抽出したすべての主張について、confidence rating ごとに grouped した完全結果です。

### VERIFIED

#### [C#] -- [簡潔な説明]
- **Claim:** [主張内容]
- **Source:** [URL]
- **Notes:** [source に関する relevant な文脈]

### PLAUSIBLE

#### [C#] -- [簡潔な説明]
- **Claim:** [主張内容]
- **Notes:** [なぜ verified ではなく plausible としたか]

### UNVERIFIED

#### [C#] -- [簡潔な説明]
- **Claim:** [主張内容]
- **Notes:** [何を検索したか、なぜ見つからなかったか]

### DISPUTED

#### [C#] -- [簡潔な説明]
- **Claim:** [主張内容]
- **Contradicting source:** [URL]
- **Details:** [source が述べている内容と主張の差異]

### FABRICATION RISK

#### [C#] -- [簡潔な説明]
- **Claim:** [主張内容]
- **Pattern:** [どの hallucination pattern に一致するか]
- **Details:** [なぜフラグ化したか。例: "citation not found in any legal database"]

---

## Internal Consistency

[対象テキスト内の矛盾があれば記載。なければ "No internal contradictions detected."]

---

## What Was Not Checked

[評価できなかった主張を列挙。paywall のある source、専門 database が必要な主張、反証不能な断定など]

---

## Limitations

- This tool accelerates human verification; it does not replace it.
- Web search results may not include the most recent information or paywalled sources.
- The adversarial review uses the same underlying model that may have produced the original output. It catches many issues but cannot catch all of them.
- A claim rated VERIFIED means a supporting source was found, not that the claim is definitely correct. Sources can be wrong too.
- Claims rated PLAUSIBLE may still be wrong. The absence of contradicting evidence is not proof of accuracy.
