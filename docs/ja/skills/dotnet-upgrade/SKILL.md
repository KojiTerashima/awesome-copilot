---
name: dotnet-upgrade
description: '包括的な .NET フレームワーク アップグレード分析と実行のための、そのまま使えるプロンプト集'
---

# Project Discovery & Assessment
  - name: "プロジェクト分類分析"
    prompt: "ソリューション内のすべてのプロジェクトを特定し、(`.NET Framework`、`.NET Core`、`.NET Standard`) の種別に分類してください。各 `.csproj` を分析して、現在の `TargetFramework` と SDK 利用状況を確認してください。"

  - name: "依存関係互換性レビュー"
    prompt: "外部および内部依存関係についてフレームワーク互換性を確認してください。依存関係グラフの深さに基づいてアップグレードの複雑さを判断してください。"

  - name: "レガシー パッケージ検出"
    prompt: "`packages.config` を使用しており、`PackageReference` 形式への移行が必要なレガシー プロジェクトを特定してください。"

  # Upgrade Strategy & Sequencing
  - name: "プロジェクト アップグレード順序"
    prompt: "依存の少ないコンポーネントから多いコンポーネントへ、推奨アップグレード順を提案してください。API や Azure Function の移行前に class library のアップグレードをどう切り離すかも提案してください。"

  - name: "段階的戦略の計画"
    prompt: "ロールバック チェックポイント付きの段階的アップグレード戦略を提案してください。プロジェクト構造に応じて **Upgrade Assistant** と **manual upgrades** のどちらを使うべきか評価してください。"

  - name: "進捗追跡セットアップ"
    prompt: "全プロジェクトにわたる build、test、deployment readiness を追跡するためのアップグレード チェックリストを生成してください。"

  # Framework Targeting & Code Adjustments
  - name: "Target Framework 選定"
    prompt: "各プロジェクトに対して適切な `TargetFramework` (例: `net8.0`) を提案してください。非推奨の SDK や build 構成も確認して更新してください。"

  - name: "コード モダナイゼーション分析"
    prompt: "モダナイズが必要なコード パターン (例: `WebHostBuilder` → `HostBuilder`) を特定してください。非推奨になった .NET API やサードパーティ ライブラリの代替案も提案してください。"

  - name: "非同期パターン変換"
    prompt: "性能とスケーラビリティ向上のため、適切な箇所で同期呼び出しを async へ変換する提案をしてください。"

  # NuGet & Dependency Management
  - name: "パッケージ互換性分析"
    prompt: "古くなった、または互換性のない NuGet パッケージを分析し、互換バージョンを提案してください。.NET 8 をサポートしていないサードパーティ ライブラリを特定し、移行パスも提示してください。"

  - name: "共有依存関係戦略"
    prompt: "プロジェクト横断で共有依存関係をアップグレードする戦略を提案してください。レガシー パッケージの使用状況を評価し、Microsoft がサポートする namespace の代替案を提案してください。"

  - name: "推移的依存関係レビュー"
    prompt: "アップグレード後の推移的依存関係と潜在的なバージョン競合を確認してください。依存関係競合を解決する戦略を提案してください。"

  # CI/CD & Build Pipeline Updates
  - name: "パイプライン構成分析"
    prompt: "SDK バージョン固定のための YAML build 定義を分析し、更新案を提案してください。`UseDotNet@2` と `NuGetToolInstaller` タスクの変更案も示してください。"

  - name: "ビルド パイプラインのモダナイゼーション"
    prompt: ".NET 8 移行向けの更新済み build pipeline スニペットを生成してください。main にマージする前に feature branch で検証 build を行うことも提案してください。"

  - name: "CI 自動化の強化"
    prompt: "CI pipeline で test と build 検証を自動化できる箇所を特定してください。継続的インテグレーション検証の戦略を提案してください。"

  # Testing & Validation
  - name: "ビルド検証戦略"
    prompt: "アップグレード済みソリューションが正しく build・実行できることを保証するための検証チェックを提案してください。アップグレード後の unit/integration suite に対する自動 test 実行も推奨してください。"

  - name: "サービス統合検証"
    prompt: "logging、telemetry、service connectivity を確認するための検証ステップを生成してください。後方互換性と実行時挙動を検証する戦略も提案してください。"

  - name: "デプロイ準備確認"
    prompt: "本番展開前の UAT デプロイ確認ステップを推奨してください。アップグレード対象コンポーネントの包括的なテスト シナリオを作成してください。"

  # Breaking Change Analysis
  - name: "API 非推奨検出"
    prompt: "対象バージョン間で非推奨になった API や削除された namespace を特定してください。`.NET Upgrade Assistant` と API Analyzer を使った自動スキャンも提案してください。"

  - name: "API 置き換え戦略"
    prompt: "既知の破壊的変更領域に対する代替 API やライブラリを推奨してください。`Startup.cs` → `Program.cs` のリファクタリングのような構成変更も確認してください。"

  - name: "回帰テスト重点領域"
    prompt: "アップグレードされた API endpoint や service に焦点を当てた回帰テスト シナリオを提案してください。重要機能の検証用テスト計画も作成してください。"

  # Version Control & Commit Strategy
  - name: "ブランチ戦略の計画"
    prompt: "安全にアップグレードし、ロールバック可能にするためのブランチ戦略を推奨してください。部分的・完全なプロジェクト アップグレード向けの commit template も生成してください。"

  - name: "PR 構造の最適化"
    prompt: "構造化された PR (`Upgrade to .NET [Version]`) を作成するためのベスト プラクティスを提案してください。破壊的変更を伴う PR に対する tagging 戦略も示してください。"

  - name: "コード レビュー ガイドライン"
    prompt: "peer review で重点的に見るべき点 (build、test、dependency validation) を推奨してください。効果的な upgrade review のためのチェックリストも作成してください。"

  # Documentation & Communication
  - name: "アップグレード文書化戦略"
    prompt: "各プロジェクトのフレームワーク変更を PR にどう文書化するか提案してください。アップグレード内容と test 結果を要約した自動 release note 生成も提案してください。"

  - name: "関係者コミュニケーション"
    prompt: "利用者に対してバージョン アップグレードと移行タイムラインをどう伝えるか提案してください。依存関係更新と検証結果のための文書テンプレートも生成してください。"

  - name: "進捗追跡システム"
    prompt: "upgrade summary dashboard や markdown checklist の維持方法を提案してください。複数プロジェクトにまたがるアップグレード進捗を追跡するテンプレートも作成してください。"

  # Tools & Automation
  - name: "アップグレード ツール選定"
    prompt: "`.NET Upgrade Assistant`、`dotnet list package --outdated`、`dotnet migrate`、`graph.json` 依存関係可視化を、いつどのように使うべきか推奨してください。"

  - name: "分析スクリプト生成"
    prompt: "アップグレード前に依存関係グラフを分析するためのスクリプトやプロンプトを生成してください。Copilot がアップグレード課題を自動特定するための AI 支援プロンプトも提案してください。"

  - name: "複数リポジトリ検証"
    prompt: "複数リポジトリにまたがって自動化出力を検証する方法を提案してください。エンタープライズ規模のアップグレード向けに標準化された検証ワークフローも作成してください。"

  # Final Validation & Delivery
  - name: "最終ソリューション検証"
    prompt: "最終的にアップグレードされたソリューションがすべての検証チェックを通過することを確認するための手順を生成してください。アップグレード後の本番デプロイ確認手順も提案してください。"

  - name: "デプロイ準備完了確認"
    prompt: "最終 test 結果と build artifact を生成することを推奨してください。プロジェクト横断で build/test/deployment の完了状況を要約するチェックリストも作成してください。"

  - name: "リリース文書"
    prompt: "フレームワーク変更と CI/CD 更新を要約した release note を生成してください。包括的なアップグレード要約文書も作成してください。"

---
