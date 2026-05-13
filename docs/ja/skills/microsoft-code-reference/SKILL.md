---
name: microsoft-code-reference
description: MicrosoftのAPIリファレンスを調べ、動作するコードサンプルを見つけ、SDKコードが正しいか検証します。Azure SDK、.NETライブラリ、またはMicrosoft APIを扱う際に、適切なメソッドを探したり、パラメーターを確認したり、動作例を取得したり、エラーをトラブルシュートしたりするために使用します。公式ドキュメントを照会することで、幻覚的なメソッド、誤ったシグネチャ、非推奨のパターンを検出します。
compatibility: Microsoft Learn MCPサーバー (https://learn.microsoft.com/api/mcp) と最も相性が良いです。フォールバックとしてmslearn CLIも使用可能です。
---

# Microsoftコードリファレンス

## ツール

| 必要なもの | ツール | 例 |
|------------|--------|----|
| APIメソッド/クラスの検索 | `microsoft_docs_search` | `"BlobClient UploadAsync Azure.Storage.Blobs"` |
| 動作するコードサンプル | `microsoft_code_sample_search` | `query: "upload blob managed identity", language: "python"` |
| 完全なAPIリファレンス | `microsoft_docs_fetch` | `microsoft_docs_search`から取得したURLをフェッチ（オーバーロードや完全なシグネチャ用） |

## コードサンプルの探し方

`microsoft_code_sample_search`を使って公式の動作例を取得します：

```
microsoft_code_sample_search(query: "upload file to blob storage", language: "csharp")
microsoft_code_sample_search(query: "authenticate with managed identity", language: "python")
microsoft_code_sample_search(query: "send message service bus", language: "javascript")
```

**使用タイミング：**
- コードを書く前に、従うべき動作パターンを見つけるとき
- エラー発生後に、自分のコードと既知の良いサンプルを比較するとき
- 初期化やセットアップが不明確なときに、サンプルで完全なコンテキストを確認するとき

## APIの検索

```
# メソッドが存在するか確認（正確さのために名前空間を含める）
"BlobClient UploadAsync Azure.Storage.Blobs"
"GraphServiceClient Users Microsoft.Graph"

# クラス/インターフェースを探す
"DefaultAzureCredential class Azure.Identity"

# 正しいパッケージを探す
"Azure Blob Storage NuGet package"
"azure-storage-blob pip package"
```

メソッドに複数のオーバーロードがある場合や、完全なパラメーター詳細が必要な場合は、ページ全体を取得してください。

## エラーのトラブルシューティング

`microsoft_code_sample_search`を使って動作するコードサンプルを見つけ、自分の実装と比較します。特定のエラーには`microsoft_docs_search`と`microsoft_docs_fetch`を使用します：

| エラータイプ | クエリ例 |
|--------------|----------|
| メソッドが見つからない | `"[ClassName] methods [Namespace]"` |
| 型が見つからない | `"[TypeName] NuGet package namespace"` |
| シグネチャが間違っている | `"[ClassName] [MethodName] overloads"` → ページ全体を取得 |
| 非推奨の警告 | `"[OldType] migration v12"` |
| 認証失敗 | `"DefaultAzureCredential troubleshooting"` |
| 403 Forbidden | `"[ServiceName] RBAC permissions"` |

## 検証すべきタイミング

以下の場合は必ず検証してください：
- メソッド名が「便利すぎる」ように見えるとき（例：`UploadFile` vs 実際は`Upload`）
- SDKのバージョンを混在させているとき（v11の`CloudBlobClient` vs v12の`BlobServiceClient`）
- パッケージ名が規約に従っていないとき（.NETは`Azure.*`、Pythonは`azure-*`）
- APIを初めて使うとき

## 検証ワークフロー

Microsoft SDKを使ってコードを生成する前に、正しいか検証します：

1. **メソッドまたはパッケージが存在するか確認** — `microsoft_docs_search(query: "[ClassName] [MethodName] [Namespace]")`
2. **詳細を取得**（オーバーロードや複雑なパラメーター用） — `microsoft_docs_fetch(url: "...")`
3. **動作するサンプルを探す** — `microsoft_code_sample_search(query: "[task]", language: "[lang]")`

単純な検索ならステップ1だけで十分な場合もあります。複雑なAPI利用時は3ステップすべてを行ってください。

## CLIの代替手段

Learn MCPサーバーが利用できない場合は、ターミナルやシェル（例：Bash、PowerShell、cmd）から`mslearn` CLIを使用してください：

```sh
# 直接実行（インストール不要）
npx @microsoft/learn-cli search "BlobClient UploadAsync Azure.Storage.Blobs"

# またはグローバルインストールしてから実行
npm install -g @microsoft/learn-cli
mslearn search "BlobClient UploadAsync Azure.Storage.Blobs"
```

| MCPツール | CLIコマンド |
|----------|-------------|
| `microsoft_docs_search(query: "...")` | `mslearn search "..."` |
| `microsoft_code_sample_search(query: "...", language: "...")` | `mslearn code-search "..." --language ...` |
| `microsoft_docs_fetch(url: "...")` | `mslearn fetch "..."` |

`search`や`code-search`に`--json`を付けると、生のJSON出力を取得してさらに処理できます。
