---
name: winapp-cli
description: 'Windows App Development CLI (winapp) for building, packaging, and deploying Windows applications. Use when asked to initialize Windows app projects, create MSIX packages, generate AppxManifest.xml, manage development certificates, add package identity for debugging, sign packages, publish to the Microsoft Store, create external catalogs, or access Windows SDK build tools. Supports .NET (csproj), C++, Electron, Rust, Tauri, and cross-platform frameworks targeting Windows.'
---
# Windows アプリ開発 CLI

Windows アプリ開発 CLI (`winapp`) は、Windows SDK、MSIX パッケージの管理、アプリ ID、マニフェスト、証明書の生成、アプリ フレームワークでのビルド ツールの使用を行うためのコマンド ライン インターフェイスです。これは、クロスプラットフォーム開発と Windows ネイティブ機能の間のギャップを埋めます。

## このスキルを使用する場合

このスキルは、次の場合に使用します。

- SDK セットアップ、マニフェスト、証明書を使用して Windows アプリ プロジェクトを初期化する
- アプリケーション ディレクトリから MSIX パッケージを作成する
- AppxManifest.xml ファイルの生成または管理
- 署名用の開発証明書を作成してインストールする
- Windows APIをデバッグするためのパッケージIDを追加
- MSIX パッケージまたは実行可能ファイルに署名する
- 任意のフレームワークから Windows SDK ビルド ツールにアクセス
- クロスプラットフォーム フレームワーク (Electron、Rust、Tauri、Qt) を使用して Windows アプリを構築する
- Windows アプリ展開用の CI/CD パイプラインをセットアップする
- パッケージ ID を必要とする Windows API へのアクセス (通知、Windows AI、シェル統合)
- `winapp store` 経由でアプリを Microsoft Store に公開します
- 資産管理用の外部カタログを作成する
- NuGet 経由で Windows App SDK を使用して .NET (csproj) プロジェクトをセットアップする

## 前提条件

- Windows 10以降
- winapp CLI は次のいずれかの方法でインストールされます。
  - **WinGet**: `winget install Microsoft.WinAppCli --source winget`
  - **NPM** (電子用): `npm install @microsoft/winappcli --save-dev`
  - **GitHub Actions/Azure DevOps**: [setup-WinAppCli](https://github.com/microsoft/setup-WinAppCli) アクションを使用します。
  - **マニュアル**: [GitHub Releases](https://github.com/microsoft/WinAppCli/releases/latest) からダウンロードします。

## コア機能

### 1. プロジェクトの初期化 (`winapp init`)

最新の Windows アプリを構築するために必要なアセット (マニフェスト、証明書、ライブラリ) を含むディレクトリを初期化します。サポートされる SDK インストール モード: `stable`、`preview`、`experimental`、または `none`。

### 2. MSIX パッケージ化 (`winapp pack`)

オプションの署名、証明書の生成、自己完結型の展開バンドルを使用して、準備されたディレクトリから MSIX パッケージを作成します。

### 3. デバッグ用のパッケージ ID (`winapp create-debug-identity`)

完全なパッケージ化を行わずに ID (通知、Windows AI、シェル統合) を必要とする Windows API をデバッグするために、一時的なパッケージ ID を実行可能ファイルに追加します。

### 4. マニフェスト管理 (`winapp manifest`)AppxManifest.xml ファイルを生成し、ソース画像から画像アセットを更新して、必要なサイズとアスペクト比をすべて自動的に作成します。柔軟なアプリ ID 定義のために、AppxManifest で動的コンテンツと修飾名のためのマニフェスト プレースホルダーをサポートします。

### 5. 証明書管理 (`winapp cert`)

開発証明書を生成し、パッケージに署名するためにローカル マシン ストアにインストールします。

### 6. パッケージの署名 (`winapp sign`)

オプションのタイムスタンプ サーバー サポートを使用して、MSIX パッケージと実行可能ファイルに PFX 証明書で署名します。

### 7. SDK ビルド ツールへのアクセス (`winapp tool`)

適切に構成されたパスを使用して、任意のフレームワークまたはビルド システムから Windows SDK ビルド ツールを実行します。

### 8. Microsoft ストアの統合 (`winapp store`)

winapp から Microsoft Store Developer CLI コマンドを直接実行し、CLI を離れることなくストアの送信、パッケージの検証、ワークフローの公開を可能にします。

### 9. 外部カタログ作成 (`winapp create-external-catalog`)

外部カタログを作成して開発者の資産管理を合理化し、カタログ データをメイン パッケージから分離します。

## 使用例

### 例 1: Windows アプリの初期化とパッケージ化```bash
# Initialize workspace with defaults
winapp init
# Note: init no longer auto-generates a certificate (v0.2.0+). Generate one explicitly:
winapp cert generate

# Build your application (framework-specific)
# ...

# Create signed MSIX package
winapp pack ./build-output --generate-cert --output MyApp.msix
```### 例 2: パッケージ ID を使用したデバッグ```bash
# Add debug identity to executable for testing Windows APIs
winapp create-debug-identity ./bin/MyApp.exe

# Run your app - it now has package identity
./bin/MyApp.exe
```### 例 3: CI/CD パイプラインのセットアップ```yaml
# GitHub Actions example
- name: Setup winapp CLI
  uses: microsoft/setup-WinAppCli@v1

- name: Initialize and Package
  run: |
    winapp init --no-prompt
    winapp pack ./build-output --output MyApp.msix
```### 例 4: Electron アプリの統合```bash
# Install via npm
npm install @microsoft/winappcli --save-dev

# Initialize and add debug identity for Electron
npx winapp init
npx winapp node add-electron-debug-identity

# Package for distribution
npx winapp pack ./out --output MyElectronApp.msix
```## ガイドライン

1. **最初に `winapp init` を実行します** - SDK セットアップとマニフェストが構成されていることを確認するために、他のコマンドを使用する前に必ずプロジェクトを初期化してください。注: v0.2.0 以降、`winapp init` は開発証明書を自動的に生成しなくなりました。開発証明書で署名する必要がある場合は、`winapp cert generate` を明示的に実行します。
2. **マニフェストの変更後に `create-debug-identity` を再実行します** - AppxManifest.xml が変更されるたびに、パッケージ ID を再作成する必要があります。
3. **CI/CD には `--no-prompt` を使用** - デフォルト値を使用することで、自動パイプラインでの対話型プロンプトを防止します。
4. **共有プロジェクトには `winapp restore` を使用します** - `winapp.yaml` で定義された正確な環境状態をマシン間で再作成します。
5. **単一の画像からアセットを生成** - 1 つのロゴを持つ `winapp manifest update-assets` を使用して、必要なアイコン サイズをすべて生成します。

## 一般的なパターン

### パターン: 新しいプロジェクトを初期化する```bash
cd my-project
winapp init
# Creates: AppxManifest.xml, SDK configuration, winapp.yaml
# Note: .NET (csproj) projects skip winapp.yaml and configure NuGet packages in the .csproj directly

# Generate a dev signing certificate explicitly (no longer done by init)
winapp cert generate
```### パターン: 既存の証明書を含むパッケージ```bash
winapp pack ./build-output --cert ./mycert.pfx --cert-password secret --output MyApp.msix
```### パターン: 自己完結型の展開```bash
# Bundle Windows App SDK runtime with the package
winapp pack ./my-app --self-contained --generate-cert
```### パターン: パッケージのバージョンを更新する```bash
# Update to latest stable SDKs
winapp update

# Or update to preview SDKs
winapp update --setup-sdks preview
```## 制限事項

- Windows 10以降が必要（WindowsのみのCLI）
- パッケージ ID のデバッグでは、マニフェストの変更後に `create-debug-identity` を再実行する必要があります
- 自己完結型の展開では、Windows App SDK ランタイムをバンドルすることでパッケージ サイズが増加します
- 開発証明書はテスト専用です。本番環境には信頼できる証明書が必要です
- 一部の Windows API では、マニフェストで特定の機能を宣言する必要があります。
- `winapp init` は証明書を自動生成しなくなりました (v0.2.0 以降)。 `winapp cert generate` を明示的に実行する
- .NET (csproj) プロジェクトは `winapp.yaml` をスキップします。 SDKパッケージはプロジェクトファイル内で直接設定されます
- winapp CLI はパッケージに NuGet グローバル キャッシュを使用します (`%userprofile%/.winapp/packages` ではありません)。
- winapp CLI はパブリック プレビュー段階にあり、変更される可能性があります

## パッケージ ID によって有効になる Windows API

パッケージ ID により、強力な Windows API へのアクセスが可能になります。

| API カテゴリ |例 |
| ------------ | -------- |
| **お知らせ** |インタラクティブなネイティブ通知、通知管理 |
| **Windows AI** |オンデバイス LLM、テキスト/画像 AI API (Phi Silica、Windows ML) |
| **シェルの統合** |エクスプローラー、タスクバー、共有シートの統合 |
| **プロトコル ハンドラー** |カスタム URI スキーム (`yourapp://`) |
| **デバイスアクセス** |カメラ、マイク、位置情報 (同意あり) |
| **バックグラウンド タスク** |アプリが閉じたときに実行 |
| **ファイルの関連付け** |アプリでファイルの種類を開く |

## トラブルシューティング

|問題 |ソリューション |
| ----- | -------- |
|証明書が信頼されていません | `winapp cert install <cert-path>` を実行してローカル マシン ストアにインストールします。
|パッケージ ID が機能しない |マニフェストを変更した後は `winapp create-debug-identity` を実行します。
| SDK が見つかりません | `winapp restore` または `winapp update` を実行して、SDK がインストールされていることを確認します。
|署名が失敗する |証明書のパスワードを検証し、証明書の有効期限が切れていないことを確認します。

## 参考文献

- [GitHub リポジトリ](https://github.com/microsoft/WinAppCli)
- [完全な CLI ドキュメント](https://github.com/microsoft/WinAppCli/blob/main/docs/usage.md)
- [.NET プロジェクト ガイド](https://github.com/microsoft/WinAppCli/blob/main/docs/guides/dotnet.md)
- [サンプル アプリケーション](https://github.com/microsoft/WinAppCli/tree/main/samples)
- [Windows アプリ SDK](https://learn.microsoft.com/windows/apps/windows-app-sdk/)
- [MSIX パッケージングの概要](https://learn.microsoft.com/windows/msix/overview)
- [パッケージ ID の概要](https://learn.microsoft.com/windows/apps/desktop/modernize/package-identity-overview)