# スケルトン: 1.1-threatmodel.mmd

> **⛔ これは生の Mermaid ファイルです。マークダウン ラッパーはありません。 1 行目は `%%{init:`.** で始まらなければなりません
> **init ブロック、classDefs、および linkStyle は修正されています。色やストロークは決して変更しないでください。**
> **図の方向は常に `flowchart LR` です — 決して `flowchart TB`.**
> **⛔ 以下のテンプレートは、読みやすさを目的としてコード フェンス内に示されています。出力ファイルにはフェンスを含めないでください。**

---```
%%{init: {'theme': 'base', 'themeVariables': { 'background': '#ffffff', 'primaryColor': '#ffffff', 'lineColor': '#666666' }}}%%
flowchart LR
    classDef process fill:#6baed6,stroke:#2171b5,stroke-width:2px,color:#000000
    classDef external fill:#fdae61,stroke:#d94701,stroke-width:2px,color:#000000
    classDef datastore fill:#74c476,stroke:#238b45,stroke-width:2px,color:#000000
    [CONDITIONAL: incremental mode — include BOTH lines below]
    classDef newComponent fill:#d4edda,stroke:#28a745,stroke-width:3px,color:#000000
    classDef removedComponent fill:#e9ecef,stroke:#6c757d,stroke-width:1px,stroke-dasharray:5,color:#6c757d
    [END-CONDITIONAL]

    [REPEAT: one line per external actor/interactor — outside all subgraphs]
    [FILL: NodeID]["[FILL: Display Name]"]:::external
    [END-REPEAT]

    [REPEAT: one subgraph per trust boundary]
    subgraph [FILL: BoundaryID]["[FILL: Boundary Display Name]"]
        [REPEAT: processes and datastores inside this boundary]
        [FILL: NodeID](("[FILL: Process Name]")):::process
        [FILL: NodeID][("[FILL: DataStore Name]")]:::datastore
        [END-REPEAT]
    end
    [END-REPEAT]

    [REPEAT: one line per data flow — use <--> for bidirectional request-response]
    [FILL: SourceID] <-->|"[FILL: DF##: description]"| [FILL: TargetID]
    [END-REPEAT]

    [REPEAT: one style line per trust boundary subgraph]
    style [FILL: BoundaryID] fill:none,stroke:#e31a1c,stroke-width:3px,stroke-dasharray: 5 5
    [END-REPEAT]

    linkStyle default stroke:#666666,stroke-width:2px
```**これらの固定要素は決して変更しないでください:**
- `%%{init:` テーマ変数: `background`、`primaryColor`、`lineColor` のみ
- `flowchart LR` — 決してTBしないでください
- classDef color: process=#6baed6/#2171b5、external=#fdae61/#d94701、datastore=#74c476/#238b45
- 増分 classDefs (該当する場合): newComponent=#d4edda/#28a745 (薄緑色)、removedComponent=#e9ecef/#6c757d (灰色の破線)
- 新しいコンポーネントは `:::newComponent` (`:::process` ではありません) を使用しなければなりません。削除されたコンポーネントでは `:::removedComponent` を使用する必要があります。
- 信頼境界のスタイル: `fill:none,stroke:#e31a1c,stroke-width:3px,stroke-dasharray: 5 5`
- リンクスタイル: `stroke:#666666,stroke-width:2px`

**DFD 形状:**
- 処理: `(("Name"))` (二重括弧 = 丸)
- データ ストア: `[("Name")]` (括弧括弧 = シリンダー)
- 外部: `["Name"]` (括弧 = 長方形)
- すべてのラベルは `""` で引用符で囲む必要があります
- すべてのサブグラフ ID: `subgraph ID["Title"]`

<!-- ⛔ POST-DFD GATE — このファイルを作成した直後:
  1. 要素ノードを数える: (("..."))、[("...")]、["..."] の形状を持つ線
  2. 境界を数える: 「サブグラフ」を含む行
  3. 要素が 15 を超えるか、境界が 4 を超える場合:
     →今すぐskeleton-summary-dfd.mdを開いて1.2-threatmodel-summary.mmdを作成します。
     → 概要が存在するまで 1-threatmodel.md に進まないでください。
  4. しきい値を満たしていない場合 → サマリーをスキップし、1-threatmodel.md に進みます。
  これは最も頻繁にスキップされるステップです。ゲートは必須です。 -->