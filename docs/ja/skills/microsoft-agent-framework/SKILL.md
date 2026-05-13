---
name: microsoft-agent-framework
description: '共有ガイダンスと.NETおよびPython向けの言語別リファレンスを使用して、Microsoft Agent Frameworkソリューションの作成、更新、リファクタリング、説明、レビューを行います。'
---

# Microsoft Agent Framework

Microsoft Agent Frameworkをベースに構築されたアプリケーション、エージェント、ワークフロー、またはマイグレーションに取り組む際にこのスキルを使用してください。

Microsoft Agent FrameworkはSemantic KernelとAutoGenの統合後継であり、それらの強みを新機能と組み合わせています。まだパブリックプレビュー段階で急速に変化しているため、実装アドバイスは古い知識に頼らず、常に最新の公式ドキュメントとサンプルに基づいて行ってください。

## まず対象言語を決定する

推奨やコード変更を行う前に言語ワークフローを選択してください：

1. リポジトリに`.cs`、`.csproj`、`.sln`、`.slnx`などの.NETプロジェクトファイルが含まれている場合、またはユーザーが明示的にC#や.NETのガイダンスを求めている場合は、**.NET**ワークフローを使用します。[references/dotnet.md](references/dotnet.md)に従ってください。
2. リポジトリに`.py`、`pyproject.toml`、`requirements.txt`が含まれている場合、またはユーザーが明示的にPythonのガイダンスを求めている場合は、**Python**ワークフローを使用します。[references/python.md](references/python.md)に従ってください。
3. 両方のエコシステムが含まれている場合は、編集対象のファイルで使われている言語かユーザーの指定した対象言語に合わせます。
4. 言語が曖昧な場合は、まず現在のワークスペースを調査し、最も近い言語別リファレンスを選択してください。

## 常に最新のドキュメントを参照する

- まずMicrosoft Agent Frameworkの概要を読む：<https://learn.microsoft.com/agent-framework/overview/agent-framework-overview>
- 現行のAPIサーフェスについては公式ドキュメントとサンプルを優先してください。
- Microsoft DocsのMCPツールを利用できる場合は、最新のフレームワークガイダンスと例を取得してください。
- 古いSemantic KernelやAutoGenのパターンは移行の参考として扱い、標準実装モデルとしては使わないでください。

## 共通ガイダンス

Microsoft Agent Frameworkをどの言語で扱う場合でも：

- エージェントやワークフローの操作には非同期パターンを使用してください。
- 明示的なエラーハンドリングとログ記録を実装してください。
- 強い型付け、明確なインターフェース、保守しやすい構成パターンを優先してください。
- Azure認証が適切な場合は`DefaultAzureCredential`を使用してください。
- エージェントは自律的な意思決定、アドホックな計画、会話フロー、ツール利用、MCPサーバーとのやり取りに使用してください。
- ワークフローは複数ステップのオーケストレーション、事前定義された実行グラフ、長時間実行タスク、人間の介入があるシナリオに使用してください。
- Azure AI Foundry、Azure OpenAI、OpenAIなどのモデルプロバイダーをサポートしますが、新規プロジェクトではユーザーのニーズに合う場合Azure AI Foundryサービスを優先してください。
- 問題に適合する場合はスレッドベースまたは同等の状態管理、コンテキストプロバイダー、ミドルウェア、チェックポイント、ルーティング、オーケストレーションパターンを使用してください。

## マイグレーションガイダンス

- Semantic Kernelからの移行の場合は公式移行ガイドを使用してください：<https://learn.microsoft.com/agent-framework/migration-guide/from-semantic-kernel/>
- AutoGenからの移行の場合は公式移行ガイドを使用してください：<https://learn.microsoft.com/agent-framework/migration-guide/from-autogen/>
- 振る舞いをまず保持し、その後ネイティブなAgent Frameworkパターンを段階的に採用してください。

## ワークフロー

1. 対象言語を決定し、対応するリファレンスファイルを読みます。
2. 実装の選択を行う前に最新の公式ドキュメントとサンプルを取得します。
3. このスキルの共通エージェントおよびワークフローガイダンスを適用します。
4. 選択したリファレンスの言語別パッケージ、リポジトリ、サンプルパス、コーディング慣行を使用します。
5. リポジトリ内の例が現行ドキュメントと異なる場合は、その違いを説明し、現在サポートされているパターンに従います。

## リファレンス

- [.NETリファレンス](references/dotnet.md)
- [Pythonリファレンス](references/python.md)

## 完了基準

- 推奨事項が対象言語に合致していること。
- パッケージ名、リポジトリパス、サンプルの場所が選択したエコシステムに合っていること。
- ガイダンスがレガシーな前提ではなく、現行のMicrosoft Agent Frameworkドキュメントを反映していること。
- マイグレーションアドバイスは関連する場合にのみSemantic KernelとAutoGenを言及していること。
