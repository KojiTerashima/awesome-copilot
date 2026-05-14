# スケルトン: 1.2-threatmodel-summary.mmd

> **⛔ `1.1-threatmodel.mmd`.** を作成した後は、常にこのスケルトンを評価してください。
> 詳細 DFD 内の要素 (`(("..."))`、`[("...")]`、`["..."]` を持つノード) と境界 (`subgraph`) をカウントします。
> - 要素が 15 を超えるか、境界が 4 を超える場合 → このファイルは **必須**です。以下のテンプレートに記入します。
> - 要素 ≤ 15 かつ境界 ≤ 4 の場合 → **このファイルをスキップ**。 `1-threatmodel.md` に進みます。
> **⛔ これは生の Mermaid ファイルです。以下のテンプレートは、読みやすさを目的としてコード フェンス内に示されています。出力ファイルにはフェンスを含めないでください。 `.mmd` ファイルは、1 行目の `%%{init:` で始まる必要があります。**

---```
%%{init: {'theme': 'base', 'themeVariables': { 'background': '#ffffff', 'primaryColor': '#ffffff', 'lineColor': '#666666' }}}%%
flowchart LR
    classDef process fill:#6baed6,stroke:#2171b5,stroke-width:2px,color:#000000
    classDef external fill:#fdae61,stroke:#d94701,stroke-width:2px,color:#000000
    classDef datastore fill:#74c476,stroke:#238b45,stroke-width:2px,color:#000000

    [FILL: External actors — keep all, do not aggregate]
    [FILL: ExternalActor]["[FILL: Name]"]:::external

    [REPEAT: one subgraph per trust boundary — ALL boundaries MUST be preserved]
    subgraph [FILL: BoundaryID]["[FILL: Boundary Name]"]
        [FILL: Aggregated and individual nodes]
    end
    [END-REPEAT]

    [REPEAT: summary data flows using SDF prefix]
    [FILL: Source] <-->|"[FILL: SDF##: description]"| [FILL: Target]
    [END-REPEAT]

    [REPEAT: boundary styles]
    style [FILL: BoundaryID] fill:none,stroke:#e31a1c,stroke-width:3px,stroke-dasharray: 5 5
    [END-REPEAT]

    linkStyle default stroke:#666666,stroke-width:2px
```## 集計ルール

**参照:** `diagram-conventions.md` → 詳細については、概要図のルールを参照してください。

1. **すべての信頼境界を保持する必要があります** — 境界を結合したり省略したりしないでください。
2. **個別に保持します:** エントリ ポイント、コア フロー コンポーネント、セキュリティ クリティカルなサービス、プライマリ データ ストア、すべての外部アクター。
3. **集約のみ:** インフラストラクチャ、二次キャッシュ、同じ信頼レベルの複数の外部をサポートします。
4. **集約要素ラベルには内容をリストする必要があります:**```
   DataLayer[("Data Layer<br/>(UserDB, OrderDB, Redis)")]
   SupportServices(("Supporting<br/>(Logging, Monitoring)"))
   ```5. **フロー ID:** `SDF` プレフィックスを使用します: `SDF01`、`SDF02` ...

## `1-threatmodel.md` では必須です

このファイルが生成されるとき、`1-threatmodel.md` には以下を含める必要があります。
- この図を ` で囲んだ `## Summary View` セクション```mermaid ` fence
- A `## Summary to Detailed Mapping` table:

```値下げ
|概要要素 |含まれています |フローの概要 |詳細なフローへのマッピング |
|-----|----------|---------------|--------------------------|
| [フィル] | [FILL: 詳細要素のリスト] | [記入: SDF##] | [フィル: DF## リスト] |
「」