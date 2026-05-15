---
name: md-to-docx
description: Convert Markdown files to professionally formatted Word (.docx) documents with embedded PNG images — pure JavaScript, no external tools required
---

# Markdown to Word (.docx) スキル

Markdown (`.md`) ファイルを、PNG 画像が埋め込まれた専門的にフォーマットされた Word (`.docx`) ドキュメントに変換します。 `docx` および `marked` npm パッケージを介して **純粋な JavaScript** を使用します。Pandoc、LibreOffice、またはネイティブ バイナリは必要ありません。

## 変換方法
```bash
# Install dependencies (one-time, from the scripts folder)
cd skills/md-to-docx/scripts && npm install

# Convert (run from workspace root)
node skills/md-to-docx/scripts/md-to-docx.mjs <input.md> [output.docx]
```

`output.docx` を省略した場合、デフォルトは現在のディレクトリの `<input-basename>.docx` になります。

## スキルフォルダの内容

|ファイル |目的 |
|-----|----------|
| `SKILL.md` |この命令ファイル |
| `scripts/md-to-docx.mjs` | Node.js Markdown から Word へのコンバータ |
| `scripts/package.json` |依存関係 (`docx`、`marked`) |

## 前提条件

|要件 |バージョン |メモ |
|-----------|-----------|------|
| **Node.js** | 18+ |必要なランタイム |
| **`docx`** | 9+ |純粋な JS Word ドキュメント ジェネレーター |
| **`marked`** | 15 歳以上 |マークダウンパーサー |

ネイティブバイナリはありません。システムレベルのインストールはありません。 Windows、macOS、Linux で動作します。

## 特徴

コンバーター:

- **YAML の前付部分を抽出します** — タイトル ページに `title`、`date`、`version`、`audience` を使用します
- **タイトル ページを生成します** - プロジェクト名、サブタイトル、日付、バージョン、対象ユーザーを含む
- **目次を生成します** — H1 ～ H3 の見出しから作成されます
- **PNG 画像を埋め込む** - 入力 `.md` ファイルに関連する `![alt](path)` 参照を解決し、PNG を読み取り、Word 文書にインラインで埋め込みます。
- **スタイル付き出力** — Calibri フォント、色付きの見出し (`#1F3864`)、行の色が交互になるスタイル付きの表、Consolas のコード ブロック
- **すべての Markdown 要素を処理します** - 見出し、段落、表、コード ブロック、リスト、画像、リンク、水平罫線

## 画像の埋め込み

コンバーターは、Markdown で参照される PNG 画像を自動的に埋め込みます。
```markdown
![High-Level Architecture](diagrams/high-level-architecture.drawio.png)
```

画像パスは**入力Markdownファイルを基準にして**解決されます。 PNG が読み取られ、PNG ヘッダーから寸法が抽出され、アスペクト比を維持しながら画像が幅 6 インチ以内に収まるように拡大縮小されます。

画像ファイルが見つからない場合は、プレースホルダー `[Image not found: <path>]` が挿入されます。

## フロントマターフォーマット
```yaml
---
title: Project Name — Project Summary
date: 2025-01-15
version: 1.0
audience: Engineering Team, Architects, Stakeholders
---
```

タイトルは`—`または`–`でタイトルページのメインタイトルとサブタイトルに分割されます。