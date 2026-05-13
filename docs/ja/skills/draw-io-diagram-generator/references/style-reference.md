# draw.io Style Reference

`<mxCell>` element の `style` attribute に関する完全 reference です。style は semicolon 区切りの `key=value` pair です。

---

## Style Format

```text
style="key1=value1;key2=value2;key3=value3;"
```

- key と value は case-sensitive
- 末尾の semicolon は省略可能ですが、付けることを推奨します
- 未知の key は黙って無視されます
- key がない場合は draw.io の default が使われます

---

## Universal Style Keys

すべての shape と edge に適用できます。

| Key | Values | Default | Description |
| ----- | -------- | --------- | ------------- |
| `fillColor` | `#hex` / `none` | `#FFFFFF` | shape の塗り色 (draw.io default。project では semantic palette 推奨) |
| `strokeColor` | `#hex` / `none` | `#000000` | border / line color (draw.io default。project では semantic palette 推奨) |
| `fontColor` | `#hex` | `#000000` | text color |
| `fontSize` | integer | `11` | font size (pt) |
| `fontStyle` | bitmask (see below) | `0` | bold / italic / underline |
| `fontFamily` | string | `Helvetica` | font family 名 |
| `align` | `left`/`center`/`right` | `center` | text の horizontal alignment |
| `verticalAlign` | `top`/`middle`/`bottom` | `middle` | text の vertical alignment |
| `opacity` | 0–100 | `100` | shape opacity (%) |
| `shadow` | `0`/`1` | `0` | drop shadow |
| `dashed` | `0`/`1` | `0` | dashed border |
| `dashPattern` | 例 `8 8` | — | custom dash/gap pattern (px) |
| `strokeWidth` | float | `2` | border / line width (px) |
| `spacing` | integer | `2` | text 周りの padding (px) |
| `spacingTop` | integer | `0` | 上側 text padding |
| `spacingBottom` | integer | `0` | 下側 text padding |
| `spacingLeft` | integer | `4` | 左側 text padding |
| `spacingRight` | integer | `4` | 右側 text padding |
| `html` | `0`/`1` | `0` | label で HTML を許可 |
| `whiteSpace` | `wrap`/`nowrap` | `nowrap` | text wrap |
| `overflow` | `visible`/`hidden`/`fill` | `visible` | text overflow の扱い |
| `rotatable` | `0`/`1` | `1` | editor で回転可能にする |
| `movable` | `0`/`1` | `1` | editor で移動可能にする |
| `resizable` | `0`/`1` | `1` | editor で resize 可能にする |
| `deletable` | `0`/`1` | `1` | editor で削除可能にする |
| `editable` | `0`/`1` | `1` | editor で label 編集可能にする |
| `locked` | `0`/`1` | `0` | 編集全体を lock する |
| `nolabel` | `0`/`1` | `0` | label を完全に隠す |
| `noLabel` | `0`/`1` | `0` | `nolabel` の alias |
| `labelPosition` | `left`/`center`/`right` | `center` | label anchor の horizontal 位置 |
| `verticalLabelPosition` | `top`/`middle`/`bottom` | `middle` | label anchor の vertical 位置 |
| `imageAlign` | `left`/`center`/`right` | `center` | image alignment |

### `fontStyle` Bitmask Values

| Value | Effect |
| ------- | -------- |
| `0` | Normal |
| `1` | Bold |
| `2` | Italic |
| `4` | Underline |
| `8` | Strikethrough |

足し算で組み合わせます: `3` = bold + italic, `5` = bold + underline, `7` = bold + italic + underline。

---

## Shape Keys (Vertex Only)

| Key | Values | Description |
| ----- | -------- | ------------- |
| `shape` | see Shape Catalog | default rectangle 以外の shape を指定 |
| `rounded` | `0`/`1` | rectangle の角を丸める |
| `arcSize` | 0–50 | corner radius の割合 (`rounded=1` 時) |
| `perimeter` | function name | 接続 perimeter type |
| `aspect` | `fixed` | resize 時の aspect ratio を固定 |
| `rotation` | float | 回転角 (degree) |
| `fixedSize` | `0`/`1` | label 編集時の auto-size を防ぐ |
| `container` | `0`/`1` | child を持つ container として扱う |
| `collapsible` | `0`/`1` | collapse / expand toggle を許可 |
| `startSize` | integer | swimlane / container header size (px) |
| `swimlaneHead` | `0`/`1` | swimlane header を表示 |
| `swimlaneBody` | `0`/`1` | swimlane body を表示 |
| `fillOpacity` | 0–100 | fill のみの opacity (`opacity` と独立) |
| `strokeOpacity` | 0–100 | stroke のみの opacity |
| `gradientColor` | `#hex` / `none` | gradient end color |
| `gradientDirection` | `north`/`south`/`east`/`west` | gradient direction |
| `sketch` | `0`/`1` | rough hand-drawn style |
| `comic` | `0`/`1` | comic/cartoon line style |
| `glass` | `0`/`1` | glass reflection effect |

---

## Shape Catalog

### Basic Shapes

| Shape | Style String | Visual |
| ------- | ------------- | -------- |
| Rectangle (default) | *(no shape key needed)* | □ |
| Rounded rectangle | `rounded=1;` | ▢ |
| Ellipse / Circle | `ellipse;` | ○ |
| Diamond | `rhombus;` | ◇ |
| Triangle | `triangle;` | △ |
| Hexagon | `shape=hexagon;` | ⬡ |
| Pentagon | `shape=mxgraph.basic.pentagon;` | ⬠ |
| Star | `shape=mxgraph.basic.star;` | ★ |
| Cross | `shape=mxgraph.basic.x;` | ✕ |
| Cloud | `shape=cloud;` | ☁ |
| Note / Callout | `shape=note;folded=1;` | 📝 |
| Document | `shape=document;` | 📄 |
| Cylinder (database) | `shape=cylinder3;` | 🗄 |
| Tape | `shape=tape;` | — |
| Parallelogram | `shape=parallelogram;perimeter=parallelogramPerimeter;` | ▱ |

### Flowchart Shapes (`mxgraph.flowchart.*`)

| Shape | Style String | Used For |
| ------- | ------------- | ---------- |
| Process | `shape=mxgraph.flowchart.process;` | 標準 process |
| Start/End (terminal) | `ellipse;` or `shape=mxgraph.flowchart.terminate;` | flow の開始 / 終了 |
| Decision | `rhombus;` | Yes / No 分岐 |
| Data (I/O) | `shape=mxgraph.flowchart.io;` | 入出力 |
| Predefined Process | `shape=mxgraph.flowchart.predefined_process;` | subroutine |
| Manual Input | `shape=mxgraph.flowchart.manual_input;` | manual entry |
| Manual Operation | `shape=mxgraph.flowchart.manual_operation;` | manual step |
| Database | `shape=mxgraph.flowchart.database;` | data store |
| Internal Storage | `shape=mxgraph.flowchart.internal_storage;` | internal data |
| Direct Data | `shape=mxgraph.flowchart.direct_data;` | drum storage |
| Document | `shape=mxgraph.flowchart.document;` | document |
| Multi-document | `shape=mxgraph.flowchart.multi-document;` | multiple document |
| On-page Connector | `ellipse;` (small) | page connector |
| Off-page Connector | `shape=mxgraph.flowchart.off_page_connector;` | off-page reference |
| Preparation | `shape=mxgraph.flowchart.preparation;` | initialization |
| Delay | `shape=mxgraph.flowchart.delay;` | wait state |
| Display | `shape=mxgraph.flowchart.display;` | display output |
| Sort | `shape=mxgraph.flowchart.sort;` | sort operation |
| Extract | `shape=mxgraph.flowchart.extract;` | extract operation |
| Merge | `shape=mxgraph.flowchart.merge;` | merge path |
| Or | `shape=mxgraph.flowchart.or;` | OR gate |
| And | `shape=mxgraph.flowchart.and;` | AND gate |
| Annotation | `shape=mxgraph.flowchart.annotation;` | comment / note |

### UML Shapes (`mxgraph.uml.*`)

| Shape | Style String | Used For |
| ------- | ------------- | ---------- |
| Actor | `shape=mxgraph.uml.actor;` | use-case actor |
| Boundary | `shape=mxgraph.uml.boundary;` | system boundary |
| Control | `shape=mxgraph.uml.control;` | controller object |
| Entity | `shape=mxgraph.uml.entity;` | entity object |
| Component | `shape=component;` | component box |
| Package | `shape=mxgraph.uml.package;` | package |
| Note | `shape=note;` | UML note |
| Lifeline | `shape=umlLifeline;startSize=40;` | sequence lifeline |
| Activation | `shape=umlActivation;` | activation box |
| Destroy | `shape=mxgraph.uml.destroy;` | destroy marker |
| State | `ellipse;` | state node |
| Initial State | `ellipse;fillColor=#000000;` | UML initial state |
| Final State | `shape=doubleEllipse;fillColor=#000000;` | UML final state |
| Fork/Join | `shape=mxgraph.uml.fork_or_join;` | fork/join bar |

### Network Shapes (`mxgraph.network.*`)

| Shape | Style String |
| ------- | ------------- |
| Server | `shape=server;` |
| Database server | `shape=mxgraph.network.database;` |
| Firewall | `shape=mxgraph.cisco.firewalls.firewall;` |
| Router | `shape=mxgraph.cisco.routers.router;` |
| Switch | `shape=mxgraph.cisco.switches.workgroup_switch;` |
| Cloud | `shape=cloud;` |
| Internet | `shape=mxgraph.network.internet;` |
| Laptop | `shape=mxgraph.network.laptop;` |
| Desktop | `shape=mxgraph.network.desktop;` |
| Mobile | `shape=mxgraph.network.mobile;` |

### AWS Shapes (`mxgraph.aws4.*`)

AWS4 library を使います。代表 shape:

| Shape | Style String |
| ------- | ------------- |
| EC2 | `shape=mxgraph.aws4.resourceIcon;resIcon=mxgraph.aws4.ec2;` |
| Lambda | `shape=mxgraph.aws4.resourceIcon;resIcon=mxgraph.aws4.lambda;` |
| S3 | `shape=mxgraph.aws4.resourceIcon;resIcon=mxgraph.aws4.s3;` |
| RDS | `shape=mxgraph.aws4.resourceIcon;resIcon=mxgraph.aws4.rds;` |
| API Gateway | `shape=mxgraph.aws4.resourceIcon;resIcon=mxgraph.aws4.api_gateway;` |
| CloudFront | `shape=mxgraph.aws4.resourceIcon;resIcon=mxgraph.aws4.cloudfront;` |
| Load Balancer | `shape=mxgraph.aws4.resourceIcon;resIcon=mxgraph.aws4.elb;` |
| SQS | `shape=mxgraph.aws4.resourceIcon;resIcon=mxgraph.aws4.sqs;` |
| SNS | `shape=mxgraph.aws4.resourceIcon;resIcon=mxgraph.aws4.sns;` |
| DynamoDB | `shape=mxgraph.aws4.resourceIcon;resIcon=mxgraph.aws4.dynamodb;` |
| ECS | `shape=mxgraph.aws4.resourceIcon;resIcon=mxgraph.aws4.ecs;` |
| EKS | `shape=mxgraph.aws4.resourceIcon;resIcon=mxgraph.aws4.eks;` |
| VPC | `shape=mxgraph.aws4.group;grIcon=mxgraph.aws4.group_vpc;` |
| Region | `shape=mxgraph.aws4.group;grIcon=mxgraph.aws4.group_region;` |

### Azure Shapes (`mxgraph.azure.*`)

| Shape | Style String |
| ------- | ------------- |
| App Service | `shape=mxgraph.azure.app_service;` |
| Function App | `shape=mxgraph.azure.function_apps;` |
| SQL Database | `shape=mxgraph.azure.sql_database;` |
| Blob Storage | `shape=mxgraph.azure.blob_storage;` |
| API Management | `shape=mxgraph.azure.api_management;` |
| Service Bus | `shape=mxgraph.azure.service_bus;` |
| AKS | `shape=mxgraph.azure.aks;` |
| Container Registry | `shape=mxgraph.azure.container_registry_registries;` |

### GCP Shapes (`mxgraph.gcp2.*`)

| Shape | Style String |
| ------- | ------------- |
| Cloud Run | `shape=mxgraph.gcp2.cloud_run;` |
| Cloud Functions | `shape=mxgraph.gcp2.cloud_functions;` |
| Cloud SQL | `shape=mxgraph.gcp2.cloud_sql;` |
| Cloud Storage | `shape=mxgraph.gcp2.cloud_storage;` |
| GKE | `shape=mxgraph.gcp2.container_engine;` |
| Pub/Sub | `shape=mxgraph.gcp2.cloud_pubsub;` |
| BigQuery | `shape=mxgraph.gcp2.bigquery;` |

---

## Edge Style Keys

| Key | Values | Description |
| ----- | -------- | ------------- |
| `edgeStyle` | see below | 接続 routing algorithm |
| `rounded` | `0`/`1` | orthogonal edge の corner を丸める |
| `curved` | `0`/`1` | curved segment を使う |
| `orthogonal` | `0`/`1` | orthogonal routing を強制 |
| `jettySize` | `auto`/integer | source / target jet size |
| `exitX` | 0.0–1.0 | source exit point X (0=left, 0.5=center, 1=right) |
| `exitY` | 0.0–1.0 | source exit point Y (0=top, 0.5=center, 1=bottom) |
| `exitDx` | float | source exit X offset (px) |
| `exitDy` | float | source exit Y offset (px) |
| `entryX` | 0.0–1.0 | target entry point X |
| `entryY` | 0.0–1.0 | target entry point Y |
| `entryDx` | float | target entry X offset (px) |
| `entryDy` | float | target entry Y offset (px) |
| `endArrow` | see Arrow Types | target 側の arrow head |
| `startArrow` | see Arrow Types | source 側の arrow tail |
| `endFill` | `0`/`1` | end arrow head を塗りつぶす |
| `startFill` | `0`/`1` | start arrow head を塗りつぶす |
| `endSize` | integer | end arrow head size (px) |
| `startSize` | integer | start arrow head size (px) |
| `labelBackgroundColor` | `#hex`/`none` | label 背景色 |
| `labelBorderColor` | `#hex`/`none` | label border color |

### `edgeStyle` Values

| Value | Routing | Use When |
| ------- | --------- | ---------- |
| `none` | 直線 | 単純な直接接続 |
| `orthogonalEdgeStyle` | 直角で曲がる | flowchart、architecture |
| `elbowEdgeStyle` | 1 回だけ折れる | 方向性が必要な diagram |
| `entityRelationEdgeStyle` | ER 用 routing | ER diagram |
| `segmentEdgeStyle` | handle 付き segmented routing | 細かく routing 調整したいとき |
| `isometricEdgeStyle` | isometric grid | isometric diagram |

### Arrow Types (`endArrow` / `startArrow`)

| Value | Shape | Use For |
| ------- | ------- | --------- |
| `block` | filled triangle | 標準の方向付き arrow |
| `open` | open chevron → | 軽い arrow |
| `classic` | classic arrow | draw.io 既定 arrow |
| `classicThin` | thin classic | compact diagram |
| `none` | arrowhead なし | 無向線 |
| `oval` | circle dot | aggregation start |
| `diamond` | hollow diamond | aggregation |
| `diamondThin` | thin diamond | slim diagram |
| `ERone` | `\|` bar | ER cardinality の "one" |
| `ERmany` | crow's foot | ER cardinality の "many" |
| `ERmandOne` | `\|\|` | ER mandatory one |
| `ERzeroToOne` | `o\|` | ER zero-or-one |
| `ERzeroToMany` | `o<` | ER zero-or-many |
| `ERoneToMany` | `\|<` | ER one-or-many |

---

## Color Palette

### Semantic Colors (一貫した diagram のための推奨 palette)

| Meaning | Fill | Stroke | Usage |
| --------- | ------ | -------- | ------- |
| User / Client | `#dae8fc` | `#6c8ebf` | browser、client app |
| Service / Process | `#d5e8d4` | `#82b366` | backend service |
| Database / Storage | `#f5f5f5` | `#666666` | database、file |
| Decision / Warning | `#fff2cc` | `#d6b656` | decision node、alert |
| Error / Critical | `#f8cecc` | `#b85450` | error path、critical |
| External / Partner | `#e1d5e7` | `#9673a6` | third-party、external |
| Queue / Async | `#ffe6cc` | `#d79b00` | message queue |
| Gateway / Proxy | `#dae8fc` | `#0050ef` | API gateway、proxy |

### Dark Background Shapes

dark theme の diagram では、次に差し替えます。

- Fill: `#1e4d78` (dark blue), `#1a4731` (dark green)
- Stroke: `#4aa3df`, `#67ab9f`
- Font: `#ffffff`

---

## Complete Style Examples

### Rounded Blue Box

```text
rounded=1;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;
```

### Green Process Step

```text
rounded=1;whiteSpace=wrap;html=1;fillColor=#d5e8d4;strokeColor=#82b366;
```

### Yellow Decision Diamond

```text
rhombus;whiteSpace=wrap;html=1;fillColor=#fff2cc;strokeColor=#d6b656;
```

### Red Error Box

```text
rounded=1;whiteSpace=wrap;html=1;fillColor=#f8cecc;strokeColor=#b85450;
```

### Database Cylinder

```text
shape=cylinder3;whiteSpace=wrap;html=1;boundedLbl=1;backgroundOutline=1;fillColor=#f5f5f5;strokeColor=#666666;
```

### Swimlane Container

```text
shape=pool;startSize=30;horizontal=1;fillColor=#f5f5f5;strokeColor=#999999;
```

### Swimlane Lane

```text
swimlane;startSize=30;fillColor=#ffffff;strokeColor=#999999;
```

### Orthogonal Connector

```text
edgeStyle=orthogonalEdgeStyle;rounded=0;html=1;
```

### Directed Arrow (bold)

```text
edgeStyle=orthogonalEdgeStyle;rounded=0;html=1;endArrow=block;endFill=1;strokeWidth=2;
```

### Dashed Dependency Line

```text
edgeStyle=orthogonalEdgeStyle;dashed=1;endArrow=open;endFill=0;strokeColor=#666666;
```

### ER Relationship Line (one-to-many)

```text
edgeStyle=entityRelationEdgeStyle;html=1;endArrow=ERmany;startArrow=ERmandOne;endFill=1;startFill=1;
```

### UML Inheritance Arrow (hollow triangle)

```text
edgeStyle=orthogonalEdgeStyle;html=1;endArrow=block;endFill=0;
```

### UML Composition (filled diamond)

```text
edgeStyle=orthogonalEdgeStyle;html=1;startArrow=diamond;startFill=1;endArrow=none;
```

### UML Aggregation (open diamond)

```text
edgeStyle=orthogonalEdgeStyle;html=1;startArrow=diamond;startFill=0;endArrow=none;
```

### UML Dependency (dashed arrow)

```text
edgeStyle=orthogonalEdgeStyle;dashed=1;html=1;endArrow=open;endFill=0;
```

### Invisible connector (for alignment)

```text
edgeStyle=none;strokeColor=none;endArrow=none;
```
