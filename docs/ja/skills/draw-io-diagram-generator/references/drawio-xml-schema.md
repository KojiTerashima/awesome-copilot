# draw.io XML Schema Reference

`.drawio` file format (mxGraph XML) の完全 reference です。diagram file を生成、解析、検証するときに使ってください。

---

## Top-Level Structure

すべての `.drawio` file は、次の root structure を持つ XML です。

```xml
<!-- Set modified to the current ISO 8601 timestamp when generating a new file -->
<mxfile host="Electron" modified=""
        agent="draw.io" version="26.0.0" type="device">
  <diagram id="<unique-id>" name="<Page Name>">
    <mxGraphModel ...attributes...>
      <root>
        <mxCell id="0" />
        <mxCell id="1" parent="0" />
        <!-- All content cells here -->
      </root>
    </mxGraphModel>
  </diagram>
</mxfile>
```

### `<mxfile>` Attributes

| Attribute | Required | Default | Description |
| ----------- | ---------- | --------- | ------------- |
| `host` | No | `"app.diagrams.net"` | 生成元 editor (`"Electron"` は desktop / VS Code) |
| `modified` | No | — | ISO 8601 timestamp |
| `agent` | No | — | User agent string |
| `version` | No | — | draw.io version |
| `type` | No | `"device"` | Storage type |

### `<diagram>` Attributes

| Attribute | Required | Description |
| ----------- | ---------- | ------------- |
| `id` | Yes | 一意な page identifier (任意の string) |
| `name` | Yes | editor 上で表示される tab label |

### `<mxGraphModel>` Attributes

| Attribute | Type | Default | Description |
| ----------- | ------ | --------- | ------------- |
| `dx` | int | `1422` | Scroll X offset |
| `dy` | int | `762` | Scroll Y offset |
| `grid` | `0`/`1` | `1` | grid を表示 |
| `gridSize` | int | `10` | Grid snap size (px) |
| `guides` | `0`/`1` | `1` | alignment guide を表示 |
| `tooltips` | `0`/`1` | `1` | tooltip を有効化 |
| `connect` | `0`/`1` | `1` | hover 時の connection arrow を有効化 |
| `arrows` | `0`/`1` | `1` | directional arrow を表示 |
| `fold` | `0`/`1` | `1` | group の fold/collapse を有効化 |
| `page` | `0`/`1` | `1` | page boundary を表示 |
| `pageScale` | float | `1` | Page zoom scale |
| `pageWidth` | int | `1169` | Page width in px (A4 landscape) |
| `pageHeight` | int | `827` | Page height in px (A4 landscape) |
| `math` | `0`/`1` | `0` | LaTeX math rendering を有効化 |
| `shadow` | `0`/`1` | `0` | shape に global shadow を付ける |

**Common page sizes (96dpi での px):**

| Format | Width | Height |
| -------- | ------- | -------- |
| A4 landscape | `1169` | `827` |
| A4 portrait | `827` | `1169` |
| A3 landscape | `1654` | `1169` |
| Letter landscape | `1100` | `850` |
| Letter portrait | `850` | `1100` |
| Screen (16:9) | `1654` | `931` |

---

## Reserved Cells (Always Required)

```xml
<mxCell id="0" />                 <!-- Root cell — never omit, never add attributes -->
<mxCell id="1" parent="0" />     <!-- Default layer — all cells are children of this -->
```

この 2 つの cell は必ず `<root>` の最初の entry でなければなりません。ID `0` と `1` は予約済みで、他の cell に使ってはいけません。

---

## Vertex (Shape) Element

```xml
<mxCell
  id="2"
  value="Label Text"
  style="rounded=1;whiteSpace=wrap;html=1;"
  vertex="1"
  parent="1">
  <mxGeometry x="200" y="160" width="120" height="60" as="geometry" />
</mxCell>
```

### `<mxCell>` Vertex Attributes

| Attribute | Required | Type | Description |
| ----------- | ---------- | ------ | ------------- |
| `id` | Yes | string | この diagram 内で一意な identifier |
| `value` | Yes | string | label text (`html=1` があれば HTML 可) |
| `style` | Yes | string | semicolon 区切りの key=value style string |
| `vertex` | Yes | `"1"` | shape であることを示すため必ず `"1"` |
| `parent` | Yes | string | 親 cell ID (default layer は `"1"`) |

### `<mxGeometry>` Vertex Attributes

| Attribute | Required | Type | Description |
| ----------- | ---------- | ------ | ------------- |
| `x` | Yes | float | shape 左端の位置 (canvas origin からの px) |
| `y` | Yes | float | shape 上端の位置 (canvas origin からの px) |
| `width` | Yes | float | shape の幅 (px) |
| `height` | Yes | float | shape の高さ (px) |
| `as` | Yes | `"geometry"` | 常に `"geometry"` |

---

## Edge (Connector) Element

```xml
<mxCell
  id="5"
  value="Label"
  style="edgeStyle=orthogonalEdgeStyle;rounded=0;html=1;"
  edge="1"
  source="2"
  target="3"
  parent="1">
  <mxGeometry relative="1" as="geometry" />
</mxCell>
```

### `<mxCell>` Edge Attributes

| Attribute | Required | Type | Description |
| ----------- | ---------- | ------ | ------------- |
| `id` | Yes | string | 一意な identifier |
| `value` | Yes | string | connector label (ラベルなしなら空文字列) |
| `style` | Yes | string | style string (Edge Styles 参照) |
| `edge` | Yes | `"1"` | connector であることを示すため必ず `"1"` |
| `source` | No | string | source vertex の ID |
| `target` | No | string | target vertex の ID |
| `parent` | Yes | string | 親 cell ID (通常は `"1"`) |

### `<mxGeometry>` Edge Attributes

| Attribute | Required | Type | Description |
| ----------- | ---------- | ------ | ------------- |
| `relative` | No | `"1"` | edge では通常 `"1"` |
| `as` | Yes | `"geometry"` | 常に `"geometry"` |

### Edge with Label Offset

```xml
<mxGeometry x="-0.1" y="10" relative="1" as="geometry">
  <mxPoint as="offset" />
</mxGeometry>
```

relative geometry の `x` は edge 上のラベル位置を動かします (-1 〜 1)。`y` は直交方向の offset (px) です。

### Edge with Manual Waypoints (Control Points)

```xml
<mxGeometry relative="1" as="geometry">
  <Array as="points">
    <mxPoint x="340" y="80" />
    <mxPoint x="340" y="200" />
  </Array>
</mxGeometry>
```

---

## Multi-Page Diagrams

```xml
<mxfile>
  <diagram id="page-1" name="Overview">
    <mxGraphModel>...</mxGraphModel>
  </diagram>
  <diagram id="page-2" name="Detail">
    <mxGraphModel>...</mxGraphModel>
  </diagram>
</mxfile>
```

各 `<diagram>` は別 page / tab です。cell ID は各 `<diagram>` ごとに閉じた scope を持つため、別 page なら同じ ID を使っても衝突しません。

---

## Layer Cells

layer を使う場合は default `id="1"` layer の代わりに cell を定義します。cell は `parent` で layer に割り当てます。

```xml
<mxCell id="0" />
<mxCell id="1" value="Background" parent="0" />        <!-- layer 1 -->
<mxCell id="layer2" value="Services" parent="0" />     <!-- layer 2 -->
<mxCell id="layer3" value="Connectors" parent="0" />   <!-- layer 3 -->

<!-- Assign layer via parent attribute -->
<mxCell id="10" value="API" ... parent="layer2">
  <mxGeometry ... />
</mxCell>
```

layer visibility の切り替え:

```xml
<mxCell id="layer2" value="Services" parent="0" visible="0" />
```

---

## Swimlane Container

```xml
<!-- Swimlane container -->
<mxCell id="swim1" value="Process" style="shape=pool;startSize=30;horizontal=1;" 
        vertex="1" parent="1">
  <mxGeometry x="40" y="40" width="800" height="340" as="geometry" />
</mxCell>

<!-- Lane 1 (child of swimlane container) -->
<mxCell id="lane1" value="Customer" style="swimlane;startSize=30;" 
        vertex="1" parent="swim1">
  <mxGeometry x="0" y="30" width="800" height="150" as="geometry" />
</mxCell>

<!-- Shape inside lane (child of lane) -->
<mxCell id="step1" value="Place Order" style="rounded=1;whiteSpace=wrap;html=1;" 
        vertex="1" parent="lane1">
  <mxGeometry x="80" y="50" width="120" height="60" as="geometry" />
</mxCell>
```

> **Key**: swimlane 内の cell は `parent` を `"1"` ではなく **lane の ID** に設定します。  
> lane 内の座標は **lane origin 基準の相対座標** です。

---

## Group Cells

```xml
<!-- Invisible group container -->
<mxCell id="group1" value="" style="group;" vertex="1" parent="1">
  <mxGeometry x="100" y="100" width="300" height="200" as="geometry" />
</mxCell>

<!-- Children relative to group origin -->
<mxCell id="child1" value="A" style="rounded=1;" vertex="1" parent="group1">
  <mxGeometry x="20" y="20" width="100" height="60" as="geometry" />
</mxCell>
```

---

## HTML Labels

style に `html=1` があると、`value` に HTML を含められます。

```xml
<mxCell value="&lt;b&gt;OrderService&lt;/b&gt;&lt;br&gt;&lt;i&gt;:8080&lt;/i&gt;"
        style="rounded=1;html=1;" vertex="1" parent="1">
  <mxGeometry x="100" y="100" width="160" height="60" as="geometry" />
</mxCell>
```

HTML は XML escape が必要です。

- `<` → `&lt;`
- `>` → `&gt;`
- `&` → `&amp;`
- `"` → `&quot;`

よく使える HTML tag: `<b>`, `<i>`, `<u>`, `<br>`, `<font color="#hex">`, `<span style="...">`, `<hr/>`

---

## Tooltip / Metadata

```xml
<mxCell value="Service Name" tooltip="Handles order processing" style="..." vertex="1" parent="1">
  <mxGeometry ... />
</mxCell>
```

---

## ID Generation Rules

| Rule | Detail |
| ------ | -------- |
| IDs `0` and `1` | 予約済み — root と default layer 専用 |
| All other IDs | 各 `<diagram>` 内で一意である必要がある |
| Safe pattern | `2` から始まる連番、または UUID string |
| Cross-page | 別 `<diagram>` 間では一意でなくてよい |

**Safe sequential ID example:**

```text
id="2", id="3", id="4", ...
```

**UUID-style example:**

```text
id="a1b2c3d4-e5f6-7890-abcd-ef1234567890"
```

---

## Coordinate System

- origin `(0, 0)` は canvas の **左上**
- `x` は **右方向** に増える
- `y` は **下方向** に増える
- 単位はすべて **pixel**

---

## Recommended Spacing

| Context | Value |
| --------- | ------- |
| Minimum gap between shapes | `40px` |
| Comfortable gap | `80px` |
| Swimlane inner padding | `20px` |
| Page margin from edge | `40px` |
| Connector routing clearance | `10px` |

---

## Minimal Valid `.drawio` File

```xml
<mxfile host="Electron" modified="2026-03-25T00:00:00.000Z" version="26.0.0">
  <diagram id="main" name="Page-1">
    <mxGraphModel dx="1422" dy="762" grid="1" gridSize="10" guides="1"
                  tooltips="1" connect="1" arrows="1" fold="1"
                  page="1" pageScale="1" pageWidth="1169" pageHeight="827"
                  math="0" shadow="0">
      <root>
        <mxCell id="0" />
        <mxCell id="1" parent="0" />
      </root>
    </mxGraphModel>
  </diagram>
</mxfile>
```

---

## Validation Rules

### Must Pass

- [ ] `<root>` の最初の 2 child として `id="0"` と `id="1"` cell が必ず存在する
- [ ] 他の cell が `id="0"` または `id="1"` を使っていない
- [ ] 各 `<diagram>` 内の `id` value がすべて一意
- [ ] すべての `<mxCell>` がちょうど 1 つの `<mxGeometry>` child を持つ
- [ ] `<mxGeometry>` に `as="geometry"` attribute がある
- [ ] vertex cell は `vertex="1"`、edge cell は `edge="1"` を持つ
- [ ] edge の `source` / `target` ID が、同じ diagram の既存 vertex ID を参照する
- [ ] swimlane child の `parent` は `"1"` ではなく swimlane / lane ID を指す
- [ ] `value` attribute 内の HTML は XML escape されている

### Recommended

- [ ] 意図しない限り shape が重なっていない (目安は 40px 以上の gap)
- [ ] edge label は短い (4 words 以下)
- [ ] layer cell の `value` 名が分かりやすい
- [ ] すべての shape が `pageWidth` × `pageHeight` の範囲に収まっている
