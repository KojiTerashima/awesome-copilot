---
name: devops-rollout-plan
description: 'インフラストラクチャやアプリケーション変更に対して、事前確認、段階的デプロイ、検証シグナル、ロールバック手順、コミュニケーション計画を含む包括的なロールアウト計画を生成します'
---

# DevOps Rollout Plan Generator

あなたの目標は、インフラストラクチャまたはアプリケーション変更のための、包括的で本番投入可能な rollout plan を作成することです。

## Input Requirements

計画を生成する前に、次の詳細を収集してください。

### Change Description
- 何が変わるのか (infrastructure、application、configuration)
- バージョンまたは状態遷移 (from/to)
- 解決する問題または追加される機能

### Environment Details
- 対象環境 (dev、staging、production、all)
- インフラ種別 (Kubernetes、VMs、serverless、containers)
- 影響を受けるサービスと依存関係
- 現在の容量とスケール

### Constraints & Requirements
- 許容できるダウンタイム時間帯
- 変更可能時間帯の制約
- 承認要件
- 規制またはコンプライアンス上の考慮事項

### Risk Assessment
- 変更の blast radius
- データ移行またはスキーマ変更
- ロールバックの複雑さと安全性
- 既知のリスク

## Output Format

次のセクションを持つ構造化 rollout plan を生成してください。

### 1. Executive Summary
- 何を、なぜ、いつ、どのくらいの時間で行うか
- リスク レベルとロールバック所要時間
- 影響を受けるシステムとユーザー影響
- 想定ダウンタイム

### 2. Prerequisites & Approvals
- 必要な承認 (technical lead、security、compliance、business)
- 必要なリソース (capacity、backup、monitoring、rollback automation)
- デプロイ前バックアップ

### 3. Preflight Checks
- インフラ健全性の検証
- アプリケーション健全性のベースライン
- 依存関係の可用性
- monitoring のベースライン メトリクス
- Go/no-go 判定チェックリスト

### 4. Step-by-Step Rollout Procedure
**Phases**: Pre-deployment、deployment、progressive verification
- 各ステップの具体的なコマンド
- 各ステップ後の検証
- 所要時間の見積もり

### 5. Verification Signals
**Immediate** (0-2 min): デプロイ成功、pods/containers 起動、health check 通過
**Short-term** (2-5 min): アプリケーション応答、エラー率許容範囲、レイテンシ正常
**Medium-term** (5-15 min): メトリクス継続安定、接続安定、integration 動作
**Long-term** (15+ min): 劣化なし、capacity 健全、ビジネス メトリクス正常

### 6. Rollback Procedure
**Decision Criteria**: ロールバック開始条件
**Rollback Steps**: 自動化、インフラ差し戻し、または完全復元
**Post-Rollback Verification**: システム健全性が回復したことを確認
**Communication**: 関係者への通知

### 7. Communication Plan
- Pre-deployment (T-24h): 日程と影響の通知
- Deployment start: 開始通知
- Progress updates: X 分ごとの状況更新
- Completion: 成功確認
- Rollback (if needed): 問題発生の通知

**Stakeholder Matrix**: 誰に、いつ、どの方法で、どの内容を通知するか

### 8. Post-Deployment Tasks
- Immediate (1h): 基準達成確認、ログレビュー
- Short-term (24h): メトリクス監視、エラーレビュー
- Medium-term (1 week): デプロイ後レビュー、学びの整理

### 9. Contingency Plans
Scenarios: 部分障害、性能劣化、データ不整合、依存関係障害
各シナリオについて: 症状、対応、タイムライン

### 10. Contact Information
- Primary and secondary on-call
- Escalation path
- Emergency contacts (infrastructure、security、database、networking)

## Plan Customization

次に応じて調整してください。
- **Infrastructure Type**: Kubernetes、VMs、serverless、databases
- **Risk Level**: Low (簡略)、medium (標準)、high (追加ゲートあり)
- **Change Type**: Code deployment、infrastructure、configuration、data migration
- **Environment**: Production (完全版)、staging (簡略版)、development (最小版)

## Remember

- 必ずテスト済みの rollback plan を用意する
- 早めに、そして継続的に連絡する
- ログだけでなくメトリクスを監視する
- すべてを文書化する
- 各デプロイから学ぶ
- 金曜午後にはデプロイしない (緊急時を除く)
- 検証ステップを省略しない
- 「動くはず」と決めつけない
