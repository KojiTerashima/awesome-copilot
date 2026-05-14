---
name: typescript-mcp-server-generator
description: 'Generate a complete MCP server project in TypeScript with tools, resources, and proper configuration'
---
# TypeScript MCP サーバーを生成する

次の仕様を使用して、TypeScript で完全なモデル コンテキスト プロトコル (MCP) サーバーを作成します。

## 要件

1. **プロジェクト構造**: 適切なディレクトリ構造で新しい TypeScript/Node.js プロジェクトを作成します。
2. **NPM パッケージ**: @modelcontextprotocol/sdk、zod@3、および Express (HTTP 用) または stdio サポートのいずれかを含みます。
3. **TypeScript 構成**: ES モジュールをサポートする適切な tsconfig.json
4. **サーバー タイプ**: HTTP (ストリーミング可能な HTTP トランスポートを使用) または標準入出力サーバーのいずれかを選択します
5. **ツール**: 適切なスキーマ検証を備えた便利なツールを少なくとも 1 つ作成します
6. **エラー処理**: 包括的なエラー処理と検証が含まれます

## 実装の詳細

### プロジェクトのセットアップ
- `npm init`で初期化してpackage.jsonを作成
- 依存関係のインストール: `@modelcontextprotocol/sdk`、`zod@3`、およびトランスポート固有のパッケージ
- ES モジュールを使用して TypeScript を構成します: `"type": "module"` in package.json
- 開発依存関係を追加: 開発用 `tsx` または `ts-node`
- 適切な .gitignore ファイルを作成する

### サーバー構成
- 高度な実装には `McpServer` クラスを使用します
- サーバー名とバージョンを設定します
- 適切なトランスポート (StreamableHTTPServerTransport または StdioServerTransport) を選択します。
- HTTP の場合: 適切なミドルウェアとエラー処理を使用して Express をセットアップします。
- stdio の場合: StdioServerTransport を直接使用します。

### ツールの実装
- `registerTool()` メソッドをわかりやすい名前で使用します
- 入力と出力の検証に zod を使用してスキーマを定義する
- 明確な `title` フィールドと `description` フィールドを提供します
- `content` と `structuredContent` の両方を結果として返します
- try-catch ブロックを使用して適切なエラー処理を実装する
- 必要に応じて非同期操作をサポートします

### リソース/プロンプトのセットアップ (オプション)
- `registerResource()` と動的 URI の ResourceTemplate を使用してリソースを追加します
- `registerPrompt()` と引数スキーマを使用してプロンプトを追加します
- UX を向上させるために補完サポートの追加を検討してください

### コードの品質
- タイプ セーフティのために TypeScript を使用する
- 一貫して非同期/待機パターンに従います
- トランスポート終了イベントで適切なクリーンアップを実装する
- 設定に環境変数を使用する
- 複雑なロジックにはインライン コメントを追加します
- 懸念事項を明確に分離した構造コード## 考慮すべきツールの種類の例
- データの処理と変換
- 外部 API 統合
- ファイル システム操作 (読み取り、検索、分析)
- データベースクエリ
- テキスト分析または要約（サンプリング付き）
- システム情報の取得

## 構成オプション
- **HTTP サーバーの場合**: 
  - 環境変数によるポート設定
  - ブラウザクライアントの CORS セットアップ
  - セッション管理 (ステートレス vs ステートフル)
  - ローカルサーバーのDNSリバインディング保護
  
- **標準入出力サーバーの場合**:
  - 適切な標準入力/標準出力処理
  - 環境ベースの構成
  - プロセスのライフサイクル管理

## テストのガイダンス
- サーバーの実行方法の説明 (`npm start` または `npx tsx server.ts`)
- MCP Inspector コマンドを提供します: `npx @modelcontextprotocol/inspector`
- HTTP サーバーの場合は、接続 URL を含めます: `http://localhost:PORT/mcp`
- ツール呼び出しの例を含める
- 一般的な問題のトラブルシューティングのヒントを追加

## 考慮すべき追加機能
- LLM を利用したツールのサンプリング サポート
- インタラクティブなワークフローのためのユーザー入力の引き出し
- 有効化/無効化機能を備えた動的なツール登録
- 一括更新の通知のデバウンス
- 効率的なデータ参照のためのリソースリンク

包括的なドキュメント、タイプ セーフティ、およびエラー処理を備えた完全な運用準備完了の MCP サーバーを生成します。