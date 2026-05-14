---
name: suggest-awesome-github-copilot-instructions
description: 'Suggest relevant GitHub Copilot instruction files from the awesome-copilot repository based on current repository context and chat history, avoiding duplicates with existing instructions in this repository, and identifying outdated instructions that need updates.'
---
# 素晴らしい GitHub コパイロット手順を提案する

現在のリポジトリ コンテキストを分析し、このリポジトリでまだ利用できない関連する copilot-instructions ファイルを [GitHub awesome-copilot リポジトリ](https://github.com/github/awesome-copilot/blob/main/docs/README.instructions.md) から提案します。

＃＃ プロセス1. **利用可能な命令を取得**: [awesome-copilot README.instructions.md](https://github.com/github/awesome-copilot/blob/main/docs/README.instructions.md) から命令リストと説明を抽出します。 `#fetch` ツールを使用する必要があります。
2. **ローカル命令のスキャン**: `.github/instructions/` フォルダー内の既存の命令ファイルを検出します。
3. **説明の抽出**: ローカル命令ファイルから前付を読み取り、説明と `applyTo` パターンを取得します。
4. **リモート バージョンのフェッチ**: ローカル命令ごとに、生の GitHub URL (例: `https://raw.githubusercontent.com/github/awesome-copilot/main/instructions/<filename>`) を使用して、awesome-copilot リポジトリから対応するバージョンをフェッチします。
5. **バージョンの比較**: ローカルの指示内容とリモート バージョンを比較して、以下を特定します。
   - 最新の指示 (完全一致)
   ・説明書が古い（内容が異なる）
   - 古い命令の主な違い (説明、applyTo パターン、コンテンツ)
6. **コンテキストの分析**: チャット履歴、リポジトリ ファイル、および現在のプロジェクトのニーズを確認します。
7. **既存の比較**: このリポジトリで既に利用可能な手順と比較して確認します。
8. **関連性の一致**: 利用可能な命令を特定されたパターンおよび要件と比較します。
9. **現在のオプション**: 説明、根拠、および古い指示を含む利用可能状況を含む関連指示を表示します。
10. **検証**: 提案された手順が、既存の手順ではまだカバーされていない価値を追加することを確認します。
11. **出力**: 提案、説明、awesome-copilot 命令と同様のローカル命令の両方へのリンクを含む構造化された表を提供します。
   特定の手順のインストールまたは更新を続行するためのユーザー要求を **待ちます**。指示がない限り、インストールまたはアップデートを行わないでください。
12. **アセットのダウンロード/更新**: 要求された手順については、次のことが自動的に行われます。
    - 新しい手順を `.github/instructions/` フォルダーにダウンロードします
    - 古い手順を、awesome-copilot の最新バージョンに置き換えて更新します。
    - ファイルの内容を調整しないでください
    - アセットをダウンロードするには `#fetch` ツールを使用しますが、すべてのコンテンツが確実に取得されるように `#runInTerminal` ツールを使用して `curl` を使用することもできます
    - `#todos` ツールを使用して進行状況を追跡する

## コンテキスト分析基準🔍 **リポジトリ パターン**:
- 使用するプログラミング言語 (.cs、.js、.py、.ts など)
- フレームワーク指標 (ASP.NET、React、Azure、Next.js など)
- プロジェクトの種類 (Web アプリ、API、ライブラリ、ツール)
- 開発ワークフロー要件 (テスト、CI/CD、デプロイメント)

🗨️ **チャット履歴コンテキスト**:
- 最近の議論と問題点
- テクノロジー固有の質問
- コーディング標準の議論
- 開発ワークフローの要件

## 出力フォーマット

分析結果を、awesome-copilot 命令と既存のリポジトリ命令を比較する構造化テーブルで表示します。

|素晴らしい副操縦士の指示 |説明 |すでにインストールされています |同様のローカル指示 |提案の根拠 |
|----------------------------|-------------|---------------------|--------------------------|---------------------|
| [blazor.instructions.md](https://github.com/github/awesome-copilot/blob/main/instructions/blazor.instructions.md) | Blazor 開発ガイドライン | ✅ はい | blazor.instructions.md |既存の Blazor 命令ですでにカバーされています。
| [reactjs.instructions.md](https://github.com/github/awesome-copilot/blob/main/instructions/reactjs.instructions.md) | ReactJS 開発標準 | ❌ いいえ |なし |確立されたパターンで React 開発を強化します |
| [java.instructions.md](https://github.com/github/awesome-copilot/blob/main/instructions/java.instructions.md) | Java 開発のベスト プラクティス | ⚠️ 古い | java.instructions.md | applyTo パターンが異なります: リモートでは `'**/*.java'` を使用しますが、ローカルでは `'*.java'` を使用します - 更新をお勧めします |

## ローカル命令の検出プロセス

1. `instructions/` ディレクトリ内のすべての `*.instructions.md` ファイルを一覧表示します。
2. 検出されたファイルごとに前付を読み、`description` および `applyTo` パターンを抽出します。
3. 適用可能なファイル パターンを含む既存の命令の包括的なインベントリを構築する
4. 重複の提案を避けるためにこのインベントリを使用します

## バージョン比較プロセス1. ローカル命令ファイルごとに、生の GitHub URL を構築してリモート バージョンを取得します。
   - パターン: `https://raw.githubusercontent.com/github/awesome-copilot/main/instructions/<filename>`
2. `#fetch` ツールを使用してリモート バージョンを取得します
3. ファイルの内容全体 (前付と本文を含む) を比較します。
4. 具体的な違いを特定します。
   - **前付の変更** (説明、applyTo パターン)
   - **コンテンツの更新** (ガイドライン、例、ベストプラクティス)
5. 古い手順の重要な相違点を文書化する
6. 類似性を計算して更新が必要かどうかを判断します

## ファイル構造の要件

GitHub ドキュメントに基づくと、copilot-instructions ファイルは次のようになります。
- **リポジトリ全体の手順**: `.github/copilot-instructions.md` (リポジトリ全体に適用されます)
- **パス固有の命令**: `.github/instructions/NAME.instructions.md` (`applyTo` フロントマター経由で特定のファイル パターンに適用されます)
- **コミュニティへの指示**: `instructions/NAME.instructions.md` (共有と配布用)

## フロントマター構造

awesome-copilot の命令ファイルは、次のフロントマター形式を使用します。```markdown
---
description: 'Brief description of what this instruction provides'
applyTo: '**/*.js,**/*.ts' # Optional: glob patterns for file matching
---
```## 要件

- `githubRepo` ツールを使用して、awesome-copilot リポジトリ指示フォルダーからコンテンツを取得します
- ローカル ファイル システムをスキャンして `.github/instructions/` ディレクトリ内の既存の命令を探します
- ローカル命令ファイルから YAML 前付を読み取り、説明と `applyTo` パターンを抽出します
- ローカル命令とリモートバージョンを比較して、古い命令を検出します
- 重複を避けるために、このリポジトリ内の既存の命令と比較します。
- 現在の命令ライブラリの対象範囲のギャップに焦点を当てる
- 提案された手順がリポジトリの目的および標準と一致していることを検証します。
- それぞれの提案に対して明確な根拠を提供する
- awesome-copilot 命令と同様のローカル命令の両方へのリンクを含めます
- 特定の相違点が記載されている古い手順を明確に識別します
- テクノロジースタックの互換性とプロジェクト固有のニーズを考慮する
- 表と分析以外の追加情報やコンテキストを提供しないでください。

## アイコンのリファレンス

- ✅ すでにインストールされており、最新の状態です
- ⚠️ インストールされているが古い (アップデートが利用可能)
- ❌ リポジトリにインストールされていません

## 更新処理

古い命令が特定された場合:
1. ⚠️ ステータスを含む出力テーブルにそれらを含めます。
2.「提案の根拠」列に具体的な相違点を文書化します。
3. 重要な変更を記録して更新するよう推奨する
4. ユーザーが更新を要求すると、ローカル ファイル全体がリモート バージョンに置き換えられます。
5. ファイルの場所を `.github/instructions/` ディレクトリに保存します