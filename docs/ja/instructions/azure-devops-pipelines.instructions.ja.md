---
description: 'Azure DevOps パイプライン YAML ファイルのベストプラクティス'
applyTo: '**/azure-pipelines.yml, **/azure-pipelines*.yml, **/*.pipeline.yml'
---

# Azure DevOps パイプライン YAML のベストプラクティス

## 一般的なガイドライン

- 適切なインデント (スペース 2 つ) を使用して YAML 構文を一貫して使用します。
- パイプライン、ステージ、ジョブ、ステップには意味のある名前と表示名を常に含めてください。
- 適切なエラー処理と条件付き実行を実装する
- 変数とパラメータを使用してパイプラインを再利用可能および保守可能にする
- サービス接続とアクセス許可については最小特権の原則に従ってください
- トラブルシューティングのための包括的なログと診断を含める

## パイプライン構造

- ステージを使用して複雑なパイプラインを整理し、視覚化と制御を改善します
- ジョブを使用して関連するステップをグループ化し、可能な場合は並列実行を有効にします。
- ステージとジョブ間の適切な依存関係を実装する
- 再利用可能なパイプラインコンポーネントのテンプレートを使用する
- パイプラインファイルの焦点を絞ってモジュール化する - 大規模なパイプラインを複数のファイルに分割する

## ベストプラクティスを構築する

- 一貫性を保つために特定のエージェントプール バージョンと VM イメージを使用する
- 依存関係 (npm、NuGet、Maven など) をキャッシュしてビルドのパフォーマンスを向上させる
- 意味のある名前と保持ポリシーを使用して、適切なアーティファクト管理を実装します。
- バージョン番号とビルドメタデータにビルド変数を使用する
- コード品質ゲート (リンティング、テスト、セキュリティスキャン) を含める
- ビルドが再現可能で環境に依存しないことを保証する

## 統合のテスト

- ビルドプロセスの一部として単体テストを実行する
- テスト結果を標準形式 (JUnit、VSTest など) で公開します。
- コードカバレッジレポートと品質ゲートを含める
- 適切な段階で統合およびエンドツーエンドのテストを実装する
- 可能な場合はテスト影響分析を使用して、テスト実行を最適化します。
- テストの失敗時にフェイルファストで迅速なフィードバックを提供します

## セキュリティに関する考慮事項

- 機密性の高い構成とシークレットには Azure Key Vault を使用する
- 可変グループによる適切なシークレット管理の実装
- 必要最小権限でサービス接続を使用する
- セキュリティスキャンを有効にする (依存関係の脆弱性、静的分析)
- 実稼働デプロイメントに承認ゲートを実装する
- 可能な場合は、サービスプリンシパルの代わりにマネージド ID を使用します。

## 導入戦略

- 適切な環境プロモーション（開発→ステージング→本番）を実装する
- 適切な環境ターゲットを指定した展開ジョブを使用する
- 必要に応じて、Blue-Green または Canary デプロイメント戦略を実装します。
- ロールバックメカニズムとヘルスチェックを含める
- コードとしてのインフラストラクチャ (ARM、Bicep、Terraform) を使用して一貫したデプロイメントを実現
- 環境ごとに適切な構成管理を実装する

## 変数とパラメータの管理

- 変数グループを使用してパイプライン間で構成を共有する
- 柔軟なパイプライン実行のためのランタイムパラメーターを実装する
- ブランチまたは環境に基づいて条件変数を使用する
- 機密変数を保護し、シークレットとしてマークする
- 変数の目的と期待値を文書化する
- 複雑な変数ロジックには変数テンプレートを使用する

## パフォーマンスの最適化

- 必要に応じて並列ジョブとマトリックス戦略を使用する
- 依存関係に対する適切なキャッシュ戦略を実装し、出力を構築する
- 完全な履歴が必要ない場合は、Git 操作にシャロークローンを使用する
- マルチステージビルドとレイヤーキャッシュを使用して Docker イメージビルドを最適化します。
- パイプラインのパフォーマンスを監視し、ボトルネックを最適化する
- パイプラインリソーストリガーを効率的に使用する

## 監視と可観測性

- パイプライン全体に包括的なログを含める
- デプロイメントの追跡に Azure Monitor と Application Insights を使用する
- 失敗と成功に対する適切な通知戦略を実装する
- デプロイメントのヘルスチェックと自動ロールバックトリガーを含める
- パイプライン分析を使用して改善の機会を特定する
- パイプラインの動作とトラブルシューティング手順を文書化する

## テンプレートと再利用性

- 一般的なパターンのパイプラインテンプレートを作成する
- 完全なパイプライン継承のために拡張テンプレートを使用する
- 再利用可能なタスクシーケンス用のステップテンプレートを実装する
- 複雑な変数ロジックには変数テンプレートを使用する
- 安定性を確保するためにテンプレートを適切にバージョンアップする
- ドキュメントテンプレートのパラメータと使用例

## ブランチ＆トリガー戦略

- さまざまなブランチタイプに適切なトリガーを実装する
- パスフィルターを使用して、関連ファイルが変更された場合にのみビルドをトリガーする
- メイン/マスターブランチに適切な CI/CD トリガーを構成する
- コード検証にプルリクエスト トリガーを使用する
- メンテナンスタスク用にスケジュールされたトリガーを実装する
- マルチリポジトリシナリオのリソーストリガーを検討する

## 構造例

```yaml
# azure-pipelines.yml
trigger:
  branches:
    include:
      - main
      - develop
  paths:
    exclude:
      - docs/*
      - README.md

variables:
  - group: shared-variables
  - name: buildConfiguration
    value: 'Release'

stages:
  - stage: Build
    displayName: 'Build and Test'
    jobs:
      - job: Build
        displayName: 'Build Application'
        pool:
          vmImage: 'ubuntu-latest'
        steps:
          - task: UseDotNet@2
            displayName: 'Use .NET SDK'
            inputs:
              version: '8.x'
          
          - task: DotNetCoreCLI@2
            displayName: 'Restore dependencies'
            inputs:
              command: 'restore'
              projects: '**/*.csproj'
          
          - task: DotNetCoreCLI@2
            displayName: 'Build application'
            inputs:
              command: 'build'
              projects: '**/*.csproj'
              arguments: '--configuration $(buildConfiguration) --no-restore'

  - stage: Deploy
    displayName: 'Deploy to Staging'
    dependsOn: Build
    condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
    jobs:
      - deployment: DeployToStaging
        displayName: 'Deploy to Staging Environment'
        environment: 'staging'
        strategy:
          runOnce:
            deploy:
              steps:
                - download: current
                  displayName: 'Download drop artifact'
                  artifact: drop
                - task: AzureWebApp@1
                  displayName: 'Deploy to Azure Web App'
                  inputs:
                    azureSubscription: 'staging-service-connection'
                    appType: 'webApp'
                    appName: 'myapp-staging'
                    package: '$(Pipeline.Workspace)/drop/**/*.zip'
```

## 避けるべき一般的なアンチパターン

- 機密性の高い値を YAML ファイルに直接ハードコーディングする
- 不必要なビルドを引き起こす広範すぎるトリガーを使用する
- 単一ステージでのビルドとデプロイのロジックの混合
- 適切なエラー処理とクリーンアップが実装されていない
- アップグレード計画なしで非推奨のタスクバージョンを使用する
- 維持が難しいモノリシックなパイプラインの作成
- 明確にするために適切な命名規則を使用していない
- パイプラインセキュリティのベストプラクティスを無視する
