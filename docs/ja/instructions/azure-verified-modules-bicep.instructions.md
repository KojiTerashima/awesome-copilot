---
description: 'Azure 検証済みモジュール (AVM) と Bicep'
applyTo: '**/*.bicep, **/*.bicepparam'
---

# Azure 検証済みモジュール (AVM) 上腕二頭筋

## 概要

Azure Verified Module (AVM) は、Azure のベストプラクティスに従って、事前に構築、テスト、検証された Bicep モジュールです。これらのモジュールを使用すると、自信を持って Azure Infrastructure as Code (IaC) を作成、更新、またはレビューできます。

## モジュールの検出

### 上腕二頭筋の公開レジストリ

- モジュールの検索: `br/public:avm/res/{service}/{resource}:{version}`
- 利用可能なモジュールを参照: `https://github.com/Azure/bicep-registry-modules/tree/main/avm/res`
- 例: `br/public:avm/res/storage/storage-account:0.30.0`

### 公式AVMインデックス

- **上腕二頭筋リソースモジュール**: `https://raw.githubusercontent.com/Azure/Azure-Verified-Modules/refs/heads/main/docs/static/module-indexes/BicepResourceModules.csv`
- **上腕二頭筋パターンモジュール**: `https://raw.githubusercontent.com/Azure/Azure-Verified-Modules/refs/heads/main/docs/static/module-indexes/BicepPatternModules.csv`

### モジュールのドキュメント

- **GitHub リポジトリ**: `https://github.com/Azure/bicep-registry-modules/tree/main/avm/res/{service}/{resource}`
- **README**: 各モジュールには例を含む包括的なドキュメントが含まれています

## モジュールの使用法

### 例から

1. `https://github.com/Azure/bicep-registry-modules/tree/main/avm/res/{service}/{resource}` でモジュールの README を確認してください
2. モジュールのドキュメントからサンプルコードをコピーします
3. `br/public:avm/res/{service}/{resource}:{version}`を使用した参照モジュール
4. 必須およびオプションのパラメータを構成する

### 使用例

```bicep
module storageAccount 'br/public:avm/res/storage/storage-account:0.30.0' = {
  name: 'storage-account-deployment'
  scope: resourceGroup()
  params: {
    name: storageAccountName
    location: location
    skuName: 'Standard_LRS'
    tags: tags
  }
}
```

### AVMモジュールが利用できない場合

リソースタイプに AVM モジュールが存在しない場合は、最新の安定した API バージョンでネイティブ Bicep リソース宣言を使用します。

## 命名規則

### モジュールリファレンス

- **リソースモジュール**: `br/public:avm/res/{service}/{resource}:{version}`
- **パターンモジュール**: `br/public:avm/ptn/{pattern}:{version}`
- 例: `br/public:avm/res/network/virtual-network:0.7.2`

### 記号名

- すべての名前 (変数、パラメーター、リソース、モジュール) には lowerCamelCase を使用してください。
- リソースタイプを説明する名前を使用します (例: `storageAccountName` ではなく `storageAccount`)
- シンボリック名では、リソースの名前ではなくリソースを表すため、「name」接尾辞を使用しないでください。
- 変数とパラメータを接尾辞で区別しないようにする

## バージョン管理

### バージョン固定のベストプラクティス

- 常に特定のモジュールバージョンに固定します: `:{version}`
- セマンティックバージョニングを使用する (例: `:0.30.0`)
- アップグレードする前にモジュールの変更ログを確認する
- 最初に非実稼働環境でバージョンのアップグレードをテストします

## 開発のベストプラクティス

### モジュールの検出と使用

- ✅ **常に** 未加工リソースを作成する前に既存の AVM モジュールを確認してください
- ✅ 実装前にモジュールのドキュメントと例を **確認**してください
- ✅ **モジュールのバージョンを明示的に固定**
- ✅ 可能な場合はモジュールの型を **使用** (モジュールから型をインポート)
- ✅ **生のリソース宣言よりも AVM モジュールを優先します**

### コード構造

- ✅ `@sys.description()` デコレータを使用してファイルの先頭でパラメータを **宣言**
- ✅ **指定** `@minLength()` および `@maxLength()` をパラメータに命名します
- ✅ **有効なデプロイメントをブロックしないように、`@allowed()` デコレータを慎重に使用してください**
- ✅ **テスト環境に安全なデフォルト値を設定**します (低コスト SKU)
- ✅ 複雑な式にはリソースプロパティに埋め込む代わりに変数を **使用**してください
- ✅ **外部構成ファイルに `loadJsonContent()` を活用**

### リソース参照

- ✅ **参照には `reference()` や `resourceId()` ではなく、シンボリック名を使用してください** (例: `storageAccount.id`)
- ✅ **明示的な `dependsOn` ではなく、シンボリック名を使用して依存関係を作成**します。
- ✅ **他のリソースからプロパティにアクセスするには、`existing` キーワードを使用します**
- ✅ **アクセス** モジュール出力はドット表記 (例: `storageAccount.outputs.resourceId`)

### リソースの名前付け

- ✅ **一意の名前には意味のある接頭辞を付けて `uniqueString()` を使用します**
- ✅ **接頭辞を追加**します。リソースによっては数字で始まる名前が許可されないためです。
- ✅ **尊重** リソース固有の名前付け制約 (長さ、文字数)

### 子リソース

- ✅ **子リソースの過度のネストを避ける**
- ✅ **手動で名前を作成する代わりに `parent` プロパティまたはネストを使用します**

### 安全

- ❌ **出力にはシークレットやキーを決して含めないでください**
- ✅ **リソースプロパティを出力で直接使用** (例: `storageAccount.outputs.primaryBlobEndpoint`)
- ✅ 可能な場合はマネージド ID を **有効**
- ✅ ネットワーク分離が有効な場合はパブリックアクセスを **無効**

### 種類

- ✅ 利用可能な場合はモジュールから **インポート** タイプ: `import { deploymentType } from './module.bicep'`
- ✅ **複雑なパラメータ構造にはユーザー定義型を使用**
- ✅ **変数の型推論を活用**

### ドキュメント

- ✅ **複雑なロジックに役立つ `//` コメントを含めます**
- ✅ **すべてのパラメータで `@sys.description()` を明確な説明とともに使用します**
- ✅ **文書化** 自明ではない設計上の決定

## 検証要件

### ビルド検証 (必須)

Bicep ファイルに変更を加えた後、次のコマンドを実行して、すべてのファイルが正常にビルドされたことを確認します。

```shell
# Ensure Bicep CLI is up to date
az bicep upgrade

# Build and validate changed Bicep files
az bicep build --file main.bicep
```

### 上腕二頭筋パラメータファイル

- ✅ `*.bicep` ファイルを変更するときは、付随する `*.bicepparam` ファイルを **常に**更新してください
- ✅ **検証** パラメータファイルが現在のパラメータ定義と一致する
- ✅ コミットする前にパラメータファイルを使用して展開をテストします**

## ツールの統合

### 利用可能なツールを使用する

- **スキーマ情報**: リソーススキーマには `azure_get_schema_for_Bicep` を使用します
- **展開ガイダンス**: `azure_get_deployment_best_practices` ツールを使用する
- **サービスドキュメント**: Azure サービス固有のガイダンスには `microsoft.docs.mcp` を使用してください

### GitHub コパイロットの統合

上腕二頭筋を使用する場合:

1. リソースを作成する前に既存の AVM モジュールを確認する
2. 公式モジュールのサンプルを出発点として使用する
3. すべての変更を行った後、`az bicep build` を実行します
4. 付随する `.bicepparam` ファイルを更新する
5. ドキュメントのカスタマイズまたは例からの逸脱

## トラブルシューティング

### よくある問題

1. **モジュールバージョン**: モジュールリファレンスでは常に正確なバージョンを指定してください
2. **依存関係の欠落**: 依存モジュールの前にリソースが作成されていることを確認してください
3. **検証の失敗**: `az bicep build` を実行して構文/型エラーを特定します
4. **パラメータファイル**: パラメータが変更されたときに `.bicepparam` ファイルが更新されるようにします

### サポートリソース

- **AVM ドキュメント**: `https://azure.github.io/Azure-Verified-Modules/`
- **上腕二頭筋レジストリ**: `https://github.com/Azure/bicep-registry-modules`
- **上腕二頭筋のドキュメント**: `https://learn.microsoft.com/azure/azure-resource-manager/bicep/`
- **ベストプラクティス**: `https://learn.microsoft.com/azure/azure-resource-manager/bicep/best-practices`

## コンプライアンスチェックリスト

Bicep コードを送信する前に:

- [ ] 利用可能な場合は AVM モジュールを使用
- [ ] モジュールのバージョンが固定されています
- [ ] コードは正常にビルドされました (`az bicep build`)
- [ ] 付随する `.bicepparam` ファイルが更新されました
- すべてのパラメータに対する [ ] `@sys.description()`
- [ ] 参照に使用される記号名
- [ ] 出力にシークレットはありません
- [ ] 必要に応じてインポート/定義されたタイプ
- [ ] 複雑なロジックのためにコメントが追加されました
- [ ] lowerCamelCase の命名規則に従います
