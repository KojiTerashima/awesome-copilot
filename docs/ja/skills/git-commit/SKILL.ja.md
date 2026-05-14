---
name: git-commit
description: '差分を解析し、Conventional Commits に沿ったコミットメッセージ分析、賢いステージング、メッセージ生成を行いながら git commit を実行します。ユーザーが変更のコミット、git commit の作成、または "/commit" に言及したときに使います。対応内容: (1) 変更から type と scope を自動判定、(2) diff から Conventional Commit メッセージを生成、(3) type/scope/description の上書きも可能な対話式コミット、(4) 論理単位での賢いファイルステージング'
license: MIT
allowed-tools: Bash
---

# Conventional Commits を使った Git Commit

## 概要

Conventional Commits 仕様に従って、標準化された意味のある git commit を作成します。実際の diff を解析して、適切な type、scope、メッセージを判断します。

## Conventional Commit の形式

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

## Commit Type

| Type | 用途 |
| ---------- | ------------------------------ |
| `feat` | 新機能 |
| `fix` | バグ修正 |
| `docs` | ドキュメントのみ |
| `style` | フォーマット / スタイル（ロジック変更なし） |
| `refactor` | リファクタリング（機能追加や修正なし） |
| `perf` | パフォーマンス改善 |
| `test` | テストの追加 / 更新 |
| `build` | ビルドシステム / 依存関係 |
| `ci` | CI / 設定変更 |
| `chore` | 保守 / その他 |
| `revert` | コミットの取り消し |

## Breaking Changes

```
# type/scope の後ろに感嘆符を付ける
feat!: remove deprecated endpoint

# BREAKING CHANGE フッター
feat: allow config to extend other configs

BREAKING CHANGE: `extends` key behavior changed
```

## ワークフロー

### 1. Diff を解析する

```bash
# すでにステージ済みのファイルがある場合は staged diff を使う
git diff --staged

# ステージ済みがなければ working tree diff を使う
git diff

# あわせて status も確認する
git status --porcelain
```

### 2. 必要ならファイルをステージする

何もステージされていない場合や、別の論理単位でまとめたい場合:

```bash
# 特定ファイルをステージ
git add path/to/file1 path/to/file2

# パターンでステージ
git add *.test.*
git add src/components/*

# 対話式ステージング
git add -p
```

**秘密情報は絶対にコミットしないこと**（`.env`、`credentials.json`、秘密鍵など）。

### 3. コミットメッセージを生成する

diff を解析し、次を判断する:

- **Type**: どの種類の変更か
- **Scope**: どの領域 / モジュールに影響するか
- **Description**: 何を変えたかを表す 1 行要約（現在形、命令形、72 文字未満）

### 4. コミットを実行する

```bash
# 1 行メッセージ
git commit -m "<type>[scope]: <description>"

# body/footer を含む複数行メッセージ
git commit -m "$(cat <<'EOF'
<type>[scope]: <description>

<optional body>

<optional footer>
EOF
)"
```

## ベストプラクティス

- 1 つの論理変更につき 1 コミット
- 現在形を使う: "add" であり "added" ではない
- 命令形を使う: "fix bug" であり "fixes bug" ではない
- issue 参照を使う: `Closes #123`、`Refs #456`
- description は 72 文字以内に収める

## Git 安全プロトコル

- git config は絶対に更新しない
- 明示的な依頼なしに破壊的コマンド（`--force`、hard reset）を実行しない
- ユーザーが求めない限り hooks をスキップしない（`--no-verify` を使わない）
- main/master へ強制 push しない
- hooks が原因で commit に失敗した場合は修正し、新しいコミットを作る（amend しない）
