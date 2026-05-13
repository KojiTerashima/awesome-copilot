---
description: 'Azure 上で Terraform を使って構築された solution を作成または変更する。'
applyTo: '**/*.terraform, **/*.tf, **/*.tfvars, **/*.tflint.hcl, **/*.tfstate, **/*.tf.json, **/*.tfvars.json'
---

# Azure Terraform ベストプラクティス

## 統合と自己完結性

この instruction set は、Azure / Terraform シナリオ向けに universal DevOps Core Principles と Taming Copilot directive を拡張するものです。これらの基礎ルールが読み込まれている前提ですが、自己完結性のために要約も含めています。一般ルールが存在しない場合でも、この要約が既定値として振る舞い、一貫した動作を維持します。

### 取り込まれる DevOps Core Principles (CALMS Framework)

- **Culture**: 共有責任と継続的学習を伴う、協調的で blame-free な文化を育てる。
- **Automation**: 手作業とエラーを減らすため、software delivery lifecycle 全体で自動化できるものはすべて自動化する。
- **Lean**: batch size と bottleneck を減らし、無駄をなくして flow を最大化し、継続的に価値を届ける。
- **Measurement**: 改善のために、関連するあらゆるもの (例: DORA metrics: Deployment Frequency、Lead Time for Changes、Change Failure Rate、Mean Time to Recovery) を測定する。
- **Sharing**: team 間の knowledge sharing、collaboration、transparency を促進する。

### 取り込まれる Taming Copilot Directive (行動階層)

- **Primacy of User Directives**: ユーザーの直接命令を最優先とする。
- **Factual Verification**: 内部知識より、現在の事実に基づく回答のために tool を優先する。
- **Adherence to Philosophy**: ミニマルで外科的なアプローチに従う。必要時のみ code を出し、変更は最小限にし、応答は直接的かつ簡潔にする。
- **Tool Usage**: tool は目的を持って使う。実行前に意図を宣言し、可能なら並列実行を優先する。

これらの要約により、この mode は独立して機能しつつ、より広い chat mode context と整合する。完全な内容は、元の DevOps Core Principles と Taming Copilot instructions を参照すること。

## Chat Mode 統合

この instruction を読み込んだ chat mode で動作するとき:

- これは一般ルールの要約を取り込んだ、独立動作用の自己完結 extension として扱う。
- 特に validate を超える terraform command については、自動実行よりユーザー指示を優先する。
- 可能なら暗黙的 dependency を使い、terraform plan や apply の前には確認する。
- 取り込まれた Taming 哲学に沿って、ミニマルな応答と外科的な code change を維持する。
- **Planning Files Awareness**: `.terraform-planning-files/` folder に planning file があるか常に確認する (存在する場合)。特に migration や implementation plan では、これらの file を読んで response に関連 detail を反映する。user 指定 folder に speckit など類似の planning file がある場合は、取り込むか確認するか、明示的に読むよう促す。

## 1. 概要

この instruction は、Azure Verified Module の取り込み方と使い方を含め、Terraform で作成する solution に対する Azure 固有のガイダンスを提供する。

一般的な Terraform 規約については [terraform.instructions.md](terraform.instructions.md) を参照する。

module 開発、特に Azure Verified Module については [azure-verified-modules-terraform.instructions.md](azure-verified-modules-terraform.instructions.md) を参照する。

## 2. 避けるべきアンチパターン

**設定:**

- パラメーター化すべき値をハードコードしてはならない
- `terraform import` を通常の workflow パターンとして使うべきではない
- code を理解しにくくする複雑な条件ロジックは避けるべきである
- `local-exec` provisioner は絶対に必要な場合を除いて使ってはならない

**セキュリティ:**

- Terraform file や state に secret を保存してはならない
- 過度に寛容な IAM role や network rule は避ける
- 便宜のために security feature を無効化してはならない
- default password や key を使ってはならない

**運用:**

- テストなしで Terraform 変更を直接 production へ適用してはならない
- Terraform 管理下の resource に手動変更を加えるべきではない
- Terraform state file の破損や不整合を無視してはならない
- production 向け Terraform を local machine から実行してはならない
- Terraform state file (`**/*.tfstate`) は読み取り専用操作にのみ使い、変更はすべて Terraform CLI または HCL 経由で行う
- `**/.terraform/**` (取得済み module / provider) の内容は読み取り専用操作にのみ使う

これらは、組み込まれた Taming Copilot directive の secure / operational practice を補強するものである。

---

## 3. code を整理して構成する

Terraform 設定は、論理的に file を分けて構成する:

- resource には `main.tf` を使う
- input には `variables.tf` を使う
- output には `outputs.tf` を使う
- provider 設定には `terraform.tf` を使う
- 複雑な式の抽象化と可読性向上のため、`locals.tf` を使う
- 一貫した命名規則と formatting (`terraform fmt`) に従う
- main.tf や variables.tf が大きくなりすぎたら、resource 種別や機能単位で複数 file に分割する (例: `main.networking.tf`、`main.storage.tf`。対応する変数は `variables.networking.tf` などへ移す)

変数と module 名には `snake_casing` を使う。

## 4. Azure Verified Modules (AVM) を使う

重要な resource には、利用可能なら AVM を使うべきである。AVM は Well Architected Framework に整合するよう設計され、Microsoft が support / maintain しているため、保守すべき code 量を減らせる。これらを見つける方法は [Azure Verified Modules for Terraform](azure-verified-modules-terraform.instructions.md) にある。

対象 resource に Azure Verified Module が存在しない場合は、既存 work に整合し、upstream として community に貢献する機会を持てるよう、AVM の "style に沿った" module を作ることを提案する。

ただし、user が internal private registry を使うよう指示されている場合、または Azure Verified Module を使いたくないと明示した場合は、この指示の例外とする。

これは、事前検証済みで community が保守する module を活用することで、組み込まれた DevOps Automation principle とも整合する。

## 5. 変数と code style の標準

solution code では AVM に沿った coding standard を使い、一貫性を維持する:

- **変数命名**: すべての変数名に snake_case を使う (TFNFR4、TFNFR16)。説明的で一貫した命名にする。
- **変数定義**: すべての変数に明示的な type 宣言 (TFNFR18) と十分な description (TFNFR17) を付ける。特別な必要がない限り、collection 値に nullable default (TFNFR20) は避ける。
- **機密変数**: sensitive 変数は適切に扱い、`sensitive = false` を明示的に設定しない (TFNFR22)。sensitive default value は正しく扱う (TFNFR23)。
- **dynamic block**: 必要に応じて optional な nested object には dynamic block を使い (TFNFR12)、default value には `coalesce` や `try` function を活用する (TFNFR13)。
- **code 構成**: local value 用に `locals.tf` を使うことを検討し (TFNFR31)、locals には正確な typing を確保する (TFNFR33)。

## 6. Secret

最良の secret は、保存しなくてよい secret である。例えば password や key ではなく Managed Identity を使う。

サポートされる場合は、Terraform v1.11+ の write-only parameter を伴う `ephemeral` secret を使い、state file に secret が保存されないようにする。利用可否は module documentation を確認する。

secret が必要な場合は、別 service を使うよう指示されていなければ Key Vault に保存する。

secret を local filesystem に書き込んだり git に commit したりしてはならない。

sensitive value は適切に扱い、他属性から分離し、絶対に必要な場合を除いて sensitive data を output しない。TFNFR19、TFNFR22、TFNFR23 に従う。

## 7. Output

- **不要な output は避ける**。他の configuration に必要な情報を公開する場合にのみ使う
- secret を含む output には `sensitive = true` を使う
- すべての output に明確な description を付ける

```hcl
output "resource_group_name" {
  description = "作成された resource group の名前"
  value       = azurerm_resource_group.example.name
}

output "virtual_network_id" {
  description = "virtual network の ID"
  value       = azurerm_virtual_network.example.id
}
```

## 8. Local Values の使い方

- 計算値や複雑な式には locals を使う
- 繰り返し使う式を抽出して可読性を上げる
- 関連値は構造化された locals にまとめる

```hcl
locals {
  common_tags = {
    Environment = var.environment
    Project     = var.project_name
    Owner       = var.owner
    CreatedBy   = "terraform"
  }

  resource_name_prefix = "${var.project_name}-${var.environment}"
  location_short       = substr(var.location, 0, 3)
}
```

## 9. 推奨される Terraform practice に従う

- **冗長な depends_on の検出**: 同じ resource block 内で依存先 resource がすでに暗黙参照されている場合は `depends_on` を探して削除する。`depends_on` は明示的に必要な場合にのみ残す。module output に依存してはならない。

- **反復**: 0-1 resource には `count`、複数 resource には `for_each` を使う。stable な resource address のため map を優先する。TFNFR7 に合わせる。

- **data source**: root module では許容されるが、再利用可能な module では避ける。data source lookup より明示的な module parameter を優先する。

- **パラメーター化**: 明示的な `type` 宣言 (TFNFR18)、十分な description (TFNFR17)、nullable でない default (TFNFR20) を持つ strongly typed variable を使う。AVM が公開する変数を活用する。

- **バージョン管理**: Terraform と Azure provider は最新安定版を対象にする。version は code で明示し、更新を保つ (TFFR3)。

## 10. Folder 構成

Terraform 設定には一貫した folder 構成を使う。

環境差分の変更には tfvars を使う。一般に、非本番環境ではコスト最適化をしつつ、環境間はなるべく似た状態を保つ。

アンチパターン - environment ごとの branch、environment ごとの repository、environment ごとの folder、または同様の layout。こうした構成は environment 間で root folder の logic をテストしにくくする。

Terragrunt のような tool がこの設計に影響する可能性があることを意識する。

**推奨** 構成の一例:

```text
my-azure-app/
├── infra/                          # Terraform root module (AZD compatible)
│   ├── main.tf                     # Core resource
│   ├── variables.tf                # Input variable
│   ├── outputs.tf                  # Output
│   ├── terraform.tf                # Provider 設定
│   ├── locals.tf                   # Local value
│   └── environments/               # Environment 固有設定
│       ├── dev.tfvars              # Development 環境
│       ├── test.tfvars             # Test 環境
│       └── prod.tfvars             # Production 環境
├── .github/workflows/              # CI/CD pipeline (GitHub を使う場合)
├── .azdo/                          # CI/CD pipeline (Azure DevOps を使う場合の推奨)
└── README.md                       # Documentation
```

ユーザーの直接同意なしに folder 構成を変更してはならない。

一貫した file 命名と構成のため、AVM specification TFNFR1、TFNFR2、TFNFR3、TFNFR4 に従う。

## Azure 固有のベストプラクティス

### Resource の命名とタグ付け

- [Azure naming conventions](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/resource-naming) に従う
- multi-region deployment には一貫した region 命名と variable を使う
- 一貫した tagging を実装する

### Resource Group 戦略

- 指定されている場合は既存 resource group を使う
- 新しい resource group は必要な場合にのみ、確認を取ったうえで作成する
- 用途と環境がわかる説明的な名前を使う

### Networking に関する考慮事項

- 新しい network resource を作る前に、既存 VNet / subnet ID を検証する (例: この solution は既存 hub & spoke landing zone にデプロイされるのか)
- NSG と ASG を適切に使う
- 必要なら PaaS service に private endpoint を実装し、そうでなければ resource firewall 制限で public access を制限する。public endpoint が必要な例外は comment で説明する。

### セキュリティとコンプライアンス

- service principal ではなく Managed Identity を使う
- 適切な RBAC を伴う Key Vault を実装する
- audit trail のため diagnostic setting を有効にする
- 最小権限の原則に従う

## コスト管理

- 高価な resource には budget approval を確認する
- 環境に応じた sizing を使う (dev と prod)
- 指定がなければ cost constraint を確認する

## State 管理

- state locking 付きの remote backend (Azure Storage) を使う
- state file を source control に commit してはならない
- 保存時 / 転送時の暗号化を有効にする

## 検証

- 既存 resource の inventory を行い、未使用 resource block の削除を提案する
- 構文確認には `terraform validate` を実行する
- `terraform plan` 実行前には確認する。Terraform plan には subscription ID が必要であり、これは provider block に書くのではなく ARM_SUBSCRIPTION_ID environment variable から取得すべきである。
- まず非本番環境で設定をテストする
- 冪等性を確保する (複数回 apply しても同じ結果になる)

## Fallback Behavior

一般ルールが読み込まれていない場合の既定は次のとおり: ミニマルな code 生成、validate を超える terraform command には明示的な同意、そしてすべての提案で CALMS principle に従うこと。
