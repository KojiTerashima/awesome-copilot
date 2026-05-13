---
name: draw-io-diagram-generator
description: draw.io diagram file (.drawio, .drawio.svg, .drawio.png) を作成、編集、生成するときに使います。mxGraph XML authoring、shape library、style string、flowchart、system architecture、sequence diagram、ER diagram、UML class diagram、network topology、layout strategy、hediet.vscode-drawio VS Code extension、そして依頼からそのまま開けるファイル完成までの agent workflow 全体を扱います。
---

# Draw.io Diagram Generator

この skill を使うと、正しい mxGraph XML 構造を持つ draw.io (`.drawio`) diagram file を生成、編集、検証できます。生成された file はすべて、手動修正なしで [Draw.io VS Code extension](https://marketplace.visualstudio.com/items?itemName=hediet.vscode-drawio) (`hediet.vscode-drawio`) ですぐに開けます。必要なら、draw.io web app や desktop app でも開けます。

---

## 1. When to Use This Skill

**Trigger phrases (これらを見たらこの skill を読み込む)**

- "create a diagram", "draw a flowchart", "generate an architecture diagram"
- "design a sequence diagram", "make a UML class diagram", "build an ER diagram"
- "add a .drawio file", "update the diagram", "visualise the flow"
- "document the architecture", "show the data model", "diagram the service interactions"
- `.drawio`, `.drawio.svg`, `.drawio.png` file の生成または変更を求めるあらゆる依頼

**Supported diagram types**

| Diagram Type | Template Available | Description |
|---|---|---|
| Flowchart | `assets/templates/flowchart.drawio` | 分岐や判断を含む process flow |
| System Architecture | `assets/templates/architecture.drawio` | multi-tier / layered service architecture |
| Sequence Diagram | `assets/templates/sequence.drawio` | actor lifeline と時系列 message flow |
| ER Diagram | `assets/templates/er-diagram.drawio` | relationship 付き database table |
| UML Class Diagram | `assets/templates/uml-class.drawio` | class、interface、enum、relationship |
| Network Topology | (use shape library) | router、server、firewall、subnet |
| BPMN Workflow | (use shape library) | business process event、task、gateway |
| Mind Map | (manual) | 中心 topic から放射する branch |

---

## 2. Prerequisites

- VS Code integration が有効なら、drawio extension を install してください: **draw.io VS Code extension** — `hediet.vscode-drawio` (extension id)。install は次で行えます:
  ```
  ext install hediet.vscode-drawio
  ```
- **Supported file extensions**: `.drawio`, `.drawio.svg`, `.drawio.png`
- **Python 3.8+** (optional) — `scripts/` 内の validation / shape insertion script 用

---

## 3. Step-by-Step Agent Workflow

diagram generation task では毎回この順番で進めてください。

### Step 1 — Understand the Request

確認または推定する項目:
1. **Diagram type** — どんな種類の diagram か (flowchart, architecture, UML, ER, sequence, network...)
2. **Entities / actors** — 主な component、actor、class、table は何か
3. **Relationships** — どう接続されるか。方向は。cardinality は
4. **Output path** — `.drawio` file をどこへ保存するか
5. **Existing file** — 新規作成か既存 file 編集か

依頼が曖昧なら、文脈から最も自然な diagram type を推定してください (例: "show the tables" → ER diagram, "show how the API call flows" → sequence diagram)。

### Step 2 — Select a Template or Start Fresh

- diagram type が `assets/templates/` の template に合うなら **template を使う**。template 構造をコピーして placeholder を置き換えます。
- 新しい layout なら **新規作成**。次の最小 valid skeleton から始めます。

```xml
<!-- Set modified="" to the current ISO 8601 timestamp when generating a new file -->
<mxfile host="Electron" modified="" version="26.0.0">
  <diagram id="page-1" name="Page-1">
    <mxGraphModel dx="1422" dy="762" grid="1" gridSize="10" guides="1"
                  tooltips="1" connect="1" arrows="1" fold="1"
                  page="1" pageScale="1" pageWidth="1169" pageHeight="827"
                  math="0" shadow="0">
      <root>
        <mxCell id="0" />
        <mxCell id="1" parent="0" />
        <!-- Your cells go here -->
      </root>
    </mxGraphModel>
  </diagram>
</mxfile>
```

> **Rule**: id `0` と `1` は **常に必須** で、最初の 2 cell でなければなりません。再利用してはいけません。

### Step 3 — Plan the Layout

XML を生成する前に、論理配置を先に考えます。
- **row** または **tier** に整理する (layer には swimlane を使う)
- **Horizontal spacing**: 同じ row の shape 間は 40–60px
- **Vertical spacing**: tier row 間は 80–120px
- 標準 shape size: process box は `120x60` px、swimlane は `160x80` px
- 既定 canvas: A4 landscape = `1169 x 827` px

### Step 4 — Generate the mxGraph XML

**Vertex cell** (すべての shape):
```xml
<mxCell id="unique-id" value="Label"
        style="rounded=1;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;"
        vertex="1" parent="1">
  <mxGeometry x="100" y="100" width="120" height="60" as="geometry" />
</mxCell>
```

**Edge cell** (すべての connector):
```xml
<mxCell id="edge-id" value="Label (optional)"
        style="edgeStyle=orthogonalEdgeStyle;html=1;"
        edge="1" source="source-id" target="target-id" parent="1">
  <mxGeometry relative="1" as="geometry" />
</mxCell>
```

**Critical rules**:
- すべての cell id は file 内で **globally unique** であること
- すべての vertex は `x`, `y`, `width`, `height`, `as="geometry"` を持つ `mxGeometry` child を持つこと
- すべての edge は既存 vertex id に一致する `source` と `target` を持つこと。**例外**: floating edge (例: sequence diagram の lifeline) は代わりに `<mxGeometry>` 内の `sourcePoint` / `targetPoint` を使います。§4 Sequence Diagram 参照
- すべての cell の `parent` は既存 cell id を参照すること
- label に HTML (`<b>`, `<i>`, `<br>`) を含むなら style に `html=1` を入れること
- label 内の XML special character を escape すること: `&` => `&amp;`, `<` => `&lt;`, `>` => `&gt;`

### Step 5 — Apply Correct Styles

一貫性のため、標準の semantic color palette を使います。

| Purpose | fillColor | strokeColor |
|---|---|---|
| Primary / Info | `#dae8fc` | `#6c8ebf` |
| Success / Start | `#d5e8d4` | `#82b366` |
| Warning / Decision | `#fff2cc` | `#d6b656` |
| Error / End | `#f8cecc` | `#b85450` |
| Neutral | `#f5f5f5` | `#666666` |
| External / Partner | `#e1d5e7` | `#9673a6` |

diagram type ごとの代表 style string:

```
# Rounded process box (flowchart)
rounded=1;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;

# Decision diamond
rhombus;whiteSpace=wrap;html=1;fillColor=#fff2cc;strokeColor=#d6b656;

# Start/End terminal
ellipse;whiteSpace=wrap;html=1;fillColor=#d5e8d4;strokeColor=#82b366;

# Database cylinder
shape=mxgraph.flowchart.database;whiteSpace=wrap;html=1;fillColor=#f8cecc;strokeColor=#b85450;

# Swimlane container (tier)
swimlane;startSize=30;fillColor=#dae8fc;strokeColor=#6c8ebf;fontStyle=1;

# UML class box
swimlane;fontStyle=1;align=center;startSize=40;fillColor=#dae8fc;strokeColor=#6c8ebf;

# Interface / stereotype box
swimlane;fontStyle=3;align=center;startSize=40;fillColor=#f5f5f5;strokeColor=#666666;

# ER table container
shape=table;startSize=30;container=1;collapsible=1;childLayout=tableLayout;

# Orthogonal connector
edgeStyle=orthogonalEdgeStyle;html=1;

# ER relationship (crow's foot)
edgeStyle=entityRelationEdgeStyle;html=1;endArrow=ERmany;startArrow=ERone;
```

> 完全な style key catalog は `references/style-reference.md` を、shape library 名は `references/shape-libraries.md` を参照してください。

### Step 6 — Save and Validate

1. requested path に `.drawio` extension で **file を書き出す**
2. **validator を実行** (optional だが推奨):
   ```bash
   python .github/skills/draw-io-diagram-generator/scripts/validate-drawio.py <path-to-file.drawio>
   ```
3. **ユーザーに伝える**:
   > "Open `<filename>` in VS Code — it will render automatically with the draw.io extension. You can use draw.io's web app or desktop app as well if you prefer."
4. **diagram の内容を短く説明**して、何が入っているか分かるようにする

---

## 4. Diagram-Type Recipes

### Flowchart

主要要素: Start (ellipse) => Process (rounded rectangle) => Decision (diamond) => End (ellipse)

```xml
<!-- Start node -->
<mxCell id="start" value="Start"
        style="ellipse;whiteSpace=wrap;html=1;fillColor=#d5e8d4;strokeColor=#82b366;"
        vertex="1" parent="1">
  <mxGeometry x="500" y="80" width="120" height="60" as="geometry" />
</mxCell>

<!-- Process -->
<mxCell id="p1" value="Process Step"
        style="rounded=1;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;"
        vertex="1" parent="1">
  <mxGeometry x="500" y="200" width="120" height="60" as="geometry" />
</mxCell>

<!-- Decision -->
<mxCell id="d1" value="Condition?"
        style="rhombus;whiteSpace=wrap;html=1;fillColor=#fff2cc;strokeColor=#d6b656;"
        vertex="1" parent="1">
  <mxGeometry x="460" y="320" width="200" height="100" as="geometry" />
</mxCell>

<!-- Arrow: start to p1 -->
<mxCell id="e1" value=""
        style="edgeStyle=orthogonalEdgeStyle;html=1;"
        edge="1" source="start" target="p1" parent="1">
  <mxGeometry relative="1" as="geometry" />
</mxCell>
```

### Architecture Diagram (3-tier)

各 tier には **swimlane container** を使います。service box はすべてその swimlane の child にします。

```xml
<!-- Tier swimlane -->
<mxCell id="tier1" value="Client Layer"
        style="swimlane;startSize=30;fillColor=#dae8fc;strokeColor=#6c8ebf;fontStyle=1;"
        vertex="1" parent="1">
  <mxGeometry x="60" y="100" width="1050" height="130" as="geometry" />
</mxCell>

<!-- Service inside tier (parent="tier1", coords are relative to tier) -->
<mxCell id="webapp" value="Web App"
        style="rounded=1;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;"
        vertex="1" parent="tier1">
  <mxGeometry x="80" y="40" width="120" height="60" as="geometry" />
</mxCell>
```

> tier 間 connector は `parent="1"` を使う absolute coordinate です。

### Sequence Diagram

主要要素: Actor (上部)、Lifeline (破線の縦線)、Activation box、Message arrow。

- Lifeline: `endArrow=none` と `dashed=1` を持つ `edge="1"`。source/target は持たず、geometry に `sourcePoint` / `targetPoint` を使う
- Synchronous message: `endArrow=block;endFill=1`
- Return message: `endArrow=open;endFill=0;dashed=1`
- Self-call: edge を右へループして戻す Array point を使う

**Minimal XML snippet:**

```xml
<!-- Actor (stick figure) -->
<mxCell id="actorA" value="Client"
        style="shape=mxgraph.uml.actor;pointerEvents=1;dashed=0;whiteSpace=wrap;html=1;aspect=fixed;"
        vertex="1" parent="1">
  <mxGeometry x="110" y="80" width="60" height="80" as="geometry" />
</mxCell>

<!-- Service box -->
<mxCell id="actorB" value="API Server"
        style="rounded=1;whiteSpace=wrap;html=1;fillColor=#fff2cc;strokeColor=#d6b656;"
        vertex="1" parent="1">
  <mxGeometry x="480" y="100" width="160" height="60" as="geometry" />
</mxCell>

<!-- Lifeline — floating edge: uses sourcePoint/targetPoint, NOT source/target attributes -->
<mxCell id="lifA" value=""
        style="edgeStyle=none;dashed=1;endArrow=none;"
        edge="1" parent="1">
  <mxGeometry relative="1" as="geometry">
    <mxPoint x="140" y="160" as="sourcePoint" />
    <mxPoint x="140" y="700" as="targetPoint" />
  </mxGeometry>
</mxCell>

<!-- Activation box (thin rectangle on lifeline) -->
<mxCell id="actA1" value=""
        style="fillColor=#dae8fc;strokeColor=#6c8ebf;"
        vertex="1" parent="1">
  <mxGeometry x="130" y="220" width="20" height="180" as="geometry" />
</mxCell>

<!-- Synchronous message -->
<mxCell id="msg1" value="POST /orders"
        style="edgeStyle=elbowEdgeStyle;elbow=vertical;html=1;endArrow=block;endFill=1;"
        edge="1" source="actA1" target="actorB" parent="1">
  <mxGeometry relative="1" as="geometry" />
</mxCell>

<!-- Return message (dashed) -->
<mxCell id="msg2" value="201 Created"
        style="edgeStyle=elbowEdgeStyle;elbow=vertical;dashed=1;html=1;endArrow=open;endFill=0;"
        edge="1" source="actorB" target="actA1" parent="1">
  <mxGeometry relative="1" as="geometry" />
</mxCell>
```

> **Note:** Lifeline は `<mxGeometry>` 内の `sourcePoint` / `targetPoint` を使う floating edge です。`source` / `target` attribute は使いません。これは sequence diagram における標準 draw.io pattern です。

### ER Diagram

`shape=table` container に `childLayout=tableLayout` を組み合わせて使います。row は `shape=tableRow` cell、`portConstraint=eastwest` を持ちます。各 row 内 column は `shape=partialRectangle` です。

relationship arrow は `edgeStyle=entityRelationEdgeStyle` を使います:
- One-to-One: `startArrow=ERone;endArrow=ERone`
- One-to-Many: `startArrow=ERone;endArrow=ERmany`
- Many-to-Many: `startArrow=ERmany;endArrow=ERmany`
- Mandatory: `ERmandOne`, Optional: `ERzeroToOne`

### UML Class Diagram

class box は swimlane container です。attribute と method は plain text cell、divider は高さ 0 の swimlane child を使います。

relationship type ごとの arrow style:

| Relationship | Style String |
|---|---|
| Inheritance (extends) | `edgeStyle=orthogonalEdgeStyle;html=1;endArrow=block;endFill=0;` |
| Realization (implements) | `edgeStyle=orthogonalEdgeStyle;dashed=1;html=1;endArrow=block;endFill=0;` |
| Composition | `edgeStyle=orthogonalEdgeStyle;html=1;startArrow=diamond;startFill=1;endArrow=none;` |
| Aggregation | `edgeStyle=orthogonalEdgeStyle;html=1;startArrow=diamond;startFill=0;endArrow=none;` |
| Dependency | `edgeStyle=orthogonalEdgeStyle;dashed=1;html=1;endArrow=open;endFill=0;` |
| Association | `edgeStyle=orthogonalEdgeStyle;html=1;endArrow=open;endFill=0;` |

---

## 5. Multi-Page Diagrams

複雑な system では複数の `<diagram>` element を追加します。

```xml
<mxfile host="Electron" version="26.0.0">
  <diagram id="overview" name="Overview">
    <!-- overview mxGraphModel -->
  </diagram>
  <diagram id="detail" name="Detail View">
    <!-- detail mxGraphModel -->
  </diagram>
</mxfile>
```

各 page は独立した cell id namespace を持ちます。別 page なら同じ id を使っても衝突しません。

---

## 6. Editing Existing Diagrams

既存 `.drawio` file を変更するとき:

1. まず file を **read** し、既存 cell id、position、parent hierarchy を把握する
2. 対象 **diagram page** を特定する — index または `name` attribute で
3. 既存 id と衝突しない **新しい unique id** を割り当てる
4. **container hierarchy** を守る — swimlane の child の座標は parent 基準の相対座標
5. **edge を検証**する — node を移動したら source/target id が有効なままか確認する

raw XML を直接編集せず、`scripts/add-shape.py` を使えば 1 つの shape を安全に追加できます。
```bash
python .github/skills/draw-io-diagram-generator/scripts/add-shape.py docs/arch.drawio "New Service" 700 380
```

---

## 7. Best Practices

**Layout**
- 10px grid に揃える (すべての座標を 10 の倍数にする)
- 関連する shape は swimlane container にまとめる
- 1 page 1 topic を原則にし、複雑な system は multi-page file にする
- 可読性のため、1 page あたり 40 cell 以下を目安にする

**Labels**
- 各 page の先頭に title text cell (`text;strokeColor=none;fillColor=none;fontSize=18;fontStyle=1`) を置く
- vertex shape には常に `whiteSpace=wrap;html=1` を入れる
- label は簡潔に。可能なら 1 shape 3 words 以下

**Style consistency**
- project 全体で Section 3 Step 5 の semantic color palette を一貫して使う
- きれいな直角 connector のため、`edgeStyle=orthogonalEdgeStyle` を優先する
- 必要がない限り label に任意の HTML を埋め込まない

**File naming**
- kebab-case を使う: `order-service-flow.drawio`, `database-schema.drawio`
- diagram は説明対象の code の近くへ置く: `docs/` または `architecture/`

---

## 8. Troubleshooting

| Problem | Likely Cause | Fix |
|---|---|---|
| File opens blank in VS Code | id=0 または id=1 cell がない | 他の cell より前に両方の root cell を追加する |
| Shape at wrong position | container の child で、座標が相対になっている | `parent` を確認し、container 基準で x/y を調整する |
| Edge not visible | source または target id が既存 vertex と一致しない | 両方の id が正確に存在するか確認する |
| Diagram shows "Compressed" | mxGraphModel が base64 encode されている | draw.io web で開き、File > Export > XML (uncompressed) を使う |
| Shape style not rendering | shape= 名に typo がある | 正確な style string を `references/shape-libraries.md` で確認する |
| Label shows escaped HTML | HTML label を含む cell で html=0 になっている | cell style に `html=1;` を追加する |
| Container children overlap container edge | container 高さが不足している | mxGeometry の container height を大きくする |

---

## 9. Validation Checklist

生成した `.drawio` file を渡す前に、次を確認してください。

- [ ] File が `<mxfile>` root element で始まる
- [ ] すべての `<diagram>` に空でない `id` attribute がある
- [ ] 各 diagram で `<mxCell id="0" />` が最初の cell
- [ ] 各 diagram で `<mxCell id="1" parent="0" />` が 2 番目の cell
- [ ] 各 diagram 内の cell `id` はすべて一意
- [ ] すべての vertex cell が `vertex="1"` と child `<mxGeometry as="geometry">` を持つ
- [ ] すべての edge cell が `edge="1"` を持ち、さらに (a) 既存 vertex id を指す `source` / `target`、または (b) `<mxGeometry>` 内に `<mxPoint as="sourcePoint">` と `<mxPoint as="targetPoint">` を持つ (floating edge — sequence diagram の lifeline 用)
- [ ] id=0 を除くすべての cell が、既存 id を指す `parent` を持つ
- [ ] HTML tag を含む label の style に `html=1` が入っている
- [ ] XML が well-formed である (未閉じ tag なし、attribute value 内の `&`, `<`, `>` 未 escape なし)
- [ ] 各 page 上部に title label cell がある

automated validator を実行:
```bash
python .github/skills/draw-io-diagram-generator/scripts/validate-drawio.py <file.drawio>
```

---

## 10. Output Format

diagram を渡すときは、常に次を含めてください。

1. requested path に書き出した **`.drawio` file**
2. diagram が何を示すかの **1 文 summary**
3. **開き方**:
   > "Open `<filename>` in VS Code — the draw.io extension will render it automatically. Or you can open it in the draw.io web app or desktop app if you prefer."
4. **編集方法** (ユーザーが調整しそうな場合):
   > "Click any shape to select it. Double-click to edit the label. Drag to reposition."
5. **Validation status** — validator script を実行し、pass したかどうか

---

## 11. References

companion file はすべて `.github/skills/draw-io-diagram-generator/` にあります。

| File | Contents |
|---|---|
| `references/drawio-xml-schema.md` | 完全な mxfile / mxGraphModel / mxCell attribute reference、coordinate system、reserved cell、validation rule |
| `references/style-reference.md` | 使用可能な value を含む全 style key、vertex / edge style key、shape catalog、semantic color palette |
| `references/shape-libraries.md` | shape library category (General, Flowchart, UML, ER, Network, BPMN, Mockup, K8s) と style string |
| `assets/templates/flowchart.drawio` | すぐ使える flowchart template |
| `assets/templates/architecture.drawio` | 4-tier system architecture template |
| `assets/templates/sequence.drawio` | 3-actor sequence diagram template |
| `assets/templates/er-diagram.drawio` | crow's foot relationship 付き 3-table ER diagram |
| `assets/templates/uml-class.drawio` | interface + 2 class + enum と relationship arrow |
| `scripts/validate-drawio.py` | 任意の .drawio file の XML 構造を検証する Python script |
| `scripts/add-shape.py` | 既存 diagram に新しい shape を追加する Python CLI |
| `scripts/README.md` | script の使い方と example |
