# Microsoft Agent Framework for .NET

対象プロジェクトがC#またはその他の.NET言語で書かれている場合は、このリファレンスを使用してください。

## 公式情報源

- リポジトリ: <https://github.com/microsoft/agent-framework/tree/main/dotnet>
- サンプル: <https://github.com/microsoft/agent-framework/tree/main/dotnet/samples>

## インストール

新しいプロジェクトの場合、以下のコマンドでパッケージをインストールします。

```bash
dotnet add package Microsoft.Agents.AI
```

## .NET固有のガイダンス

- エージェント操作やワークフロー実行には、一貫して`async`/`await`パターンを使用してください。
- .NETの型安全性や依存性注入の慣習に従ってください。
- サービス登録、構成、認証は標準的な.NETホスティングパターンに沿って管理してください。
- ミドルウェア、コンテキストプロバイダー、オーケストレーションコンポーネントは.NETアプリケーションモデルに即した方法で使用してください。
- 新しいAPIやワークフローパターンを導入する前に、最新の.NETサンプルを確認してください。
