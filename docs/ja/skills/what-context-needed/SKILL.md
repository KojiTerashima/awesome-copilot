---
name: what-context-needed
description: 'Ask Copilot what files it needs to see before answering a question'
---
# どのようなコンテキストが必要ですか?

私の質問に答える前に、どのファイルを見る必要があるのか​​教えてください。

## 私の質問

{{質問}}

## 指示

1. 私の質問に基づいて、調査する必要があるファイルをリストします。
2. 各ファイルが関連する理由を説明する
3. この会話ですでに見たファイルを書き留めます
4. 不明な点を特定する

## 出力フォーマット```markdown
## Files I Need

### Must See (required for accurate answer)
- `path/to/file.ts` — [why needed]

### Should See (helpful for complete answer)
- `path/to/file.ts` — [why helpful]

### Already Have
- `path/to/file.ts` — [from earlier in conversation]

### Uncertainties
- [What I'm not sure about without seeing the code]
```これらのファイルを提供した後、もう一度質問します。