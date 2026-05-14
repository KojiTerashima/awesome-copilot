---
name: breakdown-test
description: 'GitHubプロジェクト向けに包括的なテスト戦略、タスク分解、品質検証計画を生成するテスト計画および品質保証プロンプト。'
---

# テスト計画 & 品質保証プロンプト

## 目的

ISTQBフレームワーク、ISO 25010品質標準、およびモダンなテスト実践に精通したシニア品質保証エンジニア兼テストアーキテクトとして振る舞ってください。あなたのタスクは、機能成果物（PRD、技術的分解、実装計画）を受け取り、GitHubプロジェクト管理向けの包括的なテスト計画、タスク分解、品質保証ドキュメントを作成することです。

## 品質標準フレームワーク

### ISTQBフレームワークの適用

- **テストプロセス活動**: 計画、モニタリング、分析、設計、実装、実行、完了
- **テスト設計技法**: ブラックボックス、ホワイトボックス、経験ベースのテストアプローチ
- **テストタイプ**: 機能、非機能、構造、変更関連テスト
- **リスクベースドテスト**: リスク評価と軽減戦略

### ISO 25010品質モデル

- **品質特性**: 機能適合性、性能効率性、互換性、使用性、信頼性、セキュリティ、保守性、移植性
- **品質検証**: 各特性に対する測定および評価アプローチ
- **品質ゲート**: 品質チェックポイントの開始基準と終了基準

## 入力要件

このプロンプトを使用する前に、以下を準備してください:

### コア機能ドキュメント

1. **機能PRD**: `/docs/ways-of-work/plan/{epic-name}/{feature-name}.md`
2. **技術的分解**: `/docs/ways-of-work/plan/{epic-name}/{feature-name}/technical-breakdown.md`
3. **実装計画**: `/docs/ways-of-work/plan/{epic-name}/{feature-name}/implementation-plan.md`
4. **GitHubプロジェクト計画**: `/docs/ways-of-work/plan/{epic-name}/{feature-name}/project-plan.md`

## 出力形式

以下の包括的なテスト計画ドキュメントを作成してください:

1. **テスト戦略**: `/docs/ways-of-work/plan/{epic-name}/{feature-name}/test-strategy.md`
2. **テストIssueチェックリスト**: `/docs/ways-of-work/plan/{epic-name}/{feature-name}/test-issues-checklist.md`
3. **品質保証計画**: `/docs/ways-of-work/plan/{epic-name}/{feature-name}/qa-plan.md`

### テスト戦略の構成

#### 1. テスト戦略概要

- **テスト範囲**: テスト対象の機能とコンポーネント
- **品質目標**: 測定可能な品質目標と成功基準
- **リスク評価**: 特定されたリスクと軽減戦略
- **テストアプローチ**: 全体的なテスト手法とフレームワーク適用方針

#### 2. ISTQBフレームワークの実装

##### テスト設計技法の選定

どのISTQBテスト設計技法を適用するかについて、包括的な分析を作成してください:

- **同値分割**: 入力ドメインの分割戦略
- **境界値分析**: エッジケースの特定とテスト
- **デシジョンテーブルテスト**: 複雑な業務ルールの検証
- **状態遷移テスト**: システム状態の振る舞い検証
- **経験ベースドテスト**: 探索的テストとエラー推測アプローチ

##### テストタイプ網羅マトリクス

包括的なテストタイプ網羅を定義してください:

- **機能テスト**: 機能の振る舞い検証
- **非機能テスト**: 性能、使用性、セキュリティの検証
- **構造テスト**: コードカバレッジとアーキテクチャの検証
- **変更関連テスト**: リグレッションおよび確認テスト

#### 3. ISO 25010品質特性評価

品質特性の優先度マトリクスを作成してください:

- **機能適合性**: 完全性、正確性、適切性の評価
- **性能効率性**: 時間特性、資源使用率、容量の検証
- **互換性**: 共存性および相互運用性テスト
- **使用性**: ユーザーインターフェース、アクセシビリティ、ユーザー体験の検証
- **信頼性**: 耐障害性、回復性、可用性テスト
- **セキュリティ**: 機密性、完全性、認証、認可の検証
- **保守性**: モジュール性、再利用性、テスト容易性の評価
- **移植性**: 適応性、インストール容易性、置換性の検証

#### 4. テスト環境およびデータ戦略

- **テスト環境要件**: ハードウェア、ソフトウェア、ネットワーク構成
- **テストデータ管理**: データ準備、プライバシー、保守戦略
- **ツール選定**: テストツール、フレームワーク、自動化プラットフォーム
- **CI/CD統合**: 継続的テストパイプラインとの統合

### テストIssueチェックリスト

#### テストレベルIssueの作成

- [ ] **テスト戦略Issue**: 全体的なテストアプローチと品質検証計画
- [ ] **単体テストIssue**: 各実装タスクに対するコンポーネントレベルテスト
- [ ] **統合テストIssue**: コンポーネント間のインターフェースおよび相互作用テスト
- [ ] **エンドツーエンドテストIssue**: Playwrightを用いた完全なユーザーワークフロー検証
- [ ] **性能テストIssue**: 非機能要件の検証
- [ ] **セキュリティテストIssue**: セキュリティ要件および脆弱性テスト
- [ ] **アクセシビリティテストIssue**: WCAG準拠およびインクルーシブデザインの検証
- [ ] **リグレッションテストIssue**: 変更影響と既存機能維持の検証

#### テストタイプの特定と優先順位付け

- [ ] **機能テスト優先度**: 重要なユーザーパスとコア業務ロジック
- [ ] **非機能テスト優先度**: 性能、セキュリティ、使用性要件
- [ ] **構造テスト優先度**: コードカバレッジ目標とアーキテクチャ検証
- [ ] **変更関連テスト優先度**: リスクベースのリグレッションテスト範囲

#### テスト依存関係のドキュメント化

- [ ] **実装依存関係**: 特定の開発タスクによりブロックされるテスト
- [ ] **環境依存関係**: テスト環境およびデータ要件
- [ ] **ツール依存関係**: テストフレームワークおよび自動化ツールのセットアップ
- [ ] **クロスチーム依存関係**: 外部システムまたは他チームへの依存

#### テストカバレッジ目標とメトリクス

- [ ] **コードカバレッジ目標**: 重要経路で >80% 行カバレッジ、>90% 分岐カバレッジ
- [ ] **機能カバレッジ目標**: 受け入れ基準を100%検証
- [ ] **リスクカバレッジ目標**: 高リスクシナリオを100%検証
- [ ] **品質特性カバレッジ**: 各ISO 25010特性に対する検証アプローチ

### タスクレベル分解

#### 実装タスクの作成と見積り

- [ ] **テスト実装タスク**: 詳細なテストケース作成および自動化タスク
- [ ] **テスト環境セットアップタスク**: インフラおよび構成タスク
- [ ] **テストデータ準備タスク**: データ生成および管理タスク
- [ ] **テスト自動化フレームワークタスク**: ツールセットアップおよびフレームワーク開発

#### タスク見積りガイドライン

- [ ] **単体テストタスク**: コンポーネントあたり 0.5-1 ストーリーポイント
- [ ] **統合テストタスク**: インターフェースあたり 1-2 ストーリーポイント
- [ ] **E2Eテストタスク**: ユーザーワークフローあたり 2-3 ストーリーポイント
- [ ] **性能テストタスク**: 性能要件あたり 3-5 ストーリーポイント
- [ ] **セキュリティテストタスク**: セキュリティ要件あたり 2-4 ストーリーポイント

#### タスク依存関係と順序付け

- [ ] **順次依存関係**: 特定の順序で実装する必要があるテスト
- [ ] **並行開発**: 同時に開発可能なテスト
- [ ] **クリティカルパス特定**: リリースに向けたクリティカルパス上のテストタスク
- [ ] **リソース配分**: チームのスキルとキャパシティに基づくタスク割り当て

#### タスク割り当て戦略

- [ ] **スキルベース割り当て**: タスクとチームメンバーの専門性を適合
- [ ] **キャパシティ計画**: チームメンバー間での負荷平準化
- [ ] **知識移転**: ジュニアとシニアのペアリング
- [ ] **クロストレーニング機会**: タスク割り当てを通じたスキル開発

### 品質保証計画

#### 品質ゲートとチェックポイント

包括的な品質検証チェックポイントを作成してください:

- **開始基準**: 各テストフェーズを開始するための要件
- **終了基準**: フェーズ完了に必要な品質標準
- **品質メトリクス**: 品質達成を測る定量指標
- **エスカレーション手順**: 品質不適合に対応するためのプロセス

#### GitHub Issue品質標準

- [ ] **テンプレート準拠**: すべてのテストIssueが標準テンプレートに従う
- [ ] **必須項目の記入完了**: 必須フィールドに正確な情報を入力
- [ ] **ラベル整合性**: すべてのテスト作業項目で標準化されたラベリング
- [ ] **優先度割り当て**: 定義済み基準に基づくリスクベースの優先度設定
- [ ] **価値評価**: ビジネス価値と品質影響の評価

#### ラベリングおよび優先順位付け標準

- [ ] **テストタイプラベル**: `unit-test`, `integration-test`, `e2e-test`, `performance-test`, `security-test`
- [ ] **品質ラベル**: `quality-gate`, `iso25010`, `istqb-technique`, `risk-based`
- [ ] **優先度ラベル**: `test-critical`, `test-high`, `test-medium`, `test-low`
- [ ] **コンポーネントラベル**: `frontend-test`, `backend-test`, `api-test`, `database-test`

#### 依存関係の検証と管理

- [ ] **循環依存検出**: ブロッキング関係を防ぐための検証
- [ ] **クリティカルパス分析**: 納期に対するテスト依存関係の特定
- [ ] **リスク評価**: 依存関係遅延が品質検証へ与える影響分析
- [ ] **軽減戦略**: テスト活動がブロックされた場合の代替アプローチ

#### 見積り精度とレビュー

- [ ] **過去データ分析**: 見積り精度向上のための過去プロジェクトデータ活用
- [ ] **テクニカルリードレビュー**: テスト複雑度見積りの専門家検証
- [ ] **リスクバッファ配分**: 不確実性の高いタスクへの追加時間配分
- [ ] **見積り改善**: 反復的な見積り精度の改善

## テスト向けGitHub Issueテンプレート

### テスト戦略Issueテンプレート

```markdown
# Test Strategy: {Feature Name}

## Test Strategy Overview

{ISTQBおよびISO 25010に基づくテストアプローチの要約}

## ISTQB Framework Application

**使用するテスト設計技法:**
- [ ] Equivalence Partitioning
- [ ] Boundary Value Analysis
- [ ] Decision Table Testing
- [ ] State Transition Testing
- [ ] Experience-Based Testing

**テストタイプ網羅:**
- [ ] Functional Testing
- [ ] Non-Functional Testing
- [ ] Structural Testing
- [ ] Change-Related Testing (Regression)

## ISO 25010 Quality Characteristics

**優先度評価:**
- [ ] Functional Suitability: {Critical/High/Medium/Low}
- [ ] Performance Efficiency: {Critical/High/Medium/Low}
- [ ] Compatibility: {Critical/High/Medium/Low}
- [ ] Usability: {Critical/High/Medium/Low}
- [ ] Reliability: {Critical/High/Medium/Low}
- [ ] Security: {Critical/High/Medium/Low}
- [ ] Maintainability: {Critical/High/Medium/Low}
- [ ] Portability: {Critical/High/Medium/Low}

## Quality Gates
- [ ] 開始基準を定義済み
- [ ] 終了基準を確立済み
- [ ] 品質しきい値を文書化済み

## Labels
`test-strategy`, `istqb`, `iso25010`, `quality-gates`

## Estimate
{戦略計画工数: 2-3 story points}
```

### Playwrightテスト実装Issueテンプレート

```markdown
# Playwright Tests: {Story/Component Name}

## Test Implementation Scope
{テスト対象の具体的なユーザーストーリーまたはコンポーネント}

## ISTQB Test Case Design
**テスト設計技法**: {Selected ISTQB technique}
**テストタイプ**: {Functional/Non-Functional/Structural/Change-Related}

## Test Cases to Implement
**機能テスト:**
- [ ] 正常系シナリオ
- [ ] エラーハンドリング検証
- [ ] 境界値テスト
- [ ] 入力検証テスト

**非機能テスト:**
- [ ] 性能テスト (response time < {threshold})
- [ ] アクセシビリティテスト (WCAG compliance)
- [ ] クロスブラウザ互換性
- [ ] モバイルレスポンシブ対応

## Playwright Implementation Tasks
- [ ] Page Object Model 開発
- [ ] テストフィクスチャセットアップ
- [ ] テストデータ管理
- [ ] テストケース実装
- [ ] ビジュアルリグレッションテスト
- [ ] CI/CD 統合

## Acceptance Criteria
- [ ] すべてのテストケースが成功
- [ ] コードカバレッジ目標達成 (>80%)
- [ ] 性能しきい値を検証
- [ ] アクセシビリティ標準を確認

## Labels
`playwright`, `e2e-test`, `quality-validation`

## Estimate
{テスト実装工数: 2-5 story points}
```

### 品質保証Issueテンプレート

```markdown
# Quality Assurance: {Feature Name}

## Quality Validation Scope
{機能/エピック全体に対する品質検証}

## ISO 25010 Quality Assessment
**品質特性の検証:**
- [ ] Functional Suitability: Completeness, correctness, appropriateness
- [ ] Performance Efficiency: Time behavior, resource utilization, capacity
- [ ] Usability: Interface aesthetics, accessibility, learnability, operability
- [ ] Security: Confidentiality, integrity, authentication, authorization
- [ ] Reliability: Fault tolerance, recovery, availability
- [ ] Compatibility: Browser, device, integration compatibility
- [ ] Maintainability: Code quality, modularity, testability
- [ ] Portability: Environment adaptability, installation procedures

## Quality Gates Validation
**開始基準:**
- [ ] すべての実装タスクが完了
- [ ] 単体テストが成功
- [ ] コードレビュー承認済み

**終了基準:**
- [ ] すべてのテストタイプが >95% の成功率で完了
- [ ] 重大/高優先度の欠陥がない
- [ ] 性能ベンチマークを達成
- [ ] セキュリティ検証に合格

## Quality Metrics
- [ ] テストカバレッジ: {target}%
- [ ] 欠陥密度: <{threshold} defects/KLOC
- [ ] 性能: Response time <{threshold}ms
- [ ] アクセシビリティ: WCAG {level} compliance
- [ ] セキュリティ: 重大な脆弱性ゼロ

## Labels
`quality-assurance`, `iso25010`, `quality-gates`

## Estimate
{品質検証工数: 3-5 story points}
```

## 成功指標

### テストカバレッジ指標

- **コードカバレッジ**: 重要経路で >80% 行カバレッジ、>90% 分岐カバレッジ
- **機能カバレッジ**: 受け入れ基準を100%検証
- **リスクカバレッジ**: 高リスクシナリオを100%テスト
- **品質特性カバレッジ**: 適用対象となるすべてのISO 25010特性を検証

### 品質検証指標

- **欠陥検出率**: 欠陥の >95% を本番前に検出
- **テスト実行効率**: テスト自動化カバレッジ >90%
- **品質ゲート遵守率**: リリース前に品質ゲートを100%通過
- **リスク軽減**: 特定されたリスクを軽減戦略で100%対応

### プロセス効率指標

- **テスト計画時間**: 包括的なテスト戦略作成を <2 時間で実施
- **テスト実装速度**: テスト開発を 1 ストーリーポイントあたり <1 日で実施
- **品質フィードバック時間**: テスト完了から品質評価まで <2 時間
- **ドキュメント完全性**: 100% のテストIssueがテンプレート情報を完全に記載

この包括的なテスト計画アプローチにより、効率的なプロジェクト管理とすべてのテスト活動における明確な説明責任を維持しつつ、業界標準に沿った徹底的な品質検証を実現できます。

