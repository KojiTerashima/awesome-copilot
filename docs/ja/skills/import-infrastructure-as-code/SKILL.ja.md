---
name: import-infrastructure-as-code
description: 'Azure CLIのディスカバリーとAzure Verified Modules（AVM）を使用して既存のAzureリソースをTerraformにインポートします。ライブのAzureインフラストラクチャをリバースエンジニアリングし、既存のサブスクリプション/リソースグループ/リソースIDからInfrastructure as Codeを生成し、依存関係をマッピングし、ダウンロードしたモジュールソースから正確なインポートアドレスを導出し、設定のドリフトを防止し、あらゆるAzureリソースタイプに対応したAVMベースのTerraformファイルを検証およびプランニング用に生成する必要がある場合に使用します。'
---

# インフラストラクチャをコードとしてインポート（Azure -> Terraform with AVM）

ディスカバリーデータとAzure Verified Modulesを使用して、既存のAzureインフラストラクチャを保守可能なTerraformコードに変換します。

## このスキルを使用するタイミング

ユーザーが以下を要求した場合にこのスキルを使用してください：

- 既存のAzureリソースをTerraformにインポートする
- ライブのAzure環境からIaCを生成する
- AVMがサポートする任意のAzureリソースタイプを扱う（非AVMフォールバックは正当な理由を文書化する）
- サブスクリプションまたはリソースグループからインフラを再作成する
- 発見されたAzureリソース間の依存関係をマッピングする
- 手書きの`azurerm_*`リソースの代わりにAVMモジュールを使用する

## 前提条件

- Azure CLIがインストールされ認証済みであること（`az login`）
- 対象のサブスクリプションまたはリソースグループへのアクセス権
- Terraform CLIがインストールされていること
- Terraform RegistryおよびAVMインデックスソースへのネットワークアクセス

## 入力パラメーター

| パラメーター | 必須 | デフォルト | 説明 |
|---|---|---|---|
| `subscription-id` | いいえ | アクティブなCLIコンテキスト | サブスクリプションスコープのディスカバリーとコンテキスト設定に使用するAzureサブスクリプション |
| `resource-group-name` | いいえ | なし | リソースグループスコープのディスカバリーに使用するAzureリソースグループ |
| `resource-id` | いいえ | なし | 特定リソーススコープのディスカバリーに使用する1つ以上のAzure ARMリソースID |

`subscription-id`、`resource-group-name`、または`resource-id`のいずれか1つは必須です。

## ステップバイステップのワークフロー

### 1) 必要なスコープを収集（必須）

ディスカバリーコマンドを実行する前に、以下のいずれかのスコープを要求してください：

- サブスクリプションスコープ：`<subscription-id>`
- リソースグループスコープ：`<resource-group-name>`
- 特定リソーススコープ：1つ以上の`<resource-id>`値

スコープ処理ルール：

- Azure ARMリソースID（例：`/subscriptions/.../providers/...`）はクラウドリソース識別子として扱い、ローカルファイルシステムのパスとして扱わないこと。
- リソースIDはAzure CLIの`--ids`引数（例：`az resource show --ids <resource-id>`）でのみ使用すること。
- ユーザーが明示的にローカルファイルパスであると言わない限り、`cat`、`ls`、`read_file`、グロブ検索などのファイル読み取りコマンドにリソースIDを渡さないこと。
- 既に有効なスコープが1つ提供されている場合、失敗したコマンドで必要になる場合を除き、追加のスコープ入力を求めないこと。
- 既に提供されたスコープ値から回答可能なフォローアップ質問はしないこと。

スコープが不足している場合は明示的に尋ねて処理を停止してください。

### 2) 認証とコンテキスト設定

選択されたスコープに必要なコマンドのみを実行します。

サブスクリプションスコープの場合：

```bash
az login
az account set --subscription <subscription-id>
az account show --query "{subscriptionId:id, name:name, tenantId:tenantId}" -o json
```

期待される出力：`subscriptionId`、`name`、`tenantId`を含むJSONオブジェクト。

リソースグループまたは特定リソーススコープの場合、`az login`は必須ですが、アクティブなコンテキストが正しい場合は`az account set`は任意です。

特定リソーススコープを使用する場合は、まず直接`--ids`ベースのコマンドを優先し、具体的なコマンドで必要になる場合を除き、サブスクリプションやリソースグループの追加ディスカバリーを避けてください。

### 3) ディスカバリーコマンドの実行

選択したスコープを使用してリソースをディスカバーします。正確なTerraform生成に必要なすべての情報を取得してください。

```bash
# サブスクリプションスコープ
az resource list --subscription <subscription-id> -o json

# リソースグループスコープ
az resource list --resource-group <resource-group-name> -o json

# 特定リソーススコープ
az resource show --ids <resource-id-1> <resource-id-2> ... -o json
```

期待される出力：Azureリソースのメタデータ（`id`、`type`、`name`、`location`、`tags`、`properties`）を含むJSONオブジェクトまたは配列。

### 4) コード生成前に依存関係を解決

エクスポートされたJSONを解析し、以下をマッピングします：

- 親子関係（例：NIC -> サブネット -> VNet）
- `properties`内のリソース間参照
- Terraform作成の順序

重要：以下のドキュメントを生成し、プロジェクトルートの`docs`フォルダーに保存してください。

- すべての発見されたリソースとそのメタデータ、依存関係および参照を含む`exported-resources.json`
- 発見されたリソースとその関係に基づく人間が読めるアーキテクチャ概要を記述した`EXPORTED-ARCHITECTURE.MD`

### 5) Azure Verified Modulesの選択（必須）

各リソースタイプに対して最新のAVMバージョンを使用してください。

### Terraform Registry

- 「avm」＋リソース名で検索
- 「Partner」タグで公式AVMモジュールをフィルター
- 例：「avm storage account」で検索しPartnerでフィルター

### 公式AVMインデックス

> **注意：** 以下のリンクは常にメインブランチ上のCSVファイルの最新バージョンを指します。意図的にファイルは時間とともに変わる可能性があります。特定の時点のバージョンが必要な場合は、URLに特定のリリースタグを使用してください。

- **Terraform Resource Modules**: `https://raw.githubusercontent.com/Azure/Azure-Verified-Modules/refs/heads/main/docs/static/module-indexes/TerraformResourceModules.csv`
- **Terraform Pattern Modules**: `https://raw.githubusercontent.com/Azure/Azure-Verified-Modules/refs/heads/main/docs/static/module-indexes/TerraformPatternModules.csv`
- **Terraform Utility Modules**: `https://raw.githubusercontent.com/Azure/Azure-Verified-Modules/refs/heads/main/docs/static/module-indexes/TerraformUtilityModules.csv`

### 個別モジュール情報

`.terraform`フォルダーにローカルで利用できない場合は、`web`ツールや他の適切なMCPメソッドを使用してモジュール情報を取得してください。

AVMソースを使用：

- Registry: `https://registry.terraform.io/modules/Azure/<module>/azurerm/latest`
- GitHub: `https://github.com/Azure/terraform-azurerm-avm-res-<service>-<resource>`

AVMモジュールが存在する場合は、手書きの`azurerm_*`リソースよりAVMモジュールを優先してください。

GitHubリポジトリからモジュール情報を取得する場合、通常リポジトリルートのREADME.mdファイルにモジュールの詳細情報が含まれています。例：https://raw.githubusercontent.com/Azure/terraform-azurerm-avm-res-<service>-<resource>/refs/heads/main/README.md

### 5a) コードを書く前にモジュールのREADMEを読む（必須）

**このステップは省略できません。** モジュールのHCLコードを1行も書く前に、そのモジュールのREADME全文を取得して読み込んでください。生の`azurerm`プロバイダーの知識や他のAVMモジュールの経験に頼らないでください。

選択した各AVMモジュールについてREADMEを取得：

```text
https://raw.githubusercontent.com/Azure/terraform-azurerm-avm-res-<service>-<resource>/refs/heads/main/README.md
```

または`terraform init`後にモジュールが既にダウンロードされている場合：

```bash
cat .terraform/modules/<module_key>/README.md
```

READMEからコードを書く前に抽出し記録する内容：

1. **必須入力** — モジュールが要求するすべての入力。ここにリストされている子リソース（NIC、拡張機能、サブネット、パブリックIPなど）はモジュール内で管理されます。これらのリソースの独立したモジュールブロックを作成しないでください。
2. **オプション入力** — 正確なTerraform変数名と宣言された`type`。生の`azurerm`プロバイダーの引数名やブロック構造と一致すると仮定しないでください。
3. **使用例** — 使用されているリソースグループ識別子（`parent_id` vs `resource_group_name`）、子リソースの表現方法（インラインマップ vs 別モジュール）、各入力が期待する構文を確認してください。

#### モジュールルールはパターンとして適用し、仮定しない

以下の例は、インポート失敗の原因となる不一致のタイプを示しています。すべてのAVMモジュールにこれらの名前が当てはまるとは限りません。必ず選択した各モジュールのREADMEと`variables.tf`を確認してください。

**`avm-res-compute-virtualmachine`（任意のバージョン）**

- `network_interfaces`は**必須入力**です。NICはVMモジュールが所有します。VMモジュールと並行して独立した`avm-res-network-networkinterface`モジュールを作成しないでください。すべてのNICは`network_interfaces`の下にインラインで定義します。
- TrustedLaunchはトップレベルのブール値`secure_boot_enabled = true`および`vtpm_enabled = true`で表現されます。`security_type`引数はConfidential VMディスク暗号化用に`os_disk`の下にのみ存在し、TrustedLaunchには使用しません。
- `boot_diagnostics`はオブジェクトではなく`bool`です。`boot_diagnostics = true`を使用し、ストレージURIが必要な場合は別の`boot_diagnostics_storage_account_uri`変数を使用します。
- 拡張機能はモジュール内の`extensions`マップで管理されます。独立した拡張リソースを作成しないでください。

**`avm-res-network-virtualnetwork`（任意のバージョン）**

- このモジュールは`azurerm`ではなくAzAPIプロバイダーを使用しています。リソースグループの指定には`resource_group_name`ではなく`parent_id`（完全なリソースグループリソースID文字列）を使用してください。
- READMEのすべての例は`parent_id`を示し、`resource_group_name`は示しません。

すべてのAVMモジュールに共通するポイント：

- 子リソースの所有権は**必須入力**から判断し、兄弟モジュールを作成する前に確認すること。
- 受け入れられる変数名と型は**オプション入力**と`variables.tf`から判断すること。
- 識別子のスタイルと入力形状はREADMEの使用例から判断すること。
- 生の`azurerm_*`リソースから引数名を推測しないこと。

### 6) Terraformファイルの生成

### インポートブロックを書く前にモジュールソースを検査（必須）

`terraform init`でモジュールがダウンロードされた後、各モジュールのソースファイルを検査して、`import {}`ブロックを書く前に正確なTerraformリソースアドレスを特定してください。記憶だけでインポートアドレスを書かないでください。

#### ステップA — プロバイダーとリソースラベルの特定

```bash
grep "^resource" .terraform/modules/<module_key>/main*.tf
```

これにより、モジュールが`azurerm_*`または`azapi_resource`ラベルを使用しているかがわかります。例：`avm-res-network-virtualnetwork`は`azapi_resource "vnet"`を公開し、`azurerm_virtual_network "this"`ではありません。

#### ステップB — 子モジュールとネストされたパスの特定

```bash
grep "^module" .terraform/modules/<module_key>/main*.tf
```

子リソースがサブモジュール（サブネット、拡張機能など）で管理されている場合、インポートアドレスにはすべての中間モジュールラベルを含める必要があります：

```text
module.<root_module_key>.module.<child_module_key>["<map_key>"].<resource_type>.<label>[<index>]
```

#### ステップC — `count`と`for_each`の確認

```bash
grep -n "count\|for_each" .terraform/modules/<module_key>/main*.tf
```

`count`を使用するリソースはインポートアドレスにインデックスが必要です。`count = 1`（例：条件付きLinux vs Windows選択）の場合、アドレスは`[0]`で終わる必要があります。`for_each`を使用するリソースは数値インデックスではなく文字列キーを使用します。

#### 既知のインポートアドレスパターン（学習例）

以下は例示です。テンプレートとして使用し、現在のインポートでダウンロードしたモジュールのソースコードから正確なアドレスを導出してください。

| リソース | 正しいインポート`to`アドレスパターン |
|---|---|
| AzAPI対応VNet | `module.<vnet_key>.azapi_resource.vnet` |
| サブネット（ネスト、countベース） | `module.<vnet_key>.module.subnet["<subnet_name>"].azapi_resource.subnet[0]` |
| Linux VM（countベース） | `module.<vm_key>.azurerm_linux_virtual_machine.this[0]` |
| VM NIC | `module.<vm_key>.azurerm_network_interface.virtualmachine_network_interfaces["<nic_key>"]` |
| VM拡張機能（デフォルトdeploy_sequence=5） | `module.<vm_key>.module.extension["<ext_name>"].azurerm_virtual_machine_extension.this` |
| VM拡張機能（deploy_sequence=1–4） | `module.<vm_key>.module.extension_<n>["<ext_name>"].azurerm_virtual_machine_extension.this` |
| NSG-NIC関連付け | `module.<vm_key>.azurerm_network_interface_security_group_association.this["<nic_key>-<nsg_key>"]` |

生成物：

- `providers.tf`（`azurerm`プロバイダーと必要なバージョン制約付き）
- `main.tf`（AVMモジュールブロックと明示的依存関係）
- `variables.tf`（環境固有の値用）
- `outputs.tf`（キーIDとエンドポイント用）
- `terraform.tfvars.example`（プレースホルダー値付き）

### ライブプロパティとモジュールデフォルトの差分比較（必須）

初期構成を書いた後、発見されたライブリソースのすべての非ゼロプロパティを対応するAVMモジュールの`variables.tf`に宣言されたデフォルト値と比較してください。ライブ値がモジュールデフォルトと異なるプロパティは、Terraform構成で明示的に設定する必要があります。

特に以下のプロパティカテゴリに注意してください。これらは設定のサイレントドリフトの一般的な原因です：

- **タイムアウト値**（例：パブリックIPの`idle_timeout_in_minutes`はデフォルト`4`、ライブ環境では`30`がよく使われる）
- **ネットワークポリシーフラグ**（例：サブネットの`private_endpoint_network_policies`はデフォルト`"Enabled"`、既存サブネットでは`"Disabled"`の場合が多い）
- **SKUと割り当て方法**（例：パブリックIPの`sku`、`allocation_method`）
- **可用性ゾーン**（例：VMゾーン、パブリックIPゾーン）
- **ストレージやデータベースリソースの冗長性およびレプリケーション設定**

ライブプロパティは明示的な`az`コマンドで取得してください。例：

```bash
az network public-ip show --ids <resource_id> --query "{idleTimeout:idleTimeoutInMinutes, sku:sku.name, zones:zones}" -o json
az network vnet subnet show --ids <resource_id> --query "{privateEndpointPolicies:privateEndpointNetworkPolicies, delegation:delegations}" -o json
```

`az resource list`の出力だけに依存しないでください。ネストされたプロパティや計算されたプロパティが省略される場合があります。

モジュールバージョンは明示的に固定してください：

```hcl
module "example" {
	source  = "Azure/<module>/azurerm"
	version = "<latest-compatible-version>"
}
```

### 7) 生成コードの検証

以下を実行：

```bash
terraform init
terraform fmt -recursive
terraform validate
terraform plan
```

期待される出力：構文エラーなし、検証エラーなし、発見されたインフラストラクチャの意図に合致したプラン。

## トラブルシューティング

| 問題 | 可能な原因 | 対処 |
|---|---|---|
| `az`コマンドが認証エラーで失敗 | テナント/サブスクリプション誤りまたはRBACロール不足 | `az login`を再実行し、サブスクリプションコンテキストを確認、必要な権限を確認 |
| ディスカバリー出力が空 | スコープ誤りまたはスコープ内にリソースなし | スコープ入力を再確認し、スコープ指定のlist/showコマンドを再実行 |
| リソースタイプに対応するAVMモジュールが見つからない | AVMで未対応のリソースタイプ | そのタイプにはネイティブの`azurerm_*`リソースを使用し、ギャップを文書化 |
| `terraform validate`が失敗 | 変数不足または依存関係未解決 | 必須変数と明示的依存関係を追加し、再度検証 |
| モジュールに存在しない引数や変数 | AVM変数名が`azurerm`プロバイダー引数名と異なる | モジュールREADMEの`variables.tf`やオプション入力セクションを参照し正しい名前を確認 |
| インポートブロックが失敗 — リソースがアドレスに存在しない | プロバイダーラベル誤り（`azurerm_` vs `azapi_`）、サブモジュールパス不足、`[0]`インデックス不足 | `.terraform/modules/<key>/main*.tf`で`grep "^resource"`および`grep "^module"`を実行し正確なアドレスを特定 |
| `terraform plan`でインポート済みリソースに予期しない`~ update`が表示される | ライブ値がAVMモジュールのデフォルトと異なる | `az <resource> show`でライブプロパティを取得し、モジュールデフォルトと比較、明示的に値を追加 |
| 子リソースモジュールで「provider configuration not present」エラー | 親モジュールが所有する子リソースを独立モジュールとして宣言している | READMEの必須入力を確認し、誤った独立モジュールを削除、親モジュールの入力構造で子リソースをモデル化 |
| ネストされた子リソースのインポートが「resource not found」で失敗 | 中間モジュールパス不足、マップキー誤り、インデックス不足 | ソースのモジュールブロックと`count`/`for_each`を検査し、すべてのモジュールセグメントと必要なキー/インデックスを含む完全なネストインポートアドレスを構築 |
| ツールがARMリソースIDをファイルパスとして読み取ろうとしたり、繰り返しスコープ質問をする | リソースIDを`--ids`入力として扱わず、既に提供されたスコープを信頼しなかった | ARM IDは厳密にクラウド識別子として扱い、`az ... --ids ...`を使用し、有効なスコープが1つ存在する場合は再プロンプトを停止 |

## レスポンス契約

結果を返す際は以下を提供してください：

1. 使用したスコープ（サブスクリプション、リソースグループ、またはリソースID）
2. 作成されたディスカバリーファイル
3. 検出されたリソースタイプ
4. 選択されたAVMモジュールとバージョン
5. 生成または更新されたTerraformファイル
6. 検証コマンドの結果
7. ユーザー入力が必要な未解決のギャップ（あれば）

## エージェントの実行ルール

- スコープが不足している場合は処理を続行しないこと。
- 発見されたファイルと検証出力をリストせずにインポート成功を主張しないこと。
- Terraform生成前に依存関係マッピングを省略しないこと。
- AVMモジュールを優先し、非AVMフォールバックは明確に正当化すること。
- **コードを書く前に必ずすべてのAVMモジュールのREADMEを読むこと。** 必須入力はモジュールが所有する子リソースを特定し、オプション入力は正確な変数名と型を文書化し、使用例はプロバイダー固有の慣習（`parent_id` vs `resource_group_name`）を示します。READMEを省略することはAVMベースのインポートで最も一般的なコードエラーの原因です。
- **NIC、拡張機能、パブリックIPリソースを独立リソースと仮定しないこと。** すべてのAVMモジュールで、READMEが明示的に別モジュールを要求しない限り、子リソースは親モジュール所有として扱います。兄弟モジュールを作成する前に必須入力を確認してください。
- **インポートアドレスを記憶だけで書かないこと。** `terraform init`後にダウンロードされたモジュールソースをgrepして、実際のプロバイダー（`azurerm` vs `azapi`）、リソースラベル、サブモジュールのネスト、`count` vs `for_each`の使用を確認してから`import {}`ブロックを書くこと。
- **ARMリソースIDをファイルパスとして扱わないこと。** リソースIDはAzure CLIの`--ids`引数やAPIクエリに使用し、ファイルIOツールでは使用しません。実際のワークスペースパスが提供された場合のみローカルファイルを読み取ります。
- **スコープが既に判明している場合はプロンプトを最小限にすること。** サブスクリプション、リソースグループ、または特定リソースIDが既に提供されている場合は、コマンドを直接実行し、必要なコンテキストが不足してコマンドが失敗した場合のみフォローアップを求めること。
- **`terraform plan`で破壊や不要な変更が0になるまでインポート完了を宣言しないこと。** テレメトリの`+ create`リソースは許容されます。実際のインフラストラクチャリソースに対する`~ update`や`- destroy`はすべて解決する必要があります。

## 参考資料

- [Azure Verified Modules index (Terraform)](https://github.com/Azure/Azure-Verified-Modules/tree/main/docs/static/module-indexes)
- [Terraform AVM Registry namespace](https://registry.terraform.io/namespaces/Azure)
