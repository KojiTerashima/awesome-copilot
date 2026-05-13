---
name: az-cost-optimize
description: 'アプリで使用される Azure リソース（IaC ファイルおよび/または対象 rg 内のリソース）を分析し、コストを最適化します。特定した最適化項目について GitHub issue を作成します。'
---

# Azure コスト最適化

このワークフローは、Infrastructure-as-Code（IaC）ファイルと Azure リソースを分析して、コスト最適化の推奨事項を生成します。各最適化機会ごとに個別の GitHub issue を作成し、さらに実装調整用の EPIC issue を 1 つ作成することで、コスト削減施策の効率的な追跡と実行を可能にします。

## 前提条件
- Azure MCP サーバーが設定済みかつ認証済みであること
- GitHub MCP サーバーが設定済みかつ認証済みであること  
- 対象の GitHub リポジトリが特定されていること
- Azure リソースがデプロイ済みであること（IaC ファイルは任意だがあると有用）
- 利用可能な場合は、直接 Azure CLI を使うより Azure MCP ツール（`azmcp-*`）を優先すること

## ワークフローステップ

### ステップ 1: Azure ベストプラクティスを取得
**アクション**: 分析前にコスト最適化のベストプラクティスを取得する  
**ツール**: Azure MCP ベストプラクティスツール  
**プロセス**:
1. **ベストプラクティスの読み込み**:
   - `azmcp-bestpractices-get` を実行して、最新の Azure 最適化ガイドラインを取得します。すべてのシナリオを網羅しない可能性はありますが、土台として有用です。
   - これらのプラクティスを可能な限り後続の分析と推奨事項に反映します
   - 最適化推奨には、MCP ツール出力または一般的な Azure ドキュメントからベストプラクティスを引用します

### ステップ 2: Azure インフラストラクチャを検出
**アクション**: Azure リソースと構成を動的に検出・分析する  
**ツール**: Azure MCP ツール + Azure CLI フォールバック + ローカルファイルシステムアクセス  
**プロセス**:
1. **リソース検出**:
   - `azmcp-subscription-list` を実行して利用可能なサブスクリプションを見つける
   - `azmcp-group-list --subscription <subscription-id>` を実行してリソースグループを見つける
   - 関連グループ内の全リソース一覧を取得する:
     - `az resource list --subscription <id> --resource-group <name>` を使用
   - 各リソース種別について、可能であればまず MCP ツール、次に CLI フォールバックを使用:
     - `azmcp-cosmos-account-list --subscription <id>` - Cosmos DB アカウント
     - `azmcp-storage-account-list --subscription <id>` - ストレージアカウント  
     - `azmcp-monitor-workspace-list --subscription <id>` - Log Analytics ワークスペース
     - `azmcp-keyvault-key-list` - Key Vault
     - `az webapp list` - Web アプリ（フォールバック - 利用可能な MCP ツールなし）
     - `az appservice plan list` - App Service プラン（フォールバック）
     - `az functionapp list` - Function アプリ（フォールバック）
     - `az sql server list` - SQL サーバー（フォールバック）
     - `az redis list` - Redis Cache（フォールバック）
     - ... その他のリソース種別も同様

2. **IaC 検出**:
   - `file_search` を使用して IaC ファイルをスキャン: "**/*.bicep", "**/*.tf", "**/main.json", "**/*template*.json"
   - リソース定義を解析して意図された構成を把握する
   - 検出したリソースと比較して不一致を特定する
   - 後続の実装推奨のため IaC ファイルの有無を記録する
   - リポジトリ内の他ファイルは使用せず、IaC ファイルのみを使用すること。他ファイルの使用は、信頼できる情報源ではないため許可されません。
   - IaC ファイルが見つからない場合は、そこで停止し、IaC ファイル未検出をユーザーに報告すること。

3. **構成分析**:
   - 各リソースの現在の SKU、ティア、設定を抽出する
   - リソース間の関係と依存関係を特定する
   - 利用可能な範囲でリソース利用パターンをマッピングする

### ステップ 3: 利用メトリクス収集と現行コスト検証
**アクション**: 利用データを収集し、実際のリソースコストを検証する  
**ツール**: Azure MCP 監視ツール + Azure CLI  
**プロセス**:
1. **監視データソースの特定**:
   - `azmcp-monitor-workspace-list --subscription <id>` を使用して Log Analytics ワークスペースを見つける
   - `azmcp-monitor-table-list --subscription <id> --workspace <name> --table-type "CustomLog"` を使用して利用可能なデータを検出する

2. **利用状況クエリの実行**:
   - `azmcp-monitor-log-query` を以下の定義済みクエリで使用:
     - クエリ: "recent"（最近のアクティビティパターン）
     - クエリ: "errors"（問題を示すエラーレベルログ）
   - カスタム分析には KQL クエリを使用:
   ```kql
   // App Services の CPU 使用率
   AppServiceAppLogs
   | where TimeGenerated > ago(7d)
   | summarize avg(CpuTime) by Resource, bin(TimeGenerated, 1h)
   
   // Cosmos DB の RU 消費量  
   AzureDiagnostics
   | where ResourceProvider == "MICROSOFT.DOCUMENTDB"
   | where TimeGenerated > ago(7d)
   | summarize avg(RequestCharge) by Resource
   
   // ストレージアカウントのアクセスパターン
   StorageBlobLogs
   | where TimeGenerated > ago(7d)
   | summarize RequestCount=count() by AccountName, bin(TimeGenerated, 1d)
   ```

3. **ベースラインメトリクスの算出**:
   - CPU/メモリ利用率の平均
   - データベーススループットのパターン
   - ストレージアクセス頻度
   - Function 実行レート

4. **現行コストの検証**: 
   - ステップ 2 で発見した SKU/ティア構成を使用
   - https://azure.microsoft.com/pricing/ で現行 Azure 価格を確認、または `az billing` コマンドを使用
   - 記録: リソース → 現在の SKU → 推定月額コスト
   - 推奨に進む前に、現実的な現行月額合計を算出する

### ステップ 4: コスト最適化推奨を作成
**アクション**: リソースを分析して最適化機会を特定する  
**ツール**: 収集データを使ったローカル分析  
**プロセス**:
1. **見つかったリソース種別に応じて最適化パターンを適用**:
   
   **コンピュート最適化**:
   - App Service プラン: CPU/メモリ使用量に基づいて適正サイズ化
   - Function アプリ: 低利用なら Premium → Consumption プラン
   - 仮想マシン: 過大なインスタンスをスケールダウン
   
   **データベース最適化**:
   - Cosmos DB: 
     - 変動ワークロードでは Provisioned → Serverless
     - 実利用に基づいた RU/s の適正化
   - SQL Database: DTU 使用量に基づいたサービスティアの適正化
   
   **ストレージ最適化**:
   - ライフサイクルポリシーを実装（Hot → Cool → Archive）
   - 冗長なストレージアカウントを統合
   - アクセスパターンに基づくストレージティアの適正化
   
   **インフラ最適化**:
   - 未使用/冗長リソースを削除
   - 有効な場合はオートスケーリングを実装
   - 非本番環境の稼働スケジュール化

2. **根拠ベースの削減額を算出**: 
   - 検証済み現行コスト → 目標コスト = 削減額
   - 現行構成と目標構成の両方で価格情報源を記録

3. **各推奨の優先度スコアを算出**:
   ```
   Priority Score = (Value Score × Monthly Savings) / (Risk Score × Implementation Days)
   
   High Priority: Score > 20
   Medium Priority: Score 5-20
   Low Priority: Score < 5
   ```

4. **推奨内容の検証**:
   - Azure CLI コマンドが正確であることを確認
   - 推定削減額計算を検証
   - 実装リスクと前提条件を評価
   - すべての削減額計算に裏付け根拠があることを確認

### ステップ 5: ユーザー確認
**アクション**: GitHub issue 作成前に要約を提示して承認を得る  
**プロセス**:
1. **最適化サマリーの表示**:
   ```
   🎯 Azure Cost Optimization Summary
   
   📊 Analysis Results:
   • Total Resources Analyzed: X
   • Current Monthly Cost: $X 
   • Potential Monthly Savings: $Y 
   • Optimization Opportunities: Z
   • High Priority Items: N
   
   🏆 Recommendations:
   1. [Resource]: [Current SKU] → [Target SKU] = $X/month savings - [Risk Level] | [Implementation Effort]
   2. [Resource]: [Current Config] → [Target Config] = $Y/month savings - [Risk Level] | [Implementation Effort]
   3. [Resource]: [Current Config] → [Target Config] = $Z/month savings - [Risk Level] | [Implementation Effort]
   ... and so on
   
   💡 This will create:
   • Y individual GitHub issues (one per optimization)
   • 1 EPIC issue to coordinate implementation
   
   ❓ Proceed with creating GitHub issues? (y/n)
   ```

2. **ユーザー確認待ち**: ユーザーが確認した場合のみ続行

### ステップ 6: 個別最適化 issue を作成
**アクション**: 各最適化機会ごとに個別の GitHub issue を作成する。ラベルは "cost-optimization"（緑色）、"azure"（青色）を付与する。  
**必要な MCP ツール**: 各推奨に対して `create_issue`  
**プロセス**:
1. **以下テンプレートで個別 issue を作成**:

   **タイトル形式**: `[COST-OPT] [Resource Type] - [Brief Description] - $X/month savings`
   
   **本文テンプレート**:
   ```markdown
   ## 💰 Cost Optimization: [Brief Title]
   
   **Monthly Savings**: $X | **Risk Level**: [Low/Medium/High] | **Implementation Effort**: X days
   
   ### 📋 Description
   [Clear explanation of the optimization and why it's needed]
   
   ### 🔧 Implementation
   
   **IaC Files Detected**: [Yes/No - based on file_search results]
   
   ```bash
   # If IaC files found: Show IaC modifications + deployment
   # File: infrastructure/bicep/modules/app-service.bicep
   # Change: sku.name: 'S3' → 'B2'
   az deployment group create --resource-group [rg] --template-file infrastructure/bicep/main.bicep
   
   # If no IaC files: Direct Azure CLI commands + warning
   # ⚠️ No IaC files found. If they exist elsewhere, modify those instead.
   az appservice plan update --name [plan] --sku B2
   ```
   
   ### 📊 Evidence
   - Current Configuration: [details]
   - Usage Pattern: [evidence from monitoring data]
   - Cost Impact: $X/month → $Y/month
   - Best Practice Alignment: [reference to Azure best practices if applicable]
   
   ### ✅ Validation Steps
   - [ ] Test in non-production environment
   - [ ] Verify no performance degradation
   - [ ] Confirm cost reduction in Azure Cost Management
   - [ ] Update monitoring and alerts if needed
   
   ### ⚠️ Risks & Considerations
   - [Risk 1 and mitigation]
   - [Risk 2 and mitigation]
   
   **Priority Score**: X | **Value**: X/10 | **Risk**: X/10
   ```

### ステップ 7: EPIC 連携 issue を作成
**アクション**: すべての最適化作業を追跡するためのマスター issue を作成する。ラベルは "cost-optimization"（緑色）、"azure"（青色）、"epic"（紫色）を付与する。  
**必要な MCP ツール**: EPIC 用に `create_issue`  
**mermaid 図に関する注意**: mermaid 構文が正しいことを確認し、アクセシビリティガイドライン（スタイル、色など）を考慮して図を作成すること。  
**プロセス**:
1. **EPIC issue を作成**:

   **タイトル**: `[EPIC] Azure Cost Optimization Initiative - $X/month potential savings`
   
   **本文テンプレート**:
   ```markdown
   # 🎯 Azure Cost Optimization EPIC
   
   **Total Potential Savings**: $X/month | **Implementation Timeline**: X weeks
   
   ## 📊 Executive Summary
   - **Resources Analyzed**: X
   - **Optimization Opportunities**: Y  
   - **Total Monthly Savings Potential**: $X
   - **High Priority Items**: N
   
   ## 🏗️ Current Architecture Overview
   
   ```mermaid
   graph TB
       subgraph "Resource Group: [name]"
           [Generated architecture diagram showing current resources and costs]
       end
   ```
   
   ## 📋 Implementation Tracking
   
   ### 🚀 High Priority (Implement First)
   - [ ] #[issue-number]: [Title] - $X/month savings
   - [ ] #[issue-number]: [Title] - $X/month savings
   
   ### ⚡ Medium Priority 
   - [ ] #[issue-number]: [Title] - $X/month savings
   - [ ] #[issue-number]: [Title] - $X/month savings
   
   ### 🔄 Low Priority (Nice to Have)
   - [ ] #[issue-number]: [Title] - $X/month savings
   
   ## 📈 Progress Tracking
   - **Completed**: 0 of Y optimizations
   - **Savings Realized**: $0 of $X/month
   - **Implementation Status**: Not Started
   
   ## 🎯 Success Criteria
   - [ ] All high-priority optimizations implemented
   - [ ] >80% of estimated savings realized
   - [ ] No performance degradation observed
   - [ ] Cost monitoring dashboard updated
   
   ## 📝 Notes
   - Review and update this EPIC as issues are completed
   - Monitor actual vs. estimated savings
   - Consider scheduling regular cost optimization reviews
   ```

## エラーハンドリング
- **コスト検証**: 削減見積もりに裏付け根拠がない、または Azure 価格と整合しない場合は、先に進む前に構成と価格情報源を再検証する
- **Azure 認証失敗**: 手動の Azure CLI セットアップ手順を提示する
- **リソース未検出**: Azure リソースのデプロイに関する情報提供 issue を作成する
- **GitHub 作成失敗**: 整形済みの推奨事項をコンソールに出力する
- **利用データ不足**: 制約を明記し、構成ベースの推奨のみを提供する

## 成功基準
- ✅ すべてのコスト見積もりが実際のリソース構成と Azure 価格で検証されている
- ✅ 各最適化ごとに個別 issue が作成されている（追跡・アサイン可能）
- ✅ EPIC issue が包括的な調整と追跡を提供している
- ✅ すべての推奨に、具体的で実行可能な Azure CLI コマンドが含まれている
- ✅ 優先度スコアリングにより ROI 重視の実装が可能になっている
- ✅ アーキテクチャ図が現状を正確に表している
- ✅ ユーザー確認により不要な issue 作成を防止できている

