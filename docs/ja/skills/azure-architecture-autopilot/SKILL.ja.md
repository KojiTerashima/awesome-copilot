---
name: azure-architecture-autopilot
description: >
  自然言語を使って Azure インフラを設計する、または既存の Azure リソースを分析して
  アーキテクチャ図を自動生成し、対話で改善し、Bicep でデプロイします。

  このスキルを使う場面:
  - "Create X on Azure", "Set up a RAG architecture"（新規設計）
  - "Analyze my current Azure infrastructure", "Draw a diagram for rg-xxx"（既存分析）
  - "Foundry is slow", "I want to reduce costs", "Strengthen security"（自然言語による変更）
  - Azure リソースのデプロイ、Bicep テンプレート生成、IaC コード生成
  - Microsoft Foundry、AI Search、OpenAI、Fabric、ADLS Gen2、Databricks、およびすべての Azure サービス
---

# Azure Architecture Builder

自然言語で Azure インフラを設計し、または既存リソースを分析してアーキテクチャを可視化し、変更とデプロイまで進めるパイプラインです。

図生成エンジンは **このスキル内に組み込み済み** です（`scripts/` フォルダー）。
`pip install` は不要で、同梱の Python スクリプトを直接使用し、
605+ の公式 Azure アイコンを使ったインタラクティブな HTML 図を生成します。
ネットワーク接続やパッケージインストールなしで、すぐに利用できます。

## ユーザー言語の自動検出

**🚨 ユーザーの最初のメッセージの言語を検出し、それ以降のすべての応答をその言語で提供してください。これは最優先の原則です。**

- ユーザーが韓国語で書いた場合 → 韓国語で応答
- ユーザーが英語で書いた場合 → **英語で応答**（ask_user、進捗更新、レポート、Bicep コメントを含むすべて）
- このドキュメントの指示と例は英語で書かれていますが、**ユーザー向け出力はすべてユーザーの言語に合わせる必要があります**

**⚠️ このドキュメントの例をユーザーにそのままコピーしないでください。**
構成のみを参考にし、文面はユーザーの言語に合わせて調整してください。

## ツール使用ガイド（GHCP 環境）

| 機能 | ツール名 | 備考 |
|---------|-----------|-------|
| URL コンテンツ取得 | `web_fetch` | MS Docs の参照など |
| Web 検索 | `web_search` | URL 発見 |
| ユーザーへの質問 | `ask_user` | `choices` は文字列配列である必要があります |
| サブエージェント | `task` | explore/task/general-purpose |
| シェルコマンド実行 | `powershell` | Windows PowerShell |

> すべてのサブエージェント（explore/task/general-purpose）は `web_fetch` または `web_search` を使用できません。  
> MS Docs の参照が必要なファクトチェックは、**必ずメインエージェントが直接**行ってください。

## 外部ツールパスの検出

`az`、`python`、`bicep` などは PATH 上にないことがよくあります。  
**Phase を開始する前に一度だけ検出して結果をキャッシュしてください。毎回再検出しないでください。**

> **⚠️ `Get-Command python` は使わないでください** — Windows Store エイリアスのリスクがあります。  
> 直接のファイルシステム探索（`$env:LOCALAPPDATA\Programs\Python`）を優先してください。

az CLI パス:
```powershell
$azCmd = $null
if (Get-Command az -ErrorAction SilentlyContinue) { $azCmd = 'az' }
if (-not $azCmd) {
  $azExe = Get-ChildItem -Path "$env:ProgramFiles\Microsoft SDKs\Azure\CLI2\wbin", "$env:LOCALAPPDATA\Programs\Azure CLI\wbin" -Filter "az.cmd" -ErrorAction SilentlyContinue | Select-Object -First 1 -ExpandProperty FullName
  if ($azExe) { $azCmd = $azExe }
}
```

Python パス + 組み込み図生成エンジン: `references/phase1-advisor.md` の図生成セクションを参照してください。

## 進捗更新は必須

blockquote + emoji + 太字の形式を使用:
```markdown
> **⏳ [Action]** — [Reason]
> **✅ [Complete]** — [Result]
> **⚠️ [Warning]** — [Details]
> **❌ [Failed]** — [Cause]
```

## 並列プリロード原則

`ask_user` でユーザー入力を待っている間、次のステップで必要な情報を並列でプリロードしてください。

| ask_user の質問 | 同時にプリロードする内容 |
|---|---|
| プロジェクト名 / スキャン範囲 | 参照ファイル、MS Docs、Python パス検出、**図モジュールパス検証** |
| モデル/SKU 選択 | 次の質問の選択肢に必要な MS Docs |
| アーキテクチャ確認 | `az account show/list`, `az group list` |
| サブスクリプション選択 | `az group list` |

---

## パス分岐 — ユーザー要求に応じて自動判定

### Path A: 新規設計（New Build）

**トリガー**: "create", "set up", "deploy", "build" など。
```
Phase 1 (references/phase1-advisor.md) — 対話型アーキテクチャ設計 + 図生成
    ↓
Phase 2 (references/bicep-generator.md) — Bicep コード生成
    ↓
Phase 3 (references/bicep-reviewer.md) — コードレビュー + コンパイル検証
    ↓
Phase 4 (references/phase4-deployer.md) — validate → what-if → deploy
```

### Path B: 既存分析 + 変更（Analyze & Modify）

**トリガー**: "analyze", "current resources", "scan", "draw a diagram", "show my infrastructure" など。
```
Phase 0 (references/phase0-scanner.md) — 既存リソースのスキャン + 図生成
    ↓
変更に関する対話 — "What would you like to change here?"（自然言語の変更要求 → フォローアップ質問）
    ↓
Phase 1 (references/phase1-advisor.md) — 変更内容の確認 + 図更新
    ↓
Phase 2~4 — 上記と同じ
```

### パス判定が曖昧な場合

ユーザーに直接質問してください:
```
ask_user({
  question: "What would you like to do?",
  choices: [
    "Design a new Azure architecture (Recommended)",
    "Analyze + modify existing Azure resources"
  ]
})
```

---

## Phase 遷移ルール

- 各 Phase は対応する `references/*.md` ファイルの指示を読み、従ってください
- Phase 間を遷移する際は、次のステップを必ずユーザーに伝えてください
- Phase をスキップしないでください（特に Phase 3 → Phase 4 間の what-if）
- **🚨 Phase 1 → Phase 2 遷移の必須条件**: `01_arch_diagram_draft.html` を組み込み図生成エンジンで生成し、ユーザーに提示済みであること。**図なしで Bicep 生成に進んではいけません。** 仕様収集完了だけでは Phase 1 完了にはなりません — Phase 1 には図生成 + ユーザー確認が含まれます。
- デプロイ後の変更要求 → Phase 0 ではなく Phase 1 に戻る（Delta Confirmation Rule）

## サービス対応範囲とフォールバック

### 最適化済みサービス
Microsoft Foundry、Azure OpenAI、AI Search、ADLS Gen2、Key Vault、Microsoft Fabric、Azure Data Factory、VNet/Private Endpoint、AML/AI Hub

### その他の Azure サービス
すべて対応 — 同等品質で生成するために MS Docs を自動参照します。  
**「対象外」や「ベストエフォート」のようにユーザー不安を招くメッセージは送らないでください。**

### 安定情報と動的情報の扱い

| カテゴリ | 取り扱い方法 | 例 |
|----------|----------------|---------|
| **Stable** | まず参照ファイルを確認 | `isHnsEnabled: true`, PE triple set |
| **Dynamic** | **常に MS Docs を取得** | API version, model availability, SKU, region |

## クイックリファレンス

| ファイル | 役割 |
|------|------|
| `references/phase0-scanner.md` | 既存リソーススキャン + 関係推論 + 図生成 |
| `references/phase1-advisor.md` | 対話型アーキテクチャ設計 + ファクトチェック |
| `references/bicep-generator.md` | Bicep コード生成ルール |
| `references/bicep-reviewer.md` | コードレビューチェックリスト |
| `references/phase4-deployer.md` | validate → what-if → deploy |
| `references/service-gotchas.md` | 必須プロパティ、PE マッピング |
| `references/azure-dynamic-sources.md` | MS Docs URL レジストリ |
| `references/azure-common-patterns.md` | PE/セキュリティ/命名パターン |
| `references/ai-data.md` | AI/データサービスガイド |

<system_reminder>
<sql_tables>現在テーブルは存在しません。SQL ツールを初めて使用すると、デフォルトテーブル（todos、todo_deps）が自動作成されます。</sql_tables>
</system_reminder>

