---
name: azure-resource-health-diagnose
description: 'Azure リソースの正常性を分析し、ログとテレメトリから問題を診断し、特定された問題の修復計画を作成します。'
---

# Azure リソース正常性と問題診断

このワークフローは、特定の Azure リソースを分析して正常性ステータスを評価し、ログとテレメトリデータを使用して潜在的な問題を診断し、検出された問題に対する包括的な修復計画を作成します。

## 前提条件
- Azure MCP サーバーが構成済みかつ認証済みであること
- 対象の Azure リソースが特定されていること（名前、必要に応じてリソース グループ/サブスクリプション）
- ログ/テレメトリを生成するため、リソースがデプロイ済みで稼働中であること
- 利用可能な場合は、直接 Azure CLI を使うより Azure MCP ツール（`azmcp-*`）を優先すること

## ワークフロー手順

### 手順 1: Azure のベストプラクティスを取得
**アクション**: 診断およびトラブルシューティングのベストプラクティスを取得する  
**ツール**: Azure MCP ベストプラクティス ツール  
**プロセス**:
1. **ベストプラクティスの読み込み**:
   - Azure ベストプラクティス ツールを実行して診断ガイドラインを取得する
   - 正常性監視、ログ分析、問題解決パターンに注目する
   - これらのプラクティスを診断アプローチと修復推奨の策定に活用する

### 手順 2: リソース探索と特定
**アクション**: 対象 Azure リソースを特定して見つける  
**ツール**: Azure MCP ツール + Azure CLI フォールバック  
**プロセス**:
1. **リソース検索**:
   - リソース名のみ提供されている場合: `azmcp-subscription-list` を使ってサブスクリプション横断で検索する
   - `az resource list --name <resource-name>` を使用して一致するリソースを見つける
   - 複数一致した場合、サブスクリプション/リソース グループの指定をユーザーに求める
   - 詳細なリソース情報を収集する:
     - リソースの種類と現在の状態
     - 場所、タグ、構成
     - 関連サービスと依存関係

2. **リソース種別の検出**:
   - 適切な診断アプローチを判断するためにリソース種別を特定する:
     - **Web Apps/Function Apps**: アプリケーション ログ、パフォーマンス メトリック、依存関係トラッキング
     - **Virtual Machines**: システム ログ、パフォーマンス カウンター、ブート診断
     - **Cosmos DB**: リクエスト メトリック、スロットリング、パーティション統計
     - **Storage Accounts**: アクセス ログ、パフォーマンス メトリック、可用性
     - **SQL Database**: クエリ性能、接続ログ、リソース使用率
     - **Application Insights**: アプリケーション テレメトリ、例外、依存関係
     - **Key Vault**: アクセス ログ、証明書状態、シークレット利用状況
     - **Service Bus**: メッセージ メトリック、配信不能キュー、スループット

### 手順 3: 正常性ステータス評価
**アクション**: 現在のリソース正常性と可用性を評価する  
**ツール**: Azure MCP 監視ツール + Azure CLI  
**プロセス**:
1. **基本ヘルスチェック**:
   - リソースのプロビジョニング状態と運用状態を確認する
   - サービスの可用性と応答性を検証する
   - 直近のデプロイまたは構成変更を確認する
   - 現在のリソース使用率（CPU、メモリ、ストレージなど）を評価する

2. **サービス固有の正常性指標**:
   - **Web Apps**: HTTP 応答コード、応答時間、稼働率
   - **Databases**: 接続成功率、クエリ性能、デッドロック
   - **Storage**: 可用性割合、リクエスト成功率、レイテンシ
   - **VMs**: ブート診断、ゲスト OS メトリック、ネットワーク接続性
   - **Functions**: 実行成功率、実行時間、エラー頻度

### 手順 4: ログとテレメトリ分析
**アクション**: ログとテレメトリを分析して問題とパターンを特定する  
**ツール**: Log Analytics クエリ用 Azure MCP 監視ツール  
**プロセス**:
1. **監視ソースの特定**:
   - `azmcp-monitor-workspace-list` を使って Log Analytics ワークスペースを特定する
   - リソースに関連する Application Insights インスタンスを見つける
   - `azmcp-monitor-table-list` を使って関連ログ テーブルを特定する

2. **診断クエリの実行**:
   リソース種別に応じた対象 KQL クエリで `azmcp-monitor-log-query` を使用する:

   **一般的なエラー分析**:
   ```kql
   // 最近のエラーと例外
   union isfuzzy=true 
       AzureDiagnostics,
       AppServiceHTTPLogs,
       AppServiceAppLogs,
       AzureActivity
   | where TimeGenerated > ago(24h)
   | where Level == "Error" or ResultType != "Success"
   | summarize ErrorCount=count() by Resource, ResultType, bin(TimeGenerated, 1h)
   | order by TimeGenerated desc
   ```

   **パフォーマンス分析**:
   ```kql
   // パフォーマンス劣化パターン
   Perf
   | where TimeGenerated > ago(7d)
   | where ObjectName == "Processor" and CounterName == "% Processor Time"
   | summarize avg(CounterValue) by Computer, bin(TimeGenerated, 1h)
   | where avg_CounterValue > 80
   ```

   **アプリケーション固有クエリ**:
   ```kql
   // Application Insights - 失敗したリクエスト
   requests
   | where timestamp > ago(24h)
   | where success == false
   | summarize FailureCount=count() by resultCode, bin(timestamp, 1h)
   | order by timestamp desc
   
   // データベース - 接続失敗
   AzureDiagnostics
   | where ResourceProvider == "MICROSOFT.SQL"
   | where Category == "SQLSecurityAuditEvents"
   | where action_name_s == "CONNECTION_FAILED"
   | summarize ConnectionFailures=count() by bin(TimeGenerated, 1h)
   ```

3. **パターン認識**:
   - 繰り返し発生するエラーパターンや異常を特定する
   - エラーをデプロイ時刻や構成変更と相関分析する
   - パフォーマンス傾向と劣化パターンを分析する
   - 依存関係の障害や外部サービスの問題を確認する

### 手順 5: 問題分類と根本原因分析
**アクション**: 特定した問題を分類し、根本原因を特定する  
**プロセス**:
1. **問題分類**:
   - **Critical**: サービス停止、データ損失、セキュリティ侵害
   - **High**: パフォーマンス劣化、断続的障害、高いエラー率
   - **Medium**: 警告、最適でない構成、軽微な性能問題
   - **Low**: 情報通知、最適化の機会

2. **根本原因分析**:
   - **Configuration Issues**: 設定誤り、依存関係不足
   - **Resource Constraints**: CPU/メモリ/ディスク制約、スロットリング
   - **Network Issues**: 接続問題、DNS 解決、ファイアウォール ルール
   - **Application Issues**: コード不具合、メモリリーク、非効率なクエリ
   - **External Dependencies**: サードパーティ サービス障害、API 制限
   - **Security Issues**: 認証失敗、証明書期限切れ

3. **影響評価**:
   - ビジネス影響と影響を受けるユーザー/システムを特定する
   - データ完全性とセキュリティへの影響を評価する
   - 復旧時間目標と優先順位を評価する

### 手順 6: 修復計画の作成
**アクション**: 特定した問題に対処する包括的な計画を作成する  
**プロセス**:
1. **即時対応**（Critical 問題）:
   - サービス可用性を復旧するための緊急修正
   - 影響を緩和するための暫定回避策
   - 複雑な問題に対するエスカレーション手順

2. **短期修正**（High/Medium 問題）:
   - 構成調整とリソース スケーリング
   - アプリケーション更新とパッチ適用
   - 監視とアラートの改善

3. **長期改善**（すべての問題）:
   - 回復性向上のためのアーキテクチャ変更
   - 予防策と監視強化
   - ドキュメントとプロセスの改善

4. **実装手順**:
   - 具体的な Azure CLI コマンド付きの優先度付きアクション項目
   - テストおよび検証手順
   - 各変更のロールバック計画
   - 問題解消を確認するための監視

### 手順 7: ユーザー確認とレポート作成
**アクション**: 調査結果を提示し、修復アクションの承認を得る  
**プロセス**:
1. **正常性評価サマリーの表示**:
   ```
   🏥 Azure リソース正常性評価
   
   📊 リソース概要:
   • Resource: [Name] ([Type])
   • Status: [Healthy/Warning/Critical]
   • Location: [Region]
   • Last Analyzed: [Timestamp]
   
   🚨 特定された問題:
   • Critical: 直ちに対応が必要な問題 X 件
   • High: パフォーマンス/信頼性に影響する問題 Y 件  
   • Medium: 最適化対象の問題 Z 件
   • Low: 情報項目 N 件
   
   🔍 主要な問題:
   1. [Issue Type]: [Description] - Impact: [High/Medium/Low]
   2. [Issue Type]: [Description] - Impact: [High/Medium/Low]
   3. [Issue Type]: [Description] - Impact: [High/Medium/Low]
   
   🛠️ 修復計画:
   • Immediate Actions: X 項目
   • Short-term Fixes: Y 項目  
   • Long-term Improvements: Z 項目
   • Estimated Resolution Time: [Timeline]
   
   ❓ 詳細な修復計画に進みますか？ (y/n)
   ```

2. **詳細レポートの作成**:
   ```markdown
   # Azure リソース正常性レポート: [Resource Name]
   
   **Generated**: [Timestamp]  
   **Resource**: [Full Resource ID]  
   **Overall Health**: [Status with color indicator]
   
   ## 🔍 エグゼクティブサマリー
   [正常性ステータスと主要な所見の概要]
   
   ## 📊 正常性メトリック
   - **Availability**: 過去 24 時間で X%
   - **Performance**: [平均応答時間/スループット]
   - **Error Rate**: 過去 24 時間で X%
   - **Resource Utilization**: [CPU/Memory/Storage percentages]
   
   ## 🚨 特定された問題
   
   ### Critical 問題
   - **[Issue 1]**: [Description]
     - **Root Cause**: [Analysis]
     - **Impact**: [Business impact]
     - **Immediate Action**: [Required steps]
   
   ### 高優先度問題  
   - **[Issue 2]**: [Description]
     - **Root Cause**: [Analysis]
     - **Impact**: [Performance/reliability impact]
     - **Recommended Fix**: [Solution steps]
   
   ## 🛠️ 修復計画
   
   ### フェーズ 1: 即時対応 (0-2 時間)
   ```bash
   # サービス復旧のための重要修正
   [Azure CLI commands with explanations]
   ```
   
   ### フェーズ 2: 短期修正 (2-24 時間)
   ```bash
   # パフォーマンスと信頼性の改善
   [Azure CLI commands with explanations]
   ```
   
   ### フェーズ 3: 長期改善 (1-4 週間)
   ```bash
   # アーキテクチャおよび予防策
   [Azure CLI commands and configuration changes]
   ```
   
   ## 📈 監視に関する推奨事項
   - **Alerts to Configure**: [推奨アラート一覧]
   - **Dashboards to Create**: [監視ダッシュボードの提案]
   - **Regular Health Checks**: [推奨頻度と範囲]
   
   ## ✅ 検証手順
   - [ ] ログを通じて問題解消を確認する
   - [ ] パフォーマンス改善を確認する
   - [ ] アプリケーション機能をテストする
   - [ ] 監視とアラートを更新する
   - [ ] 得られた知見を文書化する
   
   ## 📝 再発防止策
   - [類似問題を防ぐための推奨事項]
   - [プロセス改善]
   - [監視強化]
   ```

## エラーハンドリング
- **Resource Not Found**: リソース名/場所の指定方法を案内する
- **Authentication Issues**: Azure 認証設定を案内する
- **Insufficient Permissions**: リソースアクセスに必要な RBAC ロールを提示する
- **No Logs Available**: 診断設定の有効化とデータ蓄積待ちを提案する
- **Query Timeouts**: 分析をより小さい時間ウィンドウに分割する
- **Service-Specific Issues**: 制約を明記したうえで汎用的な正常性評価を提供する

## 成功基準
- ✅ リソースの正常性ステータスが正確に評価されている
- ✅ 重要な問題がすべて特定・分類されている
- ✅ 主要な問題の根本原因分析が完了している
- ✅ 具体的な手順を含む実行可能な修復計画が提示されている
- ✅ 監視および再発防止の推奨事項が含まれている
- ✅ ビジネス影響に基づく明確な優先順位付けがされている
- ✅ 実装手順に検証およびロールバック手順が含まれている

