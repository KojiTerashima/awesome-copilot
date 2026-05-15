---
name: ai-ready
description: 'あらゆるリポジトリを AI 対応にする — コードベースを分析し、AGENTS.md、copilot-instructions.md、CI ワークフロー、問題テンプレートなどを生成します。 PR レビュー パターンをマイニングし、スタックに合わせてカスタマイズされたファイルを作成します。ユーザーが「このリポジトリを AI 対応にする」、「AI 構成をセットアップする」、または「AI コントリビューション用にこのリポジトリを準備する」を要求する場合は、このスキルを使用します。'
---

# AI対応

このスキルは、ユーザーが [John Papa](https://github.com/johnpapa) による最新の [ai-ready](https://github.com/johnpapa/ai-ready) `SKILL.md` を個人スキル ディレクトリにインストールするのに役立ちます。

*なぜですか?*: 完全な AI 対応スキルは、頻繁に進化する約 600 行の詳細な命令です。このラッパーは、真実のソースが [johnpapa/ai-ready](https://github.com/johnpapa/ai-ready) にある間、ここで発見できるようにしており、常に最新の状態に保たれています。

## ステップ

1. ターミナルで次のコマンドのいずれかを実行して、最新の `SKILL.md` を個人スキル ディレクトリにダウンロードするようにユーザーに指示します。これにより、既存のローカル コピーが上書きされます。

**bash / zsh**
   ```bash
   mkdir -p ~/.copilot/skills/ai-ready
   curl -fsSL https://raw.githubusercontent.com/johnpapa/ai-ready/main/skills/ai-ready/SKILL.md \
     -o ~/.copilot/skills/ai-ready/SKILL.md
   ```

**PowerShell**
   ```powershell
   New-Item -ItemType Directory -Force -Path "$HOME/.copilot/skills/ai-ready" | Out-Null
   Invoke-WebRequest -UseBasicParsing "https://raw.githubusercontent.com/johnpapa/ai-ready/main/skills/ai-ready/SKILL.md" -OutFile "$HOME/.copilot/skills/ai-ready/SKILL.md"
   ```

動作を再現するには、ユーザーは URL 内の `main` を特定のタグに置き換えるか、SHA をコミットします。
2. ダウンロードしたスキルをロードする前にレビューして、期待される指示が含まれていることを確認することをユーザーに提案します。
   ```bash
   head -20 ~/.copilot/skills/ai-ready/SKILL.md
   ```
3. ユーザーがインストールしたことを確認したら、`/skills reload` を使用してスキルをリロードするように指示し、次に `make this repo ai-ready` と言います。
4. ユーザーに代わってインストール コマンドを実行しないでください**。ユーザーが自分で実行する必要があります。