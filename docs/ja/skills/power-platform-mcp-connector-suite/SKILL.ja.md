---
name: power-platform-mcp-connector-suite
description: 'Generate complete Power Platform custom connector with MCP integration for Copilot Studio - includes schema generation, troubleshooting, and validation'
---
# Power Platform MCP コネクタ スイート

Microsoft Copilot Studio のモデル コンテキスト プロトコル統合により、包括的な Power Platform カスタム コネクタ実装を生成します。

## Copilot Studio の MCP 機能

**現在サポートされている内容:**
- ✅ **ツール**: LLM が呼び出すことができる関数 (ユーザーの承認あり)
- ✅ **リソース**: エージェントが読み取ることができるファイルのようなデータ (ツールの出力である必要があります)

**まだサポートされていません:**
- ❌ **プロンプト**: 事前に作成されたテンプレート (将来のサポートに備えて)

## コネクタの生成

以下を使用して完全な Power Platform コネクタを作成します。

**コア ファイル:**
- `apiDefinition.swagger.json` と `x-ms-agentic-protocol: mcp-streamable-1.0`
- `apiProperties.json` コネクタのメタデータと認証を使用
- `script.csx` MCP JSON-RPC 処理用のカスタム C# 変換
- `readme.md` コネクタのドキュメント付き

**MCP の統合:**
- JSON-RPC 2.0通信用のPOST `/mcp`エンドポイント
- McpResponse および McpErrorResponse スキーマ定義
- Copilot Studio の制約準拠 (参照タイプなし、単一タイプ)
- ツール出力としてのリソースの統合 (リソースとツールはサポートされていますが、プロンプトはまだサポートされていません)

## スキーマの検証とトラブルシューティング

**Copilot Studio への準拠のためのスキーマの検証:**
- ✅ ツールの入力/出力に参照タイプ (`$ref`) がありません
- ✅ 単一型の値のみ (`["string", "number"]` ではない)
- ✅ プリミティブ型: 文字列、数値、整数、ブール値、配列、オブジェクト
- ✅ 個別のエンティティではなく、ツールの出力としてのリソース
- ✅ すべてのエンドポイントの完全な URI

**一般的な問題と修正:**
- フィルタリングされたツール → 参照型を削除し、プリミティブを使用
- 型エラー → 検証ロジックを備えた単一型
- リソースが利用できない → ツールの出力に含める
- 接続失敗 → `x-ms-agentic-protocol` ヘッダーを確認

## コンテキスト変数

- **コネクタ名**: [コネクタの表示名]
- **サーバーの目的**: [MCP サーバーが達成すべきこと]
- **必要なツール**: [実装する MCP ツールのリスト]
- **リソース**: [提供するリソースの種類]
- **認証**: [なし、API キー、oauth2、基本]
- **ホスト環境**: [Azure Function、Express.js など]
- **対象 API**: [統合する外部 API]

## 生成モード

### モード 1: 新しいコネクタの完成
CLI 検証セットアップを含む、新しい Power Platform MCP コネクタのすべてのファイルを最初から生成します。

### モード 2: スキーマの検証
paconn および検証ツールを使用して、Copilot Studio に準拠するように既存のスキーマを分析および修正します。

### モード 3: 統合のトラブルシューティング
CLI デバッグ ツールを使用して、Copilot Studio との MCP 統合の問題を診断し、解決します。

### モード 4: ハイブリッド コネクタ
適切な検証ワークフローを使用して、既存の Power Platform コネクタに MCP 機能を追加します。

### モード 5: 認定の準備
完全なメタデータと検証準拠を備えた Microsoft 認定申請用のコネクタを準備します。

### モード 6: OAuth セキュリティ強化
MCP セキュリティのベスト プラクティスと高度なトークン検証で強化された OAuth 2.0 認証を実装します。

## 期待される出力**1. apiDefinition.swagger.json**
- Microsoft 拡張機能を備えた Swagger 2.0 形式
- MCP エンドポイント: `POST /mcp` と適切なプロトコル ヘッダー
- 準拠したスキーマ定義 (プリミティブ型のみ)
- McpResponse/McpErrorResponse の定義

**2. apiProperties.json**
- コネクタのメタデータとブランディング (`iconBrandColor` が必要)
- 認証設定
- MCP 変換用のポリシー テンプレート

**3.スクリプト.csx**
- JSON-RPC 2.0 メッセージ処理
- リクエスト/レスポンスの変換
- MCP プロトコル準拠ロジック
- エラー処理と検証

**4.実装ガイダンス**
- ツールの登録と実行パターン
- リソース管理戦略
- Copilot Studio の統合手順
- テストと検証の手順

## 検証チェックリスト

### 技術的準拠
- MCP エンドポイントの [ ] `x-ms-agentic-protocol: mcp-streamable-1.0`
- [ ] どのスキーマ定義にも参照型がありません
- [ ] すべての型フィールドは単一型 (配列ではありません)
- [ ] ツール出力として含まれるリソース
- [ ] script.csx の JSON-RPC 2.0 準拠
- [ ] 全体にわたる完全な URI エンドポイント
- [ ] Copilot Studio エージェントの明確な説明
- [ ] 認証は正しく設定されています
- [ ] MCP 変換用のポリシー テンプレート
- [ ] ジェネレーティブ オーケストレーションの互換性

### CLI の検証
- [ ] **paconn validate**: `paconn validate --api-def apiDefinition.swagger.json` はエラーなしで合格しました
- [ ] **pac CLI 対応**: コネクタは `pac connector create/update` で作成/更新できます
- [ ] **スクリプト検証**: script.csx は、pac CLI アップロード中に自動検証に合格します。
- [ ] **パッケージの検証**: `ConnectorPackageValidator.ps1` は正常に実行されます

### OAuth とセキュリティ要件
- [ ] **OAuth 2.0 Enhanced**: MCP セキュリティのベスト プラクティス実装を備えた標準 OAuth 2.0
- [ ] **トークン検証**: パススルー攻撃を防ぐためにトークン オーディエンス検証を実装します。
- [ ] **カスタム セキュリティ ロジック**: MCP 準拠のための script.csx での検証の強化
- [ ] **状態パラメータ保護**: CSRF 防止のための安全な状態パラメータ
- [ ] **HTTPS の強制**: すべての運用エンドポイントは HTTPS のみを使用します
- [ ] **MCP セキュリティ実践**: OAuth 2.0 内で混乱した副攻撃防止を実装する

### 認定要件
- [ ] **完全なメタデータ**: 製品およびサービス情報を含む settings.json
- [ ] **アイコンの準拠**: PNG 形式、230x230 または 500x500 の寸法
- [ ] **ドキュメント**: 包括的な例を含む認定に対応した Readme
- [ ] **セキュリティ コンプライアンス**: MCP セキュリティ実践により強化された OAuth 2.0、プライバシー ポリシー
- [ ] **認証フロー**: カスタム セキュリティ検証が適切に構成された OAuth 2.0

## 使用例```ヤムル
モード: 新しいコネクタの完成
コネクタ名: Customer Analytics MCP
サーバーの目的: 顧客データの分析と洞察
必要なツール:
  - searchCustomers: 条件に基づいて顧客を検索します
  - getCustomerProfile: 詳細な顧客データを取得します
  -analyzeCustomerTrends: 傾向分析を生成します。
リソース:
  ・顧客プロフィール（JSONデータ）
  - 分析レポート（構造化データ）
認証: oauth2
ホスト環境：Azure機能
対象API：CRM REST API
「」
