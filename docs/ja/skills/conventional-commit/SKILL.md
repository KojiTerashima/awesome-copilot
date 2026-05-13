---
name: conventional-commit
description: '構造化された XML 形式を使って conventional commit message を生成するための prompt と workflow。instructions、examples、validation を通じて、Conventional Commits specification に沿った標準化された説明的な commit message の作成を導きます。'
---

### Instructions

```xml
	<description>このファイルは conventional commit message を生成するための prompt template を含みます。instructions、examples、formatting guideline を提供し、Conventional Commits specification に従った標準化された説明的 commit message を書けるよう支援します。</description>
```

### Workflow

**次の手順に従ってください:**

1. 変更されたファイルを確認するために `git status` を実行する。
2. `git diff` または `git diff --cached` を実行して変更内容を確認する。
3. `git add <file>` で変更を stage する。
4. 次の XML 構造を使って commit message を組み立てる。
5. commit message を生成したら、Copilot は integrated terminal で次の command を自動実行します（確認不要）。

```bash
git commit -m "type(scope): description"
```

6. この prompt を実行するだけで、Copilot が terminal で commit を処理します。

### Commit Message Structure

```xml
<commit-message>
	<type>feat|fix|docs|style|refactor|perf|test|build|ci|chore|revert</type>
	<scope>()</scope>
	<description>変更内容を命令形で簡潔に要約した文</description>
	<body>(optional: より詳細な説明)</body>
	<footer>(optional: 例 BREAKING CHANGE: details、または issue 参照)</footer>
</commit-message>
```

### Examples

```xml
<examples>
	<example>feat(parser): add ability to parse arrays</example>
	<example>fix(ui): correct button alignment</example>
	<example>docs: update README with usage instructions</example>
	<example>refactor: improve performance of data processing</example>
	<example>chore: update dependencies</example>
	<example>feat!: send email on registration (BREAKING CHANGE: email service required)</example>
</examples>
```

### Validation

```xml
<validation>
	<type>許可された type のいずれかでなければなりません。<reference>https://www.conventionalcommits.org/en/v1.0.0/#specification</reference> を参照してください。</type>
	<scope>任意ですが、明確さのため推奨されます。</scope>
	<description>必須です。命令形を使ってください（例: "added" ではなく "add"）。</description>
	<body>任意です。追加の文脈が必要な場合に使います。</body>
	<footer>breaking change または issue 参照に使います。</footer>
</validation>
```

### Final Step

```xml
<final-step>
	<cmd>git commit -m "type(scope): description"</cmd>
	<note>構成した message に置き換えてください。必要なら body と footer も含めます。</note>
</final-step>
```
