---
name: msstore-cli
description: 'Microsoft Store Developer CLI（msstore）は、WindowsアプリケーションをMicrosoft Storeに公開するためのツールです。Storeの資格情報設定、アプリ一覧表示、提出状況確認、提出物の公開、パッケージフライト管理、Store公開のCI/CD設定、Partner Centerとの連携時に使用します。Windows App SDK/WinUI、UWP、.NET MAUI、Flutter、Electron、React Native、PWAアプリケーションに対応しています。'
license: MIT
---

# Microsoft Store Developer CLI (msstore)

Microsoft Store Developer CLI（`msstore`）は、Microsoft Storeでのアプリケーション公開および管理のためのクロスプラットフォームコマンドラインインターフェイスです。Partner Center APIと統合し、さまざまなアプリケーションタイプの自動公開ワークフローをサポートします。

## このスキルを使うタイミング

以下の操作が必要な場合にこのスキルを使用してください：

- APIアクセス用のStore資格情報を設定する
- Storeアカウント内のアプリケーションを一覧表示する
- 提出物のステータスを確認する
- 提出物をStoreに公開する
- Store提出用にアプリケーションをパッケージ化する
- Store公開用にプロジェクトを初期化する
- パッケージフライト（ベータテスト）を管理する
- Store公開の自動化CI/CDパイプラインを設定する
- 提出物の段階的ロールアウトを管理する
- 提出物のメタデータをプログラムで更新する

## 前提条件

- Windows 10以降、macOS、またはLinux
- .NET 9 Desktop Runtime（Windows）または.NET 9 Runtime（macOS/Linux）
- 適切な権限を持つPartner Centerアカウント
- Partner Center APIアクセス用のAzure ADアプリ登録
- msstore CLIを以下のいずれかの方法でインストール済み：
  - **Microsoft Store**: [ダウンロード](https://www.microsoft.com/store/apps/9P53PC5S0PHJ)
  - **WinGet**: `winget install "Microsoft Store Developer CLI"`
  - **手動**: [GitHub Releases](https://aka.ms/msstoredevcli/releases)からダウンロード

### Partner Centerのセットアップ

msstoreを使用する前に、Partner Centerアクセス用のAzure ADアプリケーションを作成してください：

1. [Partner Center](https://partner.microsoft.com/dashboard)にアクセス
2. **アカウント設定** > **ユーザー管理** > **Azure ADアプリケーション**に移動
3. 新しいアプリケーションを作成し、**テナントID**、**クライアントID**、**クライアントシークレット**を控える
4. アプリケーションに適切な権限（マネージャーまたは開発者ロール）を付与する

## コアコマンドリファレンス

### info - 設定情報の表示

現在の資格情報設定を表示します。

```bash
msstore info
```

**オプション:**

| オプション | 説明 |
| ------ | ----------- |
| `-v, --verbose` | 詳細な出力を表示 |

### reconfigure - 資格情報の設定

Microsoft Store APIの資格情報を設定または更新します。

```bash
msstore reconfigure [options]
```

**オプション:**

| オプション | 説明 |
| ------ | ----------- |
| `-t, --tenantId` | Azure ADテナントID |
| `-s, --sellerId` | Partner CenterセラーID |
| `-c, --clientId` | Azure ADアプリケーションクライアントID |
| `-cs, --clientSecret` | 認証用クライアントシークレット |
| `-ct, --certificateThumbprint` | 証明書のサムプリント（クライアントシークレットの代替） |
| `-cfp, --certificateFilePath` | 証明書ファイルパス（クライアントシークレットの代替） |
| `-cp, --certificatePassword` | 証明書パスワード |
| `--reset` | 完全な再設定なしで資格情報をリセット |

**例:**

```bash
# クライアントシークレットで設定
msstore reconfigure --tenantId $TENANT_ID --sellerId $SELLER_ID --clientId $CLIENT_ID --clientSecret $CLIENT_SECRET

# 証明書で設定
msstore reconfigure --tenantId $TENANT_ID --sellerId $SELLER_ID --clientId $CLIENT_ID --certificateFilePath ./cert.pfx --certificatePassword MyPassword
```

### settings - CLI設定

Microsoft Store Developer CLIの設定を変更します。

```bash
msstore settings [options]
```

**オプション:**

| オプション | 説明 |
| ------ | ----------- |
| `-t, --enableTelemetry` | テレメトリを有効（true）または無効（false）にする |

#### パブリッシャー表示名の設定

```bash
msstore settings setpdn <publisherDisplayName>
```

`init`コマンドのデフォルトパブリッシャー表示名を設定します。

### apps - アプリケーション管理

アプリケーションの一覧表示や情報取得を行います。

#### アプリ一覧表示

```bash
msstore apps list
```

Partner Centerアカウント内のすべてのアプリを一覧表示します。

#### アプリ詳細取得

```bash
msstore apps get <productId>
```

**引数:**

| 引数 | 説明 |
| -------- | ----------- |
| `productId` | StoreのプロダクトID（例: 9NBLGGH4R315） |

**例:**

```bash
# 特定のアプリの詳細を取得
msstore apps get 9NBLGGH4R315
```

### submission - 提出物管理

Storeへの提出物を管理します。

| サブコマンド | 説明 |
| ----------- | ----------- |
| `status` | 提出物のステータスを取得 |
| `get` | 提出物のメタデータとパッケージ情報を取得 |
| `getListingAssets` | 提出物のリスティング資産を取得 |
| `updateMetadata` | 提出物のメタデータを更新 |
| `poll` | 提出物のステータスを完了までポーリング |
| `publish` | 提出物を公開 |
| `delete` | 提出物を削除 |

#### 提出物ステータス取得

```bash
msstore submission status <productId>
```

#### 提出物詳細取得

```bash
msstore submission get <productId>
```

#### メタデータ更新

```bash
msstore submission updateMetadata <productId> <metadata>
```

`<metadata>`は更新するメタデータのJSON文字列です。JSONにはシェルが解釈する文字（引用符、波括弧など）が含まれるため、適切に引用符で囲むかエスケープしてください：

- **Bash/Zsh**: JSONをシングルクォートで囲み、シェルにそのまま渡します。
  ```bash
  msstore submission updateMetadata 9NBLGGH4R315 '{"description":"My updated app"}'
  ```
- **PowerShell**: シングルクォートを使用（またはダブルクォート内のダブルクォートをエスケープ）。
  ```powershell
  msstore submission updateMetadata 9NBLGGH4R315 '{"description":"My updated app"}'
  ```
- **cmd.exe**: 内部のダブルクォートをバックスラッシュでエスケープ。
  ```cmd
  msstore submission updateMetadata 9NBLGGH4R315 "{\"description\":\"My updated app\"}"
  ```

> **ヒント:** 複雑または複数行のメタデータの場合は、JSONをファイルに保存し、その内容を渡すと引用符の問題を回避できます：
> ```bash
> msstore submission updateMetadata 9NBLGGH4R315 "$(cat metadata.json)"
> ```

**オプション:**

| オプション | 説明 |
| ------ | ----------- |
| `-s, --skipInitialPolling` | 初回のステータスポーリングをスキップ |

#### 提出物公開

```bash
msstore submission publish <productId>
```

#### 提出物ポーリング

```bash
msstore submission poll <productId>
```

提出物のステータスがPUBLISHEDまたはFAILEDになるまでポーリングします。

#### 提出物削除

```bash
msstore submission delete <productId>
```

**オプション:**

| オプション | 説明 |
| ------ | ----------- |
| `--no-confirm` | 確認プロンプトをスキップ |

### init - Store公開用プロジェクト初期化

Microsoft Store公開用にプロジェクトを初期化します。プロジェクトタイプを自動検出し、StoreのIDを設定します。

```bash
msstore init <pathOrUrl> [options]
```

**引数:**

| 引数 | 説明 |
| -------- | ----------- |
| `pathOrUrl` | プロジェクトディレクトリのパスまたはPWAのURL |

**オプション:**

| オプション | 説明 |
| ------ | ----------- |
| `-n, --publisherDisplayName` | パブリッシャー表示名 |
| `--package` | プロジェクトをパッケージ化も行う |
| `--publish` | パッケージ化して公開も行う（`--package`を含む） |
| `-f, --flightId` | 特定のフライトに公開 |
| `-prp, --packageRolloutPercentage` | 段階的ロールアウトの割合（0-100） |
| `-a, --arch` | 対応アーキテクチャ：x86、x64、arm64 |
| `-o, --output` | パッケージ出力ディレクトリ |
| `-ver, --version` | ビルド時のバージョン指定 |

**対応プロジェクトタイプ:**

- Windows App SDK / WinUI 3
- UWP
- .NET MAUI
- Flutter
- Electron
- React Native for Desktop
- PWA（プログレッシブウェブアプリ）

**例:**

```bash
# WinUIプロジェクトを初期化
msstore init ./my-winui-app

# PWAを初期化
msstore init https://contoso.com --output ./pwa-package

# 初期化して公開
msstore init ./my-app --publish
```

### package - Store提出用パッケージ作成

Microsoft Store提出用にアプリケーションをパッケージ化します。

```bash
msstore package <pathOrUrl> [options]
```

**引数:**

| 引数 | 説明 |
| -------- | ----------- |
| `pathOrUrl` | プロジェクトディレクトリのパスまたはPWAのURL |

**オプション:**

| オプション | 説明 |
| ------ | ----------- |
| `-o, --output` | パッケージ出力ディレクトリ |
| `-a, --arch` | 対応アーキテクチャ：x86、x64、arm64 |
| `-ver, --version` | パッケージのバージョン指定 |

**例:**

```bash
# デフォルトアーキテクチャでパッケージ化
msstore package ./my-app

# 複数アーキテクチャでパッケージ化
msstore package ./my-app --arch x64,arm64 --output ./packages

# バージョン指定でパッケージ化
msstore package ./my-app --version 1.2.3.0
```

### publish - Storeに公開

アプリケーションをMicrosoft Storeに公開します。

```bash
msstore publish <pathOrUrl> [options]
```

**引数:**

| 引数 | 説明 |
| -------- | ----------- |
| `pathOrUrl` | プロジェクトディレクトリのパスまたはPWAのURL |

**オプション:**

| オプション | 説明 |
| ------ | ----------- |
| `-i, --inputFile` | 既存の.msixまたは.msixuploadファイルのパス |
| `-id, --appId` | アプリケーションID（初期化していない場合） |
| `-nc, --noCommit` | 提出物をドラフト状態のままにする |
| `-f, --flightId` | 特定のフライトに公開 |
| `-prp, --packageRolloutPercentage` | 段階的ロールアウトの割合（0-100） |

**例:**

```bash
# プロジェクトを公開
msstore publish ./my-app

# 既存パッケージを公開
msstore publish ./my-app --inputFile ./packages/MyApp.msixupload

# ドラフトとして公開
msstore publish ./my-app --noCommit

# 段階的ロールアウトで公開
msstore publish ./my-app --packageRolloutPercentage 10
```

### flights - パッケージフライト管理

パッケージフライト（ベータテストグループ）を管理します。

| サブコマンド | 説明 |
| ----------- | ----------- |
| `list` | アプリのすべてのフライトを一覧表示 |
| `get` | フライトの詳細を取得 |
| `delete` | フライトを削除 |
| `create` | 新しいフライトを作成 |
| `submission` | フライトの提出物を管理 |

#### フライト一覧表示

```bash
msstore flights list <productId>
```

#### フライト詳細取得

```bash
msstore flights get <productId> <flightId>
```

#### フライト作成

```bash
msstore flights create <productId> <friendlyName> --group-ids <group-ids>
```

**オプション:**

| オプション | 説明 |
| ------ | ----------- |
| `-g, --group-ids` | フライトグループID（カンマ区切り） |
| `-r, --rank-higher-than` | 指定したフライトIDより上位にランク付け |

#### フライト削除

```bash
msstore flights delete <productId> <flightId>
```

#### フライト提出物管理

```bash
# フライト提出物取得
msstore flights submission get <productId> <flightId>

# フライト提出物公開
msstore flights submission publish <productId> <flightId>

# フライト提出物ステータス確認
msstore flights submission status <productId> <flightId>

# フライト提出物ポーリング
msstore flights submission poll <productId> <flightId>

# フライト提出物削除
msstore flights submission delete <productId> <flightId>
```

#### フライトロールアウト管理

```bash
# ロールアウトステータス取得
msstore flights submission rollout get <productId> <flightId>

# ロールアウト割合更新
msstore flights submission rollout update <productId> <flightId> <percentage>

# ロールアウト停止
msstore flights submission rollout halt <productId> <flightId>

# ロールアウト完了（100%）
msstore flights submission rollout finalize <productId> <flightId>
```

## よくあるワークフロー

### ワークフロー1: 初回Storeセットアップ

```bash
# 1. CLIをインストール
winget install "Microsoft Store Developer CLI"

# 2. 資格情報を設定（Partner Centerから取得）
msstore reconfigure --tenantId $TENANT_ID --sellerId $SELLER_ID --clientId $CLIENT_ID --clientSecret $CLIENT_SECRET

# 3. 設定を確認
msstore info

# 4. アプリ一覧を表示してアクセス確認
msstore apps list
```

### ワークフロー2: 新規アプリの初期化と公開

```bash
# 1. プロジェクトディレクトリに移動
cd my-winui-app

# 2. Store用に初期化（アプリIDを作成/更新）
msstore init .

# 3. アプリをパッケージ化
msstore package . --arch x64,arm64

# 4. Storeに公開
msstore publish .

# 5. 提出物のステータスを確認
msstore submission status <productId>
```

### ワークフロー3: 既存アプリの更新

```bash
# 1. アプリをビルド
dotnet publish -c Release

# 2. パッケージ化して公開
msstore publish ./my-app

# または既存パッケージから公開
msstore publish ./my-app --inputFile ./artifacts/MyApp.msixupload
```

### ワークフロー4: 段階的ロールアウト

```bash
# 1. 初期ロールアウト割合で公開
msstore publish ./my-app --packageRolloutPercentage 10

# 2. ステータスを監視しながらロールアウトを増加
msstore submission poll <productId>

# 3. （検証後）100%にロールアウト完了
# Partner Centerまたは提出物更新で完了
```

### ワークフロー5: フライトによるベータテスト

```bash
# 1. まずPartner Centerでフライトグループを作成
# 次にフライトを作成
msstore flights create <productId> "Beta Testers" --group-ids "group-id-1,group-id-2"

# 2. フライトに公開
msstore publish ./my-app --flightId <flightId>

# 3. フライト提出物のステータスを確認
msstore flights submission status <productId> <flightId>

# 4. テスト後、本番に公開
msstore publish ./my-app
```

### ワークフロー6: CI/CDパイプライン統合

```yaml
# GitHub Actionsの例
name: Publish to Store

on:
  release:
    types: [published]

jobs:
  publish:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '9.0.x'
      
      - name: Install msstore CLI
        run: winget install "Microsoft Store Developer CLI" --accept-package-agreements --accept-source-agreements
      
      - name: Configure Store credentials
        run: |
          msstore reconfigure --tenantId ${{ secrets.TENANT_ID }} --sellerId ${{ secrets.SELLER_ID }} --clientId ${{ secrets.CLIENT_ID }} --clientSecret ${{ secrets.CLIENT_SECRET }}
      
      - name: Build application
        run: dotnet publish -c Release
      
      - name: Publish to Store
        run: msstore publish ./src/MyApp
```

## winapp CLIとの統合

winapp CLI（v0.2.0以降）は`winapp store`サブコマンドを通じてmsstoreと統合しています：

```bash
# 以下のコマンドは同等です：
msstore reconfigure --tenantId xxx --clientId xxx --clientSecret xxx
winapp store reconfigure --tenantId xxx --clientId xxx --clientSecret xxx

# アプリ一覧表示
msstore apps list
winapp store apps list

# 公開
msstore publish ./my-app
winapp store publish ./my-app
```

パッケージ化と公開を統合したCLI体験を望む場合は`winapp store`を使用してください。

## トラブルシューティング

| 問題 | 解決策 |
| ----- | -------- |
| 認証失敗 | `msstore info`で資格情報を確認し、`msstore reconfigure`を再実行 |
| アプリが見つからない | プロダクトIDが正しいか確認し、`msstore apps list`で検証 |
| 権限不足 | Partner CenterのAzure ADアプリのロール（マネージャーまたは開発者）を確認 |
| パッケージ検証失敗 | Store要件を満たしているか確認し、Partner Centerの詳細を参照 |
| 提出物が停止 | `msstore submission poll <productId>`でステータスを確認 |
| フライトが見つからない | `msstore flights list <productId>`でフライトIDを確認 |
| ロールアウト割合が無効 | 値は0から100の範囲で指定 |
| PWAの初期化失敗 | URLが公開されており、有効なWebアプリマニフェストがあるか確認 |

## 環境変数

CLIは資格情報用に以下の環境変数をサポートしています：

| 変数名 | 説明 |
| -------- | ----------- |
| `MSSTORE_TENANT_ID` | Azure ADテナントID |
| `MSSTORE_SELLER_ID` | Partner CenterセラーID |
| `MSSTORE_CLIENT_ID` | Azure ADアプリケーションクライアントID |
| `MSSTORE_CLIENT_SECRET` | クライアントシークレット |

## 参考資料

- [Microsoft Store Developer CLI ドキュメント](https://learn.microsoft.com/windows/apps/publish/msstore-dev-cli/overview)
- [CLIコマンドリファレンス](https://learn.microsoft.com/windows/apps/publish/msstore-dev-cli/commands)
- [GitHubリポジトリ](https://github.com/microsoft/msstore-cli)
- [Partner Center API](https://learn.microsoft.com/windows/uwp/monetize/using-windows-store-services)
- [アプリ提出API](https://learn.microsoft.com/windows/uwp/monetize/create-and-manage-submissions-using-windows-store-services)
- [パッケージフライト概要](https://learn.microsoft.com/windows/uwp/publish/package-flights)
- [段階的パッケージロールアウト](https://learn.microsoft.com/windows/uwp/publish/gradual-package-rollout)
