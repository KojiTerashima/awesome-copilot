---
name: powerbi-modeling
description: 'Power BI semantic modeling assistant for building optimized data models. Use when working with Power BI semantic models, creating measures, designing star schemas, configuring relationships, implementing RLS, or optimizing model performance. Triggers on queries about DAX calculations, table relationships, dimension/fact table design, naming conventions, model documentation, cardinality, cross-filter direction, calculation groups, and data model best practices. Always connects to the active model first using power-bi-modeling MCP tools to understand the data structure before providing guidance.'
---
# Power BI セマンティック モデリング

Microsoft のベスト プラクティスに従って、最適化され、十分に文書化された Power BI セマンティック モデルを構築できるようにユーザーをガイドします。

## このスキルを使用する場合

ユーザーが次のことについて質問する場合は、このスキルを使用します。
- Power BI セマンティック モデルの作成または最適化
- スター スキーマ (ディメンション/ファクト テーブル) の設計
- DAX メジャーまたは計算列の書き込み
- テーブルのリレーションシップの構成 (カーディナリティ、クロスフィルター)
- 行レベルのセキュリティ (RLS) の実装
- テーブル、列、メジャーの命名規則
- モデルに説明とドキュメントを追加する
- パフォーマンスのチューニングと最適化
- 計算グループとフィールドパラメータ
- モデルの検証とベストプラクティスのチェック

**トリガー フレーズ:** 「メジャーの作成」、「関係の追加」、「スター スキーマ」、「モデルの最適化」、「DAX 式」、「RLS」、「命名規則」、「モデルのドキュメント」、「カーディナリティ」、「クロスフィルター」

## 前提条件

### 必要なツール
- **Power BI モデリング MCP サーバー**: セマンティック モデルに接続して変更するために必要です
  - 有効化: connection_operations、table_operations、measure_operations、relationship_operations など。
  - モデルと対話するには、設定して実行する必要があります

### オプションの依存関係
- **Microsoft Learn MCP Server**: 最新のベスト プラクティスを調査する場合に推奨します
  - 有効化:microsoft_docs_search、microsoft_docs_fetch
  - 複雑なシナリオ、新機能、公式ドキュメントに使用します。

## ワークフロー

### 1. まず接続して分析する

モデリングに関するガイダンスを提供する前に、必ず現在のモデルの状態を調べてください。「」
1. 接続のリスト: connection_operations(operation: "ListConnections")
2. 接続がない場合は、ローカル インスタンスを確認します: connection_operations(operation: "ListLocalInstances")
3. モデルに接続します (デスクトップまたはファブリック)
4. モデル概要の取得:model_operations(operation: "Get")
5. テーブルの一覧表示: table_operations(操作: "List")
6. 関係のリスト: relationship_operations(operation: "List")
7. メジャーのリスト:measure_operations(操作: "リスト")
「」### 2. モデルの健全性を評価する

接続後、ベスト プラクティスに照らしてモデルを評価します。

- **スター スキーマ**: テーブルはディメンションまたはファクトとして適切に分類されていますか?
- **関係**: 基数は正しいですか?最小限の双方向フィルタ?
- **命名**: 人間が判読できる一貫した命名規則はありますか?
- **ドキュメント**: テーブル、列、メジャーには説明がありますか?
- **対策**: 主要な計算に対する明示的な対策はありますか?
- **非表示フィールド**: 技術的な列はレポート ビューから非表示になりますか?

### 3. 的を絞ったガイダンスを提供する

分析に基づいて、参考資料を使用して改善をガイドします。
- スター スキーマの設計: [STAR-SCHEMA.md] を参照(references/STAR-SCHEMA.md)
- 関係の設定: [RELATIONSHIPS.md](references/RELATIONSHIPS.md) を参照してください。
- DAX メジャーと命名: [MEASURES-DAX.md](references/MEASURES-DAX.md) を参照してください。
- パフォーマンスの最適化: [PERFORMANCE.md](references/PERFORMANCE.md) を参照してください。
- 行レベルのセキュリティ: [RLS.md](references/RLS.md) を参照してください。

## クイック リファレンス: モデル品質チェックリスト

|エリア |ベストプラクティス |
|------|--------------|
|テーブル |明確なディメンションと事実の分類 |
|ネーミング |人間が読める形式: `CUST_NM` ではなく `Customer Name` |
|説明 |すべてのテーブル、列、メジャーを文書化 |
|対策 |ビジネス指標の明示的な DAX 測定 |
|人間関係 |ディメンションからファクトへの一対多 |
|クロスフィルター |特に必要でない限り単一方向 |
|隠しフィールド |レポート ビューからテクニカル キー、ID を非表示にする |
|日付テーブル |専用のマーク付き日付テーブル |

## MCP ツールのリファレンス

次の Power BI モデリング MCP 操作を使用します。

|操作カテゴリ |キー操作 |
|-------------------|----------------|
| `connection_operations` |接続、ListConnections、ListLocalInstances、ConnectFabric |
| `model_operations` |取得、GetStats、ExportTMDL |
| `table_operations` |リスト、取得、作成、更新、GetSchema |
| `column_operations` |リスト、取得、作成、更新 (説明、非表示、フォーマット) |
| `measure_operations` |リスト、取得、作成、更新、移動 |
| `relationship_operations` |リスト、取得、作成、更新、アクティブ化、非アクティブ化 |
| `dax_query_operations` |実行、検証 |
| `calculation_group_operations` |リスト、作成、更新 |
| `security_role_operations` |リスト、作成、更新、GetEffectivePermissions |

## 一般的なタスク

### 説明付きのメジャーを追加「」
測定操作(
  操作: "作成"、
  定義: [{
    名前: 「総売上高」、
    テーブル名: "売上",
    式: "SUM(売上[金額])",
    フォーマット文字列: "$#,##0",
    説明: 「すべての売上金額の合計」
  }]
）
「」### 列の説明を更新「」
列操作(
  操作: "更新"、
  定義: [{
    テーブル名: "顧客",
    名前: "顧客キー"、
    説明: "顧客ディメンションの一意の識別子",
    非表示: true
  }]
）
「」### 関係を作成する「」
関係操作(
  操作: "作成"、
  定義: [{
    fromTable: "売上",
    fromColumn: "CustomerKey",
    toTable: "顧客",
    toColumn: "CustomerKey",
    クロスフィルタリング動作: "OneDirection"
  }]
）
「」## Microsoft Learn MCP を使用する場合

以下について `microsoft_docs_search` を使用して現在のベスト プラクティスを調査します。
- 最新の DAX 関数ドキュメント
- 新しい Power BI の機能と機能
- 複雑なモデリング シナリオ (SCD タイプ 2、多対多)
- パフォーマンス最適化手法
- セキュリティ実装パターン