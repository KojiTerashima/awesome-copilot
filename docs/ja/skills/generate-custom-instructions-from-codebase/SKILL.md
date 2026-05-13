---
name: generate-custom-instructions-from-codebase
description: 'GitHub Copilot 向けの移行・コード進化指示ジェネレーター。2つのプロジェクトバージョン（ブランチ、コミット、またはリリース）の差分を分析し、技術移行・大規模リファクタリング・フレームワークのバージョンアップ時に Copilot が一貫性を維持できるよう、精密な指示を作成します。'
---

# 移行・コード進化指示ジェネレーター

## 設定変数

```
${MIGRATION_TYPE="Framework Version|Architecture Refactoring|Technology Migration|Dependencies Update|Pattern Changes"}
<!-- 移行または進化の種類 -->

${SOURCE_REFERENCE="branch|commit|tag"}
<!-- ソース参照点（変更前の状態） -->

${TARGET_REFERENCE="branch|commit|tag"}
<!-- ターゲット参照点（変更後の状態） -->

${ANALYSIS_SCOPE="Entire project|Specific folder|Modified files only"}
<!-- 分析範囲 -->

${CHANGE_FOCUS="Breaking Changes|New Conventions|Obsolete Patterns|API Changes|Configuration"}
<!-- 変更の主な観点 -->

${AUTOMATION_LEVEL="Conservative|Balanced|Aggressive"}
<!-- Copilot 提案の自動化レベル -->

${GENERATE_EXAMPLES="true|false"}
<!-- 変換例を含める -->

${VALIDATION_REQUIRED="true|false"}
<!-- 適用前の検証を必須にする -->
```

## 生成されるプロンプト

```
"2つのプロジェクト状態間のコード進化を分析し、GitHub Copilot 向けに精密な移行指示を生成してください。これらの指示は、今後の変更時に Copilot が同じ変換パターンを自動適用するためのガイドになります。以下の方法論に従ってください。

### フェーズ1: 状態の比較分析

#### 構造変更の検出
- ${SOURCE_REFERENCE} と ${TARGET_REFERENCE} のフォルダー構造を比較する
- 移動・リネーム・削除されたファイルを特定する
- 設定ファイルの変更を分析する
- 追加された依存関係と削除された依存関係を記録する

#### コード変換分析
${MIGRATION_TYPE == "Framework Version" ?
  "- フレームワークのバージョン間における API 変更を特定する
   - 使用されている新機能を分析する
   - 廃止されたメソッド/プロパティを記録する
   - 構文または規約の変更を明記する" : ""}

${MIGRATION_TYPE == "Architecture Refactoring" ?
  "- アーキテクチャパターンの変更を分析する
   - 導入された新しい抽象化を特定する
   - 責務再編を記録する
   - データフローの変更を明記する" : ""}

${MIGRATION_TYPE == "Technology Migration" ?
  "- ある技術から別の技術への置き換えを分析する
   - 機能上の対応関係を特定する
   - API と構文の変更を記録する
   - 新しい依存関係と設定を明記する" : ""}

#### 変換パターンの抽出
- 適用された反復的な変換を特定する
- 旧形式から新形式への変換ルールを分析する
- 例外および特殊ケースを記録する
- 変換前/変換後の対応マトリクスを作成する

### フェーズ2: 移行指示の生成

以下の構成で `.github/copilot-migration-instructions.md` ファイルを作成してください:

\`\`\`markdown
# GitHub Copilot Migration Instructions

## Migration Context
- **Type**: ${MIGRATION_TYPE}
- **From**: ${SOURCE_REFERENCE}
- **To**: ${TARGET_REFERENCE}
- **Date**: [GENERATION_DATE]
- **Scope**: ${ANALYSIS_SCOPE}

## Automatic Transformation Rules

### 1. Mandatory Transformations
${AUTOMATION_LEVEL != "Conservative" ?
  "[AUTOMATIC_TRANSFORMATION_RULES]
   - **Old Pattern**: [OLD_CODE]
   - **New Pattern**: [NEW_CODE]
   - **Trigger**: このパターンを検出する条件
   - **Action**: 自動適用する変換" : ""}

### 2. Transformations with Validation
${VALIDATION_REQUIRED == "true" ?
  "[TRANSFORMATIONS_WITH_VALIDATION]
   - **Detected Pattern**: [DESCRIPTION]
   - **Suggested Transformation**: [NEW_APPROACH]
   - **Required Validation**: [VALIDATION_CRITERIA]
   - **Alternatives**: [ALTERNATIVE_OPTIONS]" : ""}

### 3. API Correspondences
${CHANGE_FOCUS == "API Changes" || MIGRATION_TYPE == "Framework Version" ?
  "[API_CORRESPONDENCE_TABLE]
   | Old API   | New API   | Notes     | Example        |
   | --------- | --------- | --------- | -------------- |
   | [OLD_API] | [NEW_API] | [CHANGES] | [CODE_EXAMPLE] | " : ""} |

### 4. New Patterns to Adopt
[DETECTED_EMERGING_PATTERNS]
- **Pattern**: [PATTERN_NAME]
- **Usage**: [WHEN_TO_USE]
- **Implementation**: [HOW_TO_IMPLEMENT]
- **Benefits**: [ADVANTAGES]

### 5. Obsolete Patterns to Avoid
[DETECTED_OBSOLETE_PATTERNS]
- **Obsolete Pattern**: [OLD_PATTERN]
- **Why Avoid**: [REASONS]
- **Alternative**: [NEW_PATTERN]
- **Migration**: [CONVERSION_STEPS]

## File Type Specific Instructions

${GENERATE_EXAMPLES == "true" ?
  "### Configuration Files
   [CONFIG_TRANSFORMATION_EXAMPLES]

   ### Main Source Files
   [SOURCE_TRANSFORMATION_EXAMPLES]

   ### Test Files
   [TEST_TRANSFORMATION_EXAMPLES]" : ""}

## Validation and Security

### Automatic Control Points
- 各変換後に実施する検証
- 変更を妥当性確認するために実行するテスト
- 監視すべきパフォーマンス指標
- 実施すべき互換性チェック

### Manual Escalation
人の介入が必要な状況:
- [COMPLEX_CASES_LIST]
- [ARCHITECTURAL_DECISIONS]
- [BUSINESS_IMPACTS]

## Migration Monitoring

### Tracking Metrics
- 自動移行されたコードの割合
- 必要となった手動検証の件数
- 自動変換のエラー率
- ファイルあたりの平均移行時間

### Error Reporting
Copilot への誤変換の報告方法:
- ルール改善のためのフィードバックパターン
- 記録すべき例外
- 指示に加えるべき調整

\`\`\`

### フェーズ3: 文脈に応じた例の生成

${GENERATE_EXAMPLES == "true" ?
  "#### 変換例
   特定した各パターンについて、以下を生成します:

   \`\`\`
   // BEFORE (${SOURCE_REFERENCE})
   [OLD_CODE_EXAMPLE]

   // AFTER (${TARGET_REFERENCE})
   [NEW_CODE_EXAMPLE]

   // COPILOT INSTRUCTIONS
   このパターン [TRIGGER] を見つけたら、次の手順 [STEPS] に従って [NEW_PATTERN] に変換してください
   \`\`\`" : ""}

### フェーズ4: 検証と最適化

#### 指示のテスト
- テストコードに指示を適用する
- 変換の一貫性を確認する
- 結果に基づいてルールを調整する
- 例外とエッジケースを記録する

#### 反復的な最適化
${AUTOMATION_LEVEL == "Aggressive" ?
  "- 自動化を最大化するようにルールを洗練する
   - 検出時の偽陽性を減らす
   - 変換精度を向上させる
   - 得られた知見を記録する" : ""}

### 最終成果物

GitHub Copilot が次を可能にする移行指示:
1. 今後の変更時に同じ変換を**自動適用**する
2. 新たに採用した規約との**一貫性を維持**する
3. 代替案を自動提案して**廃止パターンを回避**する
4. 得られた経験を活用して**将来の移行を高速化**する
5. 反復的な変換の自動化によって**エラーを削減**する

これらの指示により、Copilot はインテリジェントな移行アシスタントへと変わり、あなたの技術進化の意思決定を一貫して信頼性高く再現できるようになります。
"
```

## 典型的なユースケース

### フレームワークのバージョン移行
Angular 14 から Angular 17、React Class Components から Hooks、.NET Framework から .NET Core への移行記録に最適です。破壊的変更を自動的に特定し、対応する変換ルールを生成します。

### 技術スタックの進化
技術を全面的に置き換える場合に不可欠です: jQuery から React、REST から GraphQL、SQL から NoSQL。パターン対応を含む包括的な移行ガイドを作成します。

### アーキテクチャリファクタリング
Monolith から Microservices、MVC から Clean Architecture、Component から Composable architecture などの大規模リファクタリングに理想的です。今後の類似変換に向けてアーキテクチャ知識を保持します。

### デザインパターンのモダナイゼーション
Repository Pattern、Dependency Injection、Observer から Reactive Programming などの新パターン採用に有用です。根拠と実装差分を記録します。

## 独自の利点

### 🧠 **人工知能の強化**
従来の移行ドキュメントとは異なり、これらの指示は GitHub Copilot を「訓練」し、将来のコード変更時にあなたの技術進化の意思決定を自動再現できるようにします。

### 🔄 **知識の資産化**
特定プロジェクトで得た経験を再利用可能なルールへ変換し、移行ノウハウの喪失を防ぎ、将来の類似変換を加速します。

### 🎯 **文脈認識に基づく高精度**
汎用的な助言ではなく、あなた固有のコードベースに合わせた指示を生成し、プロジェクト進化から得た実際の before/after 例を示します。

### ⚡ **自動化された一貫性**
新規コード追加時にも新しい規約への準拠を自動的に保証し、アーキテクチャの後退を防いでコード進化の整合性を維持します。

