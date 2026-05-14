# 図の規則 — 脅威モデルとアーキテクチャのためのマーメイド図

このファイルには、脅威モデル レポートでマーメイド ダイアグラムを作成するためのすべてのルールが含まれています。自己完結型であり、正しい図を作成するために必要なものがすべてここにあります。

---

## ⛔ 重要なルール — 図を描く前にお読みください

これらのルールは最も頻繁に違反されています。最初にそれらを読み、図を作成するたびに再確認してください。

### ルール 1: Kubernetes サイドカーのコロケーション (必須)

ターゲット システムが Kubernetes 上で実行されている場合、**ポッドを共有するコンテナは一緒に表す必要があります**。決して独立したスタンドアロン コンポーネントとして表す必要はありません。

**これを実行します — プライマリ コンテナのラベルに注釈を付けます:**```
InferencingFlow(("Inferencing Flow<br/>+ MISE, Dapr")):::process
IngestionFlow(("Ingestion Flow<br/>+ MISE, Dapr")):::process
VectorDbApi(("VectorDB API<br/>+ Dapr")):::process
```**これは行わないでください — スタンドアロンのサイドカー ノードは決して作成しないでください:**```
❌ MISE(("MISE Sidecar")):::process
❌ DaprSidecar(("Dapr Sidecar")):::process
❌ InferencingFlow -->|"localhost"| MISE
```**理由:** サイドカー (Dapr、MISE/認証プロキシ、Envoy、Istio プロキシ、ログ コレクター) は、ポッドのネットワーク名前空間、ライフサイクル、セキュリティ コンテキストをプライマリ コンテナと共有します。これらは独立したサービスではありません。

**このルールは、すべての図のタイプ (** アーキテクチャ、脅威モデル、概要) に適用されます。

### ルール 2: ポッド内フローなし (必須)

**プライマリ コンテナとそのサイドカーの間にデータ フローを描画しないでください。** これらは、コロケーション アノテーションから暗黙的に示されます。```
❌ InferencingFlow -->|"localhost:3500"| DaprSidecar
❌ InferencingFlow -->|"localhost:8080"| MISE
```ポッド内通信はローカルホスト上で行われます。ローカルホストにはセキュリティ境界がないため、図には表示されません。

### ルール 3: 境界を越えたサイドカー フローはホスト コンテナから発信される

サイドカーが信頼境界を越える呼び出しを行う場合 (MISE → Azure AD、Dapr → Redis など)、**ホスト コンテナー ノードから** 矢印を描画します。決してスタンドアロン サイドカー ノードからではありません。```
✅ InferencingFlow -->|"HTTPS (MISE auth)"| AzureAD
✅ IngestionAPI -->|"HTTPS (MISE auth)"| AzureAD
✅ InferencingFlow -->|"TCP (Dapr)"| Redis

❌ MISESidecar -->|"HTTPS"| AzureAD
❌ DaprSidecar -->|"TCP"| Redis
```複数のポッドに同じ外部ターゲットを呼び出す同じサイドカーがある場合は、ホスト コンテナごとに 1 つの矢印を描画します。同じターゲットへの複数の矢は正しいです。

### ルール 4: 要素テーブル - 個別のサイドカー行は禁止

サイドカーに個別の要素テーブル行を追加しないでください。ホスト コンテナの説明列にそれらを説明します。```
✅ | Inferencing Flow | Process | API service + MISE auth proxy + Dapr sidecar | Backend Services |
❌ | MISE Sidecar     | Process | Auth proxy for Inferencing Flow              | Backend Services |
```サイドカー クラスに独自の脅威サーフェス (MISE 認証バイパスなど) がある場合、STRIDE 分析で `## Component` セクションが取得されますが、それでも別個のダイアグラム ノードではありません。

---

## レンダリング前チェックリスト (最終処理の前に確認)

図を描いたら、次のことを確認してください。

- [ ] **すべての K8s サービス ノードにサイドカーの注釈が付けられていますか?** - 各ポッドのプロセス ノードには、同じ場所にあるすべてのコンテナの `<br/>+ SidecarName` が含まれます
- [ ] **スタンドアロン サイドカー ノードがゼロですか?** — `MISE`、`Dapr`、`Envoy`、`Istio`、`Sidecar` という名前のノードのダイアグラムを検索します。これらは個別のノードとして存在してはなりません
- [ ] **ポッド内ローカルホスト フローがゼロですか?** — ローカルホスト上のコンテナとそのサイドカーの間に矢印がありません
- [ ] **ホストからの境界を越えたサイドカー フロー?** — 外部ターゲット (Azure AD、Redis など) へのすべての矢印はホスト コンテナー ノードから発生します
- [ ] **背景は強制的に白になりますか?** — `%%{init}%%` ブロックには `'background': '#ffffff'` が含まれます
- [ ] **すべての classDef には `color:#000000`?** が含まれます — すべての要素に黒いテキストが表示されます
- [ ] **`linkStyle default` ありますか?** — `stroke:#666666,stroke-width:2px`
- [ ] **すべてのラベルは引用されていますか?** — `["Name"]`、`(("Name"))`、`-->|"Label"|`
- [ ] **サブグラフ/末尾のペアが一致しましたか?** — すべての `subgraph` には終わりの `end` があります
- [ ] **信頼境界スタイルが適用されますか?** — `stroke:#e31a1c,stroke-width:3px,stroke-dasharray: 5 5`

---

## カラーパレット

> **⛔ 重要: これらの正確な 16 進コードのみを使用してください。色を発明しないでください。Chakra UI カラー (#4299E1、#48BB78、#E53E3E)、Tailwind カラー、またはその他のパレットを使用してください。以下の色は、色盲のアクセシビリティのための ColorBrewer 定性パレットからのものです。このファイルから classDef 行をそのままコピーします。**

これらの色はすべての人魚図で共有されます。カラーは ColorBrewer の定性パレットから取得されており、色盲のアクセシビリティのために設計されています。

|色の役割 |塗りつぶし |脳卒中 |用途 |
|-----------|------|----------|----------|
|ブルー | `#6baed6` | `#2171b5` |サービス/プロセス |
|琥珀 | `#fdae61` | `#d94701` |外部インタラクター |
|緑 | `#74c476` | `#238b45` |データストア |
|赤 |該当なし | `#e31a1c` |信頼境界 (脅威モデルのみ) |
|ダークグレー |該当なし | `#666666` |矢印/リンク |
|テキスト |すべて: `color:#000000` | |すべての要素に黒いテキスト |

### 設計理論的根拠|要素 |塗りつぶし |脳卒中 |テキスト |なぜ |
|----------|------|----------|------|-----|
|プロセス | `#6baed6` | `#2171b5` | `#000000` |ミディアムブルー — 両方のテーマで表示 |
|外部インタラクター | `#fdae61` | `#d94701` | `#000000` |温かみのある琥珀色 - 青/緑とは異なります |
|データストア | `#74c476` | `#238b45` | `#000000` |ミディアムグリーン - 保管に自然 |
|信頼境界 |なし | `#e31a1c` |該当なし |赤の破線 - 視認性を高めるために 3 ピクセル |
|矢印/リンク |該当なし | `#666666` |該当なし |白い背景に濃い灰色 | 写真 白い背景に濃い灰色
|背景 | `#ffffff` |該当なし |該当なし |ダークテーマの安全性のために白を強制 |

---

## 強制的に白い背景 (必須)

すべての人魚図 (フローチャートとシーケンス) には、背景を強制的に白にする `%%{init}%%` ブロックを含める必要があります。これにより、図がダークテーマで正しくレンダリングされるようになります。

> **⛔ 重要: `primaryColor`、`secondaryColor`、`tertiaryColor`、またはカスタム カラー キーをテーマ変数に追加しないでください。 init ブロックは背景と線の色のみを制御します。すべての要素の色は classDef 行から取得されます。決して、themeVariables から取得されません。色のオーバーライドをテーマ変数に追加すると、classDef パレットが壊れます。**

### フローチャートの初期化ブロック

すべての `.mmd` ファイルの **最初の行** として追加するか、````mermaid ` flowchart:

```%%{init: {'theme': 'base', 'themeVariables': { 'background': '#ffffff', 'primaryColor': '#ffffff', 'lineColor': '#666666' }}}%%```

**THE ABOVE IS THE ONLY ALLOWED INIT BLOCK FOR FLOWCHARTS.** Do not modify it. Do not add keys. Copy it verbatim.

### Arrow / Link Default Styling

Add after classDef lines:

```linkStyle のデフォルトのストローク:#666666、ストローク幅:2px```

### Sequence Diagram Init Block

Sequence diagrams cannot use `classDef`. Use this init block:

```%%{init: {'テーマ': 'ベース', 'テーマ変数': {
  '背景': '#ffffff',
  'actorBkg': '#6baed6'、'actorBorder': '#2171b5'、'actorTextColor': '#000000'、
  'signalColor': '#666666', 'signalTextColor': '#666666',
  'noteBkgColor': '#fdae61'、'noteBorderColor': '#d94701'、'noteTextColor': '#000000'、
  'activationBkgColor': '#ddeeff', 'activationBorderColor': '#2171b5',
  'sequenceNumberColor': '#767676',
  'labelBoxBkgColor': '#f0f0f0'、'labelBoxBorderColor': '#666666'、'labelTextColor': '#000000'、
  'loopTextColor': '#000000'
}}}%%```

---

## Diagram Type: Threat Model (DFD)

Used in: `1-threatmodel.md`, `1.1-threatmodel.mmd`, `1.2-threatmodel-summary.mmd`

### `.mmd` File Format — CRITICAL

The `.mmd` file contains **raw Mermaid source only** — no markdown, no code fences. The file must start on line 1 with:
```%%{init: {'theme': 'base', 'themeVariables': { 'background': '#ffffff', 'primaryColor': '#ffffff', 'lineColor': '#666666' }}}%%```
Followed by `flowchart LR` on line 2. NEVER use `flowchart TB`.

**WRONG**: File starts with ` ```平文 ` or ````mermaid ` — these are code fences and corrupt the `.mmd` file.

### ClassDef & Shapes

```classDef プロセスの塗りつぶし:#6baed6、ストローク:#2171b5、ストローク幅:2px、色:#000000
classDef 外部塗りつぶし:#fdae61、ストローク:#d94701、ストローク幅:2px、色:#000000
classDef データストアの塗りつぶし:#74c476、ストローク:#238b45、ストローク幅:2px、カラー:#000000```

| Element Type | Shape Syntax | Example |
|-------------|-------------|---------|
| Process | `(("Name"))` circle | `WebApi(("Web API")):::process` |
| External Interactor | `["Name"]` rectangle | `User["User/Browser"]:::external` |
| Data Store | `[("Name")]` cylinder | `Database[("PostgreSQL")]:::datastore` |

### Trust Boundary Styling

```サブグラフ BoundaryId["表示名"]
    内部の %% 要素
終わり
スタイル BoundaryId 塗りつぶし:なし、ストローク:#e31a1c、ストローク幅:3px、ストローク-ダッシュ配列: 5 5```

### Flow Labels

```単方向: A -->|"ラベル"| B
双方向: <-->|"ラベル"| B```

### Data Flow IDs

- Detailed flows: `DF01`, `DF02`, `DF03`...
- Summary flows: `SDF01`, `SDF02`, `SDF03`...

### Complete DFD Template

```人魚
%%{init: {'theme': 'base', 'themeVariables': { 'background': '#ffffff', 'primaryColor': '#ffffff', 'lineColor': '#666666' }}}%%
フローチャート LR
    classDef プロセスの塗りつぶし:#6baed6、ストローク:#2171b5、ストローク幅:2px、色:#000000
    classDef 外部塗りつぶし:#fdae61、ストローク:#d94701、ストローク幅:2px、色:#000000
    classDef データストアの塗りつぶし:#74c476、ストローク:#238b45、ストローク幅:2px、カラー:#000000
    linkStyle のデフォルトのストローク:#666666、ストローク幅:2px

    ユーザー["ユーザー/ブラウザ"]:::外部

    サブグラフ Internal["内部ネットワーク"]
        WebApi(("Web API")):::プロセス
        データベース[("PostgreSQL")]:::データストア
    終わり

    ユーザー <-->|"HTTPS"|ウェブAPI
    WebApi <-->|"SQL/TLS"|データベース

    スタイル 内部塗りつぶし:なし、ストローク:#e31a1c、ストローク幅:3px、ストローク-ダッシュ配列: 5 5```

### Kubernetes DFD Template (With Sidecars)

```人魚
%%{init: {'theme': 'base', 'themeVariables': { 'background': '#ffffff', 'primaryColor': '#ffffff', 'lineColor': '#666666' }}}%%
フローチャート LR
    classDef プロセスの塗りつぶし:#6baed6、ストローク:#2171b5、ストローク幅:2px、色:#000000
    classDef 外部塗りつぶし:#fdae61、ストローク:#d94701、ストローク幅:2px、色:#000000
    classDef データストアの塗りつぶし:#74c476、ストローク:#238b45、ストローク幅:2px、カラー:#000000
    linkStyle のデフォルトのストローク:#666666、ストローク幅:2px

    ユーザー["ユーザー/ブラウザ"]:::外部
    IdP["アイデンティティプロバイダ"]:::外部

    サブグラフ K8s["Kubernetes クラスター"]
        サブグラフ Backend["バックエンド サービス"]
            ApiService(("API サービス<br/>+ AuthProxy, Dapr")):::プロセス
            Worker(("ワーカー<br/>+ Dapr")):::プロセス
        終わり
        Redis[("Redis")]:::データストア
        データベース[("PostgreSQL")]:::データストア
    終わり

    ユーザー -->|"HTTPS"| APIサービス
    APIService -->|"HTTPS"|ユーザー
    APIService -->|"HTTPS"| IdP
    APIService -->|"SQL/TLS"|データベース
    APIService -->|"Dapr HTTP"|労働者
    APIService -->|"TCP"|レディス
    ワーカー -->|"SQL/TLS"|データベース

    スタイル K8s 塗りつぶし:なし、ストローク:#e31a1c、ストローク幅:3px、ストローク-ダッシュ配列: 5 5
    スタイル バックエンド塗りつぶし:なし、ストローク:#e31a1c、ストローク幅:3px、ストローク-ダシャーレイ: 5 5```

**Key points:**
- AuthProxy and Dapr are annotated on the host node (`+ AuthProxy, Dapr`), not as separate nodes
- `ApiService -->|"HTTPS"| IdP` = auth proxy's cross-boundary call, drawn from host container
- `ApiService -->|"TCP"| Redis` = Dapr's cross-boundary call, drawn from host container
- No intra-pod flows drawn

---

## Diagram Type: Architecture

Used in: `0.1-architecture.md` only

### ClassDef & Shapes

```classDef サービスの塗りつぶし:#6baed6、ストローク:#2171b5、ストローク幅:2px、色:#000000
classDef 外部塗りつぶし:#fdae61、ストローク:#d94701、ストローク幅:2px、色:#000000
classDef データストアの塗りつぶし:#74c476、ストローク:#238b45、ストローク幅:2px、カラー:#000000```

| Element Type | Shape Syntax | Notes |
|-------------|-------------|-------|
| Services/Processes | `["Name"]` or `(["Name"])` | Rounded rectangles or stadium |
| External Actors | `(["Name"])` with `external` class | Amber distinguishes them |
| Data Stores | `[("Name")]` cylinder | Same as DFD |
| **DO NOT** use circles `(("Name"))` | | Reserved for DFD threat model diagrams |

### Layer Grouping Styling (NOT trust boundaries)

```スタイル LayerId fill:#f0f4ff、ストローク:#2171b5、ストローク幅:2px、ストローク-dasharray: 5 5```

Layer colors:
- Backend: `fill:#f0f4ff,stroke:#2171b5` (light blue)
- Data: `fill:#f0fff0,stroke:#238b45` (light green)
- External: `fill:#fff8f0,stroke:#d94701` (light amber)
- Infrastructure: `fill:#f5f5f5,stroke:#666666` (light gray)

### Flow Conventions

- Label with **what is communicated**: `"User queries"`, `"Auth tokens"`, `"Log data"`
- Protocol can be parenthetical: `"Queries (gRPC)"`
- Simpler arrows than DFD — use `-->` without requiring bidirectional flows

### Kubernetes Pods in Architecture Diagrams

Show pods with their full container composition:
```inf["推論フロー<br/>+ MISE + Dapr"]:::service
ing["取り込みフロー<br/>+ MISE + Dapr"]:::service```

### Key Difference from DFD

The architecture diagram shows **what the system does** (logical components and interactions). The threat model DFD shows **what could be attacked** (trust boundaries, data flows with protocols, element types). They share many components but serve different purposes.

### Complete Architecture Diagram Template

```人魚
%%{init: {'theme': 'base', 'themeVariables': { 'background': '#ffffff', 'primaryColor': '#ffffff', 'lineColor': '#666666' }}}%%
フローチャート LR
    classDef サービスの塗りつぶし:#6baed6、ストローク:#2171b5、ストローク幅:2px、色:#000000
    classDef 外部塗りつぶし:#fdae61、ストローク:#d94701、ストローク幅:2px、色:#000000
    classDef データストアの塗りつぶし:#74c476、ストローク:#238b45、ストローク幅:2px、カラー:#000000
    linkStyle のデフォルトのストローク:#666666、ストローク幅:2px

    ユーザー(["ユーザー"]):::外部

    サブグラフ Backend["バックエンド サービス"]
        API["API サービス"]:::サービス
        ワーカー["ワーカー"]:::サービス
    終わり

    サブグラフ Data["データ層"]
        Db[("データベース")]:::データストア
        キャッシュ[("キャッシュ")]:::データストア
    終わり

    ユーザー -->|"HTTPS"|アピ
    API --> ワーカー
    ワーカー --> DB
    API --> キャッシュ

    スタイル バックエンドの塗りつぶし:#f0f4ff、ストローク:#2171b5、ストローク幅:2px、ストローク-dasharray: 5 5
    スタイル データ塗りつぶし:#f0fff0、ストローク:#238b45、ストローク幅:2px、ストローク-ダッシュ配列: 5 5```

### Kubernetes Architecture Template

```人魚
%%{init: {'theme': 'base', 'themeVariables': { 'background': '#ffffff', 'primaryColor': '#ffffff', 'lineColor': '#666666' }}}%%
フローチャート LR
    classDef サービスの塗りつぶし:#6baed6、ストローク:#2171b5、ストローク幅:2px、色:#000000
    classDef 外部塗りつぶし:#fdae61、ストローク:#d94701、ストローク幅:2px、色:#000000
    classDef データストアの塗りつぶし:#74c476、ストローク:#238b45、ストローク幅:2px、カラー:#000000
    linkStyle のデフォルトのストローク:#666666、ストローク幅:2px

    ユーザー(["ユーザー"]):::外部
    IdP(["Azure AD"]):::外部

    サブグラフ K8s["Kubernetes クラスター"]
        Inf["推論フロー<br/>+ MISE + Dapr"]:::service
        Ing["取り込みフロー<br/>+ MISE + Dapr"]:::service
        Redis[("Redis")]:::データストア
    終わり

    ユーザー -->|"HTTPS"|インフ
    Inf -->|"認証 (MISE)"| IdP
    イング -->|"認証 (MISE)"| IdP
    Inf -->|"状態 (Dapr)"|レディス

    スタイル K8s 塗りつぶし:#f0f4ff、ストローク:#2171b5、ストローク幅:2px、ストローク-ダッシュ配列: 5 5```

---

## Sequence Diagram Rules

Used in: `0.1-architecture.md` top scenarios

- The **first 3 scenarios MUST** each include a Mermaid `sequenceDiagram`
- Scenarios 4-5 may optionally include one
- Use the **Sequence Diagram Init Block** above at the top of each
- Use `participant` aliases matching the Key Components table
- Show activations (`activate`/`deactivate`) for request-response patterns
- Include `Note` blocks for security-relevant steps (e.g., "Validates JWT token")
- Keep diagrams focused — core workflow, not every error path

### Complete Sequence Diagram Example

```人魚
%%{init: {'テーマ': 'ベース', 'テーマ変数': {
  '背景': '#ffffff',
  'actorBkg': '#6baed6'、'actorBorder': '#2171b5'、'actorTextColor': '#000000'、
  'signalColor': '#666666', 'signalTextColor': '#666666',
  'noteBkgColor': '#fdae61'、'noteBorderColor': '#d94701'、'noteTextColor': '#000000'、
  'activationBkgColor': '#ddeeff'、'activationBorderColor': '#2171b5'
}}}%%
シーケンス図
    アクターユーザー
    API サービスとしての参加者 API
    データベースとしての参加者データベース

    ユーザー >> API: POST /resource
    APIをアクティブ化する
    API に関するメモ: JWT トークンを検証する
    API->>Db: INSERT クエリ
    Db-->>Api: 結果
    API-->>ユーザー: 201 が作成されました
    APIを無効化する```

---

## Summary Diagram Rules

Used in: `1.2-threatmodel-summary.mmd` (generated only when detailed diagram has >15 elements or >4 trust boundaries)

1. **All trust boundaries must be preserved** — never combine or omit
2. **Only combine components that are NOT**: entry points, core flow components, security-critical services, primary data stores
3. **Candidates for aggregation**: supporting infrastructure, secondary caches, multiple externals at same trust level
4. **Combined element labels must list contents:**
   ```DataLayer[("データ レイヤー<br/>(UserDB、OrderDB、Redis)")]
   SupportServices(("サポート<br/>(ロギング、モニタリング)"))```
5. Use `SDF` prefix for summary data flows: `SDF01`, `SDF02`, ...
6. Include mapping table in `1-threatmodel.md`:
   ```|概要要素 |含まれています |フローの概要 |詳細なフローへのマッピング |```

---

## Naming Conventions

| Item | Convention | Example |
|------|-----------|---------|
| Element ID | PascalCase, no spaces | `WebApi`, `UserDb` |
| Display Name | Human readable in quotes | `"Web API"`, `"User Database"` |
| Flow Label | Protocol or action in quotes | `"HTTPS"`, `"SQL"`, `"gRPC"` |
| Flow ID | Unique short identifier | `DF01`, `DF02` |
| Boundary ID | PascalCase | `InternalNetwork`, `PublicDMZ` |

**CRITICAL: Always quote ALL text in Mermaid diagrams:**
- Element labels: `["Name"]`, `(("Name"))`, `[("Name")]`
- Flow labels: `-->|"Label"|`
- Subgraph titles: `subgraph ID["Title"]`

---

## Quick Reference - Shapes

```外部インタラクター: ["名前"] → 長方形
処理：((「名前」))→丸(二重括弧)
データ ストア: [("名前")] → シリンダー```

## Quick Reference - Flows

```単方向: A -->|"ラベル"| B
双方向: <-->|"ラベル"| B```

## Quick Reference - Boundaries

```サブグラフ BoundaryId["表示名"]
    内部の %% 要素
終わり
スタイル BoundaryId 塗りつぶし:なし、ストローク:#e31a1c、ストローク幅:3px、ストローク-ダッシュ配列: 5 5
「」

---

## STRIDE 分析 — サイドカーの影響

サイドカーは別個のダイアグラム ノードではありませんが、STRIDE 分析には表示されます。

- 異なる脅威の表面を持つサイドカー (MISE 認証バイパス、Dapr mTLS など) は、`2-stride-analysis.md` に独自の `## Component` セクションを取得します。
- コンポーネントの見出しは、どのポッドに同じ場所に配置されているかを示します。
- ポッド内通信に関連する脅威 (ローカルホスト バイパス、共有名前空間) は、**プライマリ コンテナ** コンポーネント セクションに分類されます。
- STRIDE テンプレートの **Pod Co-location** 行: 同じ場所に配置されたサイドカーのリスト (例: "MISE Sidecar, Dapr Sidecar")