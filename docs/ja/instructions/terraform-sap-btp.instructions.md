---
description: 'SAP Business Technology Platform (SAP BTP) 向けの Terraform 規約とガイドライン。'
applyTo: '**/*.tf, **/*.tfvars, **/*.tflint.hcl, **/*.tf.json, **/*.tfvars.json'
---

# SAP BTP 上の Terraform – ベストプラクティスと規約

## 中核原則

Terraform code は、最小限で、モジュール化され、再実行可能で、安全かつ監査可能に保つ。
Terraform HCL は常にバージョン管理し、生成された state は決してバージョン管理しない。

## セキュリティ

必須事項:
- 最新安定版の Terraform CLI と provider version を使い、security patch のため能動的に upgrade する。
- secret、credential、certificate、Terraform state、plan output artifact を commit してはならない。
- すべての secret variable と output は `sensitive = true` にする。
- 可能なら ephemeral / write-only の provider auth (Terraform >= 1.11) を優先し、state に secret が残らないようにする。
- sensitive output は最小限にし、下流 automation が本当に必要なものだけを出力する。
- CI では `tfsec`、`trivy`、`checkov` のいずれか少なくとも 1 つで継続的に scan する。
- provider credential は定期的に見直し、key を rotate し、対応していれば MFA を有効にする。

## モジュール化

明確さと速度を重視して構成する:
- environment ではなく論理 domain (例: entitlement、service instance) 単位で分割する。
- module は再利用可能な複数 resource pattern にのみ使い、単一 resource の wrapper module は避ける。
- module 階層は浅く保ち、深いネストや循環依存は避ける。
- module 間の共有 data は本当に必要なものだけ `outputs` で公開する (必要なら sensitive を付ける)。

## 保守性

implicit より explicit を目指す。
- WHAT ではなく WHY を comment し、明白な resource attribute を言い換えるだけの comment は避ける。
- ハードコードではなく parameterize (variable 化) する。default は妥当な場合にだけ与える。
- 外部の既存 infra には data source を優先するが、同じ root で直前に作成した resource には使わず output を使う。
- 汎用の再利用 module では data source を避け、代わりに input を要求する。
- 未使用または遅い data source は削除する。plan 時間を悪化させるためである。
- 派生値や繰り返し式は `locals` にまとめてロジックを集中させる。

## スタイルと formatting

### 一般
- resource、variable、output には説明的で一貫した名前を付ける。
- variable と locals には snake_case を使う。
- インデントは 2 スペース。`terraform fmt -recursive` を実行する。

### レイアウトと file

推奨構成:
```text
my-sap-btp-app/
├── infra/                      # Root module
│   ├── main.tf                 # Core resource (大きい場合は domain ごとに分割)
│   ├── variables.tf            # Input
│   ├── outputs.tf              # Output
│   ├── provider.tf             # Provider 設定
│   ├── locals.tf               # Local / 派生値
│   └── environments/           # Environment 変数 file のみ
│       ├── dev.tfvars
│       ├── test.tfvars
│       └── prod.tfvars
├── .github/workflows/          # CI/CD (GitHub の場合)
└── README.md                   # Documentation
```

ルール:
- environment ごとに separate branch / repo / folder を作ってはならない (アンチパターン)。
- environment 間の drift は最小限に抑え、差分は `*.tfvars` file だけで表現する。
- 巨大化した `main.tf` / `variables.tf` は、論理名を持つ断片 file (例: `main_services.tf`、`variables_services.tf`) に分割する。
  命名は一貫させる。

### Resource block の構成

順序 (上 → 下): 任意の `depends_on`、次に `count` / `for_each`、次に attribute、最後に `lifecycle`。
- Terraform が依存関係を推論できない場合にのみ `depends_on` を使う (例: data source が entitlement を必要とする場合)。
- 任意の単一 resource には `count`、map を key にした複数 instance には `for_each` を使い、stable な address を保つ。
- attribute は、必須を先、任意を後に並べ、論理 section の間は空行で区切る。
- 各 section 内は走査しやすいようアルファベット順にする。

### 変数
- すべての variable に明示的な `type` と空でない `description` を付ける。
- `any` より具体的な型 (`object`、`map(string)` など) を優先する。
- collection に null default は避け、代わりに空 list / map を使う。

### Locals
- 計算値や繰り返し式は集中管理する。
- 関連値は object locals にまとめて凝集性を高める。

### Output
- 下流 module / automation が使うものだけを公開する。
- secret は `sensitive = true` にする。
- 常に明確な `description` を付ける。

### Formatting と linting
- `terraform fmt -recursive` を実行する (CI では必須)。
- pre-commit / CI では `tflint` (必要に応じて `terraform validate` も) を強制する。

## ドキュメント

必須事項:
- すべての variable と output に `description` + `type` を付ける。
- ルートの `README.md` は簡潔にする: purpose、prerequisite、auth model、usage (init / plan / apply)、testing、rollback を記載する。
- module documentation は `terraform-docs` で生成する (可能なら CI に組み込む)。
- comment は、自明でない判断や制約を明確にする場合にだけ書く。

## State 管理
- locking をサポートする remote backend を使う (例: Terraform Cloud、AWS S3、GCS、Azure Storage)。SAP BTP Object Store は reliable な locking と security 機能が不足するため避ける。
- `*.tfstate` や backup は絶対に commit しない。
- state は保存時 / 転送時に暗号化し、アクセスは最小権限で制限する。

## 検証
- commit 前に `terraform validate` を実行する (syntax と内部整合性の確認)。
- `terraform plan` の前には user に確認する (auth と global account subdomain が必要)。auth は env var または tfvars で渡し、provider block に secret を inline してはならない。
- まず non-prod でテストし、apply が冪等になることを確認する。

## テスト
- module logic と invariant には Terraform test framework (`*.tftest.hcl`) を使う。
- success path と failure path の両方をカバーし、test は stateless / idempotent に保つ。
- 可能なら外部 data source は mock を優先する。

## SAP BTP Provider 固有事項

ガイドライン:
- service plan ID は `data "btp_subaccount_service_plan"` で解決し、その data source の `serviceplan_id` を参照する。

例:
```terraform
data "btp_subaccount_service_plan" "example" {
  subaccount_id = var.subaccount_id
  service_name  = "your_service_name"
  plan_name     = "your_plan_name"
}

resource "btp_subaccount_service_instance" "example" {
  subaccount_id  = var.subaccount_id
  serviceplan_id = data.btp_subaccount_service_plan.example.id
  name           = "my-example-instance"
}
```

明示的 dependency (provider が推論できない場合):
```terraform
resource "btp_subaccount_entitlement" "example" {
  subaccount_id = var.subaccount_id
  service_name  = "your_service_name"
  plan_name     = "your_plan_name"
}

data "btp_subaccount_service_plan" "example" {
  subaccount_id = var.subaccount_id
  service_name  = "your_service_name"
  plan_name     = "your_plan_name"
  depends_on    = [btp_subaccount_entitlement.example]
}
```

subscription も entitlement に依存する。provider が attribute 経由の関連 (`service_name` / `plan_name` ↔ `app_name`) を推論できない場合は `depends_on` を追加する。

## Tool 統合

### HashiCorp Terraform MCP Server
対話的な schema lookup、resource block の下書き作成、validation には Terraform MCP Server を使う。
1. server を install / run する (https://github.com/mcp/hashicorp/terraform-mcp-server を参照)。
2. Copilot / MCP client configuration で tool として追加する。
3. authoring 前に provider schema を問い合わせる (例: resource、data source 一覧)。
4. resource block の draft を生成し、その後で naming / tagging standard に合わせて手動で調整する。
5. plan summary を validate する (secret は含めない)。`apply` 前に reviewer と diff を確認する。

### Terraform Registry
SAP BTP provider documentation: https://registry.terraform.io/providers/SAP/btp/latest/docs を authoritative source として参照する。確信が持てない場合は、MCP response と registry docs を相互確認する。

## アンチパターン (避ける)

設定:
- environment 固有値のハードコード (variable と tfvars を使う)。
- `terraform import` の常用 (migration 時のみ)。
- 明確さを損なう深すぎる / 不透明な条件ロジックや dynamic block。
- 避けられない integration gap を除く `local-exec` provisioner。
- 明示的な正当化なく、同じ root で SAP BTP provider と Cloud Foundry provider を混在させること (module を分ける)。

セキュリティ:
- HCL、state、VCS に secret を保存すること。
- 速度のために暗号化、validation、scan を無効化すること。
- default password / key の使用、または environment 間で credential を使い回すこと。

運用:
- non-prod での検証なしに production へ直接 apply すること。
- Terraform 外で manual drift change を行うこと。
- state の不整合や破損兆候を無視すること。
- 管理されていない local laptop から production apply を実行すること (CI/CD または承認済み runner を使う)。
- `*.tfstate` の生データから業務 data を読むこと。代わりに output / data source を使う。

すべての変更は Terraform CLI + HCL を通して行うこと。state を手動で変更してはならない。
