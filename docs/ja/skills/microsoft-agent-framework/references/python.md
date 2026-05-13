# Microsoft Agent Framework for Python

対象プロジェクトがPythonで書かれている場合は、このリファレンスを使用してください。

## 権威ある情報源

- リポジトリ: <https://github.com/microsoft/agent-framework/tree/main/python>
- サンプル: <https://github.com/microsoft/agent-framework/tree/main/python/samples>

## インストール

新規プロジェクトの場合、以下のコマンドでパッケージをインストールしてください。

```bash
pip install agent-framework
```

## Python固有のガイダンス

- エージェントおよびワークフロー操作全体で最新の非同期パターンを使用してください。
- 型ヒントを追加し、動的コードでもAPIを明示的に保ってください。
- 依存関係やツールのために標準的なPythonパッケージングおよび環境の慣習に従ってください。
- ミドルウェア、コンテキストプロバイダー、およびオーケストレーションパターンをPythonアプリケーション構造に適合する方法で使用してください。
- 新しいAPIやワークフローパターンを導入する前に、最新のPythonサンプルを確認してください。
