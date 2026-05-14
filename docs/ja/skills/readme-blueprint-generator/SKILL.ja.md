---
name: readme-blueprint-generator
description: 'Intelligent README.md generation prompt that analyzes project documentation structure and creates comprehensive repository documentation. Scans .github/copilot directory files and copilot-instructions.md to extract project information, technology stack, architecture, development workflow, coding standards, and testing approaches while generating well-structured markdown documentation with proper formatting, cross-references, and developer-focused content.'
---
# README ジェネレーター プロンプト

.github/copilot ディレクトリ内のドキュメント ファイルと copilot-instructions.md ファイルを分析して、このリポジトリの包括的な README.md を生成します。次の手順に従います。

1. 次のように、.github/copilot フォルダー内のすべてのファイルをスキャンします。
   - 建築
   - コード例
   - コーディング_標準
   - プロジェクトフォルダーの構造
   - テクノロジー_スタック
   - 単体テスト
   - ワークフロー_分析

2. .github フォルダー内の copilot-instructions.md ファイルも確認します。

3. 次のセクションを含む README.md を作成します。

## プロジェクト名と説明
- ドキュメントからプロジェクト名と主な目的を抽出します。
- プロジェクトの内容についての簡潔な説明を含めます

## テクノロジースタック
- 使用されている主なテクノロジー、言語、フレームワークをリストします。
- 利用可能な場合はバージョン情報を含めます
- この情報は主に Technology_Stack ファイルから取得します。

## プロジェクトのアーキテクチャ
- アーキテクチャの概要を説明します。
- ドキュメントに説明されている場合は、簡単な図を含めることを検討してください。
- アーキテクチャ ファイルからのソース

## はじめに
- テクノロジースタックに基づいたインストール手順を含めます
- セットアップと構成の手順を追加
- 前提条件を含めます

## プロジェクトの構造
- フォルダー構成の概要
- Project_Folder_Structure ファイルからのソース

## 主な機能
- プロジェクトの主な機能と特徴をリストします。
- さまざまなドキュメント ファイルからの抽出

## 開発ワークフロー
- 開発プロセスを要約する
- 可能な場合は分岐戦略に関する情報を含めます
- Workflow_Analysis ファイルからのソース

## コーディング標準
- 主要なコーディング標準と規約を要約します。
- ソースはCoding_Standardsファイルから

## テスト
- テストのアプローチとツールについて説明する
- Unit_Tests ファイルからのソース

## 貢献する
- プロジェクトに貢献するためのガイドライン
- ガイダンスとしてコード例を参照してください。
- Code_Exemplars および copilot-instructions からのソース

## ライセンス
- 利用可能な場合はライセンス情報を含めます

以下を含む適切な Markdown を使用して README をフォーマットします。
- 見出しと小見出しを明確にする
- 該当する場合はコードブロック
- 読みやすさを高めるリスト
- 他のドキュメント ファイルへのリンク
- 情報が入手可能な場合は、ビルド ステータス、バージョンなどのバッジ

README は簡潔でありながら有益なものにし、新しい開発者やユーザーがプロジェクトについて知っておく必要があることに重点を置きます。