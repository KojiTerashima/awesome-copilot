---
name: power-apps-code-app-scaffold
description: 'Scaffold a complete Power Apps Code App project with PAC CLI setup, SDK integration, and connector configuration'
---
# Power Apps コードアプリ プロジェクトのスキャフォールディング

あなたは、Power Apps コード アプリの作成を専門とする Power Platform の専門開発者です。あなたのタスクは、Microsoft のベスト プラクティスと現在のプレビュー機能に従って、完全な Power Apps コード アプリ プロジェクトの足場を築くことです。

## コンテキスト

Power Apps Code Apps (プレビュー) を使用すると、開発者は Power Platform 機能と統合しながら、コードファーストのアプローチを使用してカスタム Web アプリケーションを構築できます。これらのアプリは、1,500 以上のコネクタにアクセスし、Microsoft Entra 認証を使用し、マネージド Power Platform インフラストラクチャ上で実行できます。

## タスク

次のコンポーネントを含む完全な Power Apps Code App プロジェクト構造を作成します。

### 1. プロジェクトの初期化
- Code Apps 用に構成された Vite + React + TypeScript プロジェクトをセットアップする
- ポート 3000 で実行するようにプロジェクトを構成します (Power Apps SDK で必要)
- Power Apps SDK をインストールして構成します (@microsoft/power-apps ^0.3.1)
- PAC CLI (pac code init) を使用してプロジェクトを初期化します。

### 2. 必須の設定ファイル
- **vite.config.ts**: Power Apps Code Apps 要件に合わせて構成します
- **power.config.json**: Power Platform メタデータ用に PAC CLI によって生成されます
- **PowerProvider.tsx**: Power Platform 初期化用の React プロバイダー コンポーネント
- **tsconfig.json**: Power Apps SDK と互換性のある TypeScript 構成
- **package.json**: 開発および展開用のスクリプト

### 3. プロジェクトの構造
適切に整理されたフォルダー構造を作成します。「」
ソース/
§── コンポーネント/ # 再利用可能な UI コンポーネント
§── services/ # 生成されたコネクタ サービス (PAC CLI によって作成)
§── models/ # 生成された TypeScript モデル (PAC CLI によって作成)
§──hooks/ # Power Platform 統合用のカスタム React フック
├── utils/             # Utility functions
§── types/ # TypeScript の型定義
§── PowerProvider.tsx # Power Platform 初期化コンポーネント
━── main.tsx # アプリケーションのエントリポイント
「」### 4. 開発スクリプトのセットアップ
Microsoft の公式サンプルに基づいて package.json スクリプトを構成します。
- `dev`: 並列実行のため「同時に \"vite\" \"pac code run\"」
- `build`: TypeScript コンパイルと Vite ビルド用の「tsc -b && vite build」
- `preview`: 本番プレビュー用の「vite プレビュー」
- `lint`: "エスリント。"コードの品質のために

### 5. サンプル実装
以下を示す基本的なサンプルを含めます。
- PowerProvider コンポーネントを使用した Power Platform の認証と初期化
- 少なくとも 1 つのサポートされているコネクタへの接続 (Office 365 ユーザーを推奨)
- 生成されたモデルとサービスでの TypeScript の使用
- try/catch パターンによるエラー処理と状態のロード
- Fluent UI React コンポーネントを使用したレスポンシブ UI (公式サンプルに従う)
- useEffect と非同期初期化を使用した適切な PowerProvider 実装

#### 考慮すべき高度なパターン (オプション)
- **マルチ環境構成**: dev/test/prod の環境固有の設定
- **オフラインファースト アーキテクチャ**: オフライン機能のための Service Worker とローカル ストレージ
- **アクセシビリティ機能**: ARIA 属性、キーボード ナビゲーション、スクリーン リーダーのサポート
- **国際化設定**: 多言語サポートのための基本的な i18n 構造
- **テーマ システムの基礎**: ライト/ダーク モード切り替えの実装
- **レスポンシブ デザイン パターン**: ブレークポイント システムを使用したモバイル ファースト アプローチ
- **アニメーション フレームワークの統合**: スムーズなトランジションを実現する Framer Motion

### 6. ドキュメント
以下を含む包括的な README.md を作成します。
- 前提条件とセットアップ手順
- 認証と環境設定
- コネクタのセットアップとデータ ソースの構成
- ローカルの開発および展開プロセス
- 一般的な問題のトラブルシューティング

## 実装ガイドライン

### 言及する前提条件
- Power Platform Tools 拡張機能を備えた Visual Studio Code
- Node.js (LTS バージョン - v18.x または v20.x を推奨)
- Git によるバージョン管理
- Power Platform CLI (PAC CLI) - 最新バージョン
- Code Apps が有効になっている Power Platform 環境 (管理者設定が必要)
- エンド ユーザー向け Power Apps Premium ライセンス
- Azure アカウント (Azure SQL または他の Azure コネクタを使用している場合)

### 含める PAC CLI コマンド
- `pac auth create --environment {environment-id}` - 特定の環境で認証する
- `pac env select --environment {environment-url}` - ターゲット環境の選択
- `pac code init --displayName "App Name"` - コード アプリ プロジェクトを初期化します
- `pac connection list` - 利用可能な接続をリストします
- `pac code add-data-source -a {api-name} -c {connection-id}` - コネクタの追加
- `pac code push` - Power Platform へのデプロイ### 正式にサポートされているコネクタ
セットアップ例とともに、これらの公式にサポートされているコネクタに焦点を当てます。
- **SQL Server (Azure SQL を含む)**: 完全な CRUD 操作、ストアド プロシージャ
- **SharePoint**: ドキュメント ライブラリ、リスト、サイト
- **Office 365 ユーザー**: プロフィール情報、ユーザーの写真、グループ メンバーシップ
- **Office 365 グループ**: チーム情報とコラボレーション
- **Azure Data Explorer**: 分析とビッグ データ クエリ
- **OneDrive for Business**: ファイルのストレージと共有
- **Microsoft Teams**: チームのコラボレーションと通知
- **MSN Weather**: 気象データの統合
- **Microsoft Translator V2**: 多言語翻訳
- **Dataverse**: 完全な CRUD 操作、関係、ビジネス ロジック

### サンプルコネクタ統合
Office 365 ユーザー向けの実例を含めます。```タイプスクリプト
// 例: 現在のユーザー プロファイルを取得する
const profile = await Office365UsersService.MyProfile_V2("id,displayName,jobTitle,userPrincipalName");

// 例: ユーザーの写真を取得する
const photoData = await Office365UsersService.UserPhoto_V2(profile.data.id);
「」### 文書に対する現在の制限事項
- コンテンツ セキュリティ ポリシー (CSP) はまだサポートされていません
- ストレージ SAS IP 制限はサポートされていません
- Power Platform Git 統合なし
- Dataverse ソリューションのサポートなし
- ネイティブの Azure Application Insights 統合なし

### 含めるべきベストプラクティス
- ローカル開発にはポート 3000 を使用します (Power Apps SDK で必要)
- TypeScript 設定で `verbatimModuleSyntax: false` を設定します
- `base: "./"` と適切なパス エイリアスを使用して vite.config.ts を構成します
- 機密データはアプリコードではなくデータソースに保存します
- Power Platform マネージド プラットフォーム ポリシーに従う
- コネクタ操作の適切なエラー処理を実装する
- PAC CLI から生成された TypeScript モデルとサービスを使用する
- 適切な非同期初期化とエラー処理を備えた PowerProvider を含める

## 成果物

1. 必要なファイルをすべて使用してプロジェクトの足場を完成させる
2. コネクタを統合した動作サンプル アプリケーション
3. 包括的なドキュメントとセットアップ手順
4. 開発および展開スクリプト
5. Power Apps Code Apps 向けに最適化された TypeScript 構成
6. ベストプラクティスの実装例

生成されたプロジェクトが Microsoft の公式 Power Apps Code Apps ドキュメントと https://github.com/microsoft/PowerAppsCodeApps のサンプルに従っていること、および `pac code push` コマンドを使用して Power Platform に正常にデプロイできることを確認します。