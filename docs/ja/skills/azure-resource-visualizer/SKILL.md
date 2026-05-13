---
name: azure-resource-visualizer
description: Azure リソース グループを分析し、個々のリソース間の関係を示す詳細な Mermaid アーキテクチャ図を生成します。ユーザーが Azure リソースの図を求めている場合や、リソース同士の関係性の理解を支援してほしい場合にこのスキルを使用してください。
license: LICENSE.txt の完全な利用規約
metadata:
  author: Tom Meschter (tom.meschter@microsoft.com)
---

# Azure Resource Visualizer - アーキテクチャ図ジェネレーター

ユーザーは、個々のリソースがどのように連携しているかを理解したい、またはその関係を示す図を作成したいと依頼することがあります。あなたのミッションは、Azure リソース グループを調査し、その構造と関係を理解し、アーキテクチャを明確に表現する包括的な Mermaid 図を生成することです。

## 中核となる責務

1. **リソース グループの検出**: 指定がない場合は利用可能なリソース グループを一覧表示する
2. **詳細なリソース分析**: すべてのリソース、その構成、および相互依存関係を調査する
3. **関係マッピング**: リソース間のすべての接続を特定して文書化する
4. **図の生成**: 詳細かつ正確な Mermaid 図を作成する
5. **ドキュメント作成**: 図を埋め込んだ明確な markdown ファイルを作成する

## ワークフロープロセス

### ステップ 1: リソース グループの選択

ユーザーがリソース グループを指定していない場合:

1. ツールを使って利用可能なリソース グループを照会します。このためのツールがない場合は `az` を使用します。
2. リソース グループを場所付きで番号付きリストとして提示します
3. ユーザーに番号または名前で 1 つ選択してもらいます
4. ユーザーの応答を待ってから先へ進みます

リソース グループが指定されている場合は、存在確認を行って進みます。

### ステップ 2: リソース検出と分析

リソース グループが確定したら:

1. Azure MCP ツールまたは `az` を使って、リソース グループ内の**すべてのリソースを照会**します。
2. **各リソース**を分析し、以下を取得します:
   - リソース名と種類
   - SKU/ティア情報
   - ロケーション/リージョン
   - 主要な構成プロパティ
   - ネットワーク設定（VNet、サブネット、プライベート エンドポイント）
   - ID とアクセス（Managed Identity、RBAC）
   - 依存関係と接続

3. 以下を特定して**関係をマッピング**します:
   - **ネットワーク接続**: VNet ピアリング、サブネット割り当て、NSG ルール、プライベート エンドポイント
   - **データフロー**: Apps → Databases、Functions → Storage、API Management → Backends
   - **ID**: リソースに接続するマネージド ID
   - **構成**: Key Vault を参照する App Settings、接続文字列
   - **依存関係**: 親子関係、必須リソース

### ステップ 3: 図の構築

`graph TB`（上から下）または `graph LR`（左から右）形式で、**詳細な Mermaid 図**を作成します。

**図の構造ガイドライン:**

```mermaid
graph TB
    %% Use subgraphs to group related resources
    subgraph "Resource Group: [name]"
        subgraph "Network Layer"
            VNET[Virtual Network<br/>10.0.0.0/16]
            SUBNET1[Subnet: web<br/>10.0.1.0/24]
            SUBNET2[Subnet: data<br/>10.0.2.0/24]
            NSG[Network Security Group]
        end
        
        subgraph "Compute Layer"
            APP[App Service<br/>Plan: P1v2]
            FUNC[Function App<br/>Runtime: .NET 8]
        end
        
        subgraph "Data Layer"
            SQL[Azure SQL Database<br/>DTU: S1]
            STORAGE[Storage Account<br/>Type: Standard LRS]
        end
        
        subgraph "Security & Identity"
            KV[Key Vault]
            MI[Managed Identity]
        end
    end
    
    %% Define relationships with descriptive labels
    APP -->|"HTTPS requests"| FUNC
    FUNC -->|"SQL connection"| SQL
    FUNC -->|"Blob/Queue access"| STORAGE
    APP -->|"Uses identity"| MI
    MI -->|"Access secrets"| KV
    VNET --> SUBNET1
    VNET --> SUBNET2
    SUBNET1 --> APP
    SUBNET2 --> SQL
    NSG -->|"Rules applied to"| SUBNET1
```

**図に関する主要要件:**

- **レイヤーまたは目的ごとにグループ化**: Network、Compute、Data、Security、Monitoring
- **詳細を含める**: ノードラベルに SKU、ティア、重要設定を含める（改行には `<br/>` を使用）
- **すべての接続にラベルを付ける**: リソース間を流れるもの（データ、ID、ネットワーク）を説明する
- **意味のあるノード ID を使う**: 理解しやすい略語（APP、FUNC、SQL、KV）
- **視覚的な階層**: 論理的なグループ化には subgraph を使用
- **接続タイプ**:
  - データフローまたは依存関係には `-->`
  - 任意/条件付き接続には `-.->`
  - 重要/主要経路には `==>`

**リソース種別の例:**
- App Service: プラン ティア（B1、S1、P1v2）を含める
- Functions: ランタイム（.NET、Python、Node）を含める
- Databases: ティア（Basic、Standard、Premium）を含める
- Storage: 冗長性（LRS、GRS、ZRS）を含める
- VNets: アドレス空間を含める
- Subnets: アドレス範囲を含める

### ステップ 4: ファイル作成

[template-architecture.md](./assets/template-architecture.md) をテンプレートとして使い、`[resource-group-name]-architecture.md` という名前の markdown ファイルを作成し、以下を含めます:

1. **ヘッダー**: リソース グループ名、サブスクリプション、リージョン
2. **要約**: アーキテクチャの簡潔な概要（2〜3 段落）
3. **リソース一覧**: すべてのリソースを種類と主要プロパティ付きで示す表
4. **アーキテクチャ図**: 完全な Mermaid 図
5. **関係の詳細**: 主要な接続とデータフローの説明
6. **注記**: 重要な観察事項、潜在的な問題、または推奨事項

## 運用ガイドライン

### 品質基準

- **正確性**: 図に含める前にすべてのリソース詳細を検証する
- **網羅性**: リソースを省略しない。リソース グループ内のすべてを含める
- **明瞭性**: 明確で説明的なラベルと論理的なグループ化を使う
- **詳細度**: アーキテクチャ理解に重要な構成詳細を含める
- **関係性**: 明白なものだけでなく、重要な接続をすべて示す

### ツール使用パターン

1. **Azure MCP Search**: 
   - リソース グループ検出には `intent="list resource groups"` を使用
   - グループ名を指定して全リソース取得には `intent="list resources in group"` を使用
   - 個別リソース分析には `intent="get resource details"` を使用
   - 特定の Azure 操作が必要な場合は `command` パラメーターを使用

2. **ファイル作成**:
   - 常にワークスペース ルート、または存在する場合は `docs/` フォルダーに作成
   - 明確で説明的なファイル名を使用: `[rg-name]-architecture.md`
   - Mermaid 構文が有効であることを確認（出力前に頭の中で構文確認）

3. **ターミナル（必要時）**:
   - MCP では利用できない複雑なクエリには Azure CLI を使用
   - 例: `az resource list --resource-group <name> --output json`
   - 例: `az network vnet show --resource-group <name> --name <vnet-name>`

### 制約と境界

**必ず行うこと:**
- ✅ リソース グループが未指定なら一覧表示する
- ✅ ユーザーの選択を待ってから進む
- ✅ グループ内の**すべて**のリソースを分析する
- ✅ 詳細で正確な図を作成する
- ✅ ノードラベルに構成詳細を含める
- ✅ subgraph を使ってリソースを論理的にグループ化する
- ✅ すべての接続に説明的ラベルを付ける
- ✅ 図を含む完全な markdown ファイルを作成する

**絶対にしないこと:**
- ❌ 重要でなさそうだからといってリソースを省略する
- ❌ 検証なしでリソース関係を推測する
- ❌ 不完全またはプレースホルダーの図を作る
- ❌ アーキテクチャに影響する構成詳細を省く
- ❌ リソース グループ選択の確認なしに進める
- ❌ 無効な Mermaid 構文を生成する
- ❌ Azure リソースを変更・削除する（読み取り専用分析）

### エッジケースとエラーハンドリング

- **リソースが見つからない**: ユーザーに伝え、リソース グループ名を確認する
- **権限の問題**: 何が不足しているかを説明し、RBAC の確認を提案する
- **複雑なアーキテクチャ（50+ リソース）**: レイヤーごとに複数図へ分割することを検討する
- **リソース グループをまたぐ依存関係**: 図の注記に外部依存を記載する
- **関係が明確でないリソース**: 「Other Resources」セクションにまとめる

## 出力形式仕様

### Mermaid 図の構文
- 縦レイアウトには `graph TB`（上から下）を使用
- 横レイアウトには `graph LR`（左から右）を使用（横に広いアーキテクチャ向け）
- Subgraph 構文: `subgraph "Descriptive Name"`
- ノード構文: `ID["Display Name<br/>Details"]`
- 接続構文: `SOURCE -->|"Label"| TARGET`

### Markdown 構造
- メインタイトルには H1 を使用
- 主要セクションには H2 を使用
- サブセクションには H3 を使用
- リソース一覧には表を使用
- 注記と推奨事項には箇条書きを使用
- 図には `mermaid` 言語タグ付きコードブロックを使用

## 例: インタラクション

**User**: "Analyze my production resource group"

**Agent**:
1. サブスクリプション内のすべてのリソース グループを一覧表示
2. ユーザーに選択を依頼: "Which resource group? 1) rg-prod-app, 2) rg-dev-app, 3) rg-shared"
3. ユーザーが選択: "1"
4. rg-prod-app 内のすべてのリソースを照会
5. 分析: App Service、Function App、SQL Database、Storage Account、Key Vault、VNet、NSG
6. 関係を特定: App → Function、Function → SQL、Function → Storage、All → Key Vault
7. subgraph 付きの詳細な Mermaid 図を作成
8. 完全なドキュメントとして `rg-prod-app-architecture.md` を生成
9. 表示: "Created architecture diagram in rg-prod-app-architecture.md. Found 7 resources with 8 key relationships."

## 成功基準

成功する分析には以下が含まれます:
- ✅ 有効なリソース グループが特定されている
- ✅ すべてのリソースが検出・分析されている
- ✅ 重要な関係がすべてマッピングされている
- ✅ 適切にグループ化された詳細な Mermaid 図がある
- ✅ 完全な markdown ファイルが作成されている
- ✅ 明確で実行可能なドキュメントになっている
- ✅ 正しくレンダリングされる有効な Mermaid 構文である
- ✅ プロフェッショナルでアーキテクト水準の出力である

あなたの目標は、優れた可視化によって複雑なリソース関係を理解しやすくし、Azure アーキテクチャに明確さと洞察をもたらすことです。

<system_reminder>
<sql_tables>現在テーブルは存在しません。SQL ツールを初めて使用すると、デフォルト テーブル（todos、todo_deps）が自動作成されます。</sql_tables>
</system_reminder>

