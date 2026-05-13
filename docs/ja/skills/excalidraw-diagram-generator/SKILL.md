---
name: excalidraw-diagram-generator
description: '自然言語の説明から Excalidraw diagram を生成します。"create a diagram", "make a flowchart", "visualize a process", "draw a system architecture", "create a mind map", "generate an Excalidraw file" といった依頼で使います。flowchart、relationship diagram、mind map、system architecture diagram をサポートします。Excalidraw で直接開ける .excalidraw JSON file を出力します。'
---

# Excalidraw Diagram Generator

自然言語の説明から Excalidraw 形式の diagram を生成するための skill です。手で描かなくても、process、system、relationship、idea を視覚化できます。

## When to Use This Skill

ユーザーが次のように依頼したときに使います。

- "Create a diagram showing..."
- "Make a flowchart for..."
- "Visualize the process of..."
- "Draw the system architecture of..."
- "Generate a mind map about..."
- "Create an Excalidraw file for..."
- "Show the relationship between..."
- "Diagram the workflow of..."

**Supported diagram types:**
- 📊 **Flowcharts**: 順次 process、workflow、decision tree
- 🔗 **Relationship Diagrams**: entity relationship、system component、dependency
- 🧠 **Mind Maps**: concept hierarchy、brainstorming 結果、topic 整理
- 🏗️ **Architecture Diagrams**: system design、module interaction、data flow
- 📈 **Data Flow Diagrams (DFD)**: data flow の可視化、data transformation process
- 🏊 **Business Flow (Swimlane)**: cross-functional workflow、actor ベースの process flow
- 📦 **Class Diagrams**: object-oriented design、class structure と relationship
- 🔄 **Sequence Diagrams**: 時系列での object interaction、message flow
- 🗃️ **ER Diagrams**: database entity relationship、data model

## Prerequisites

- 何を可視化したいかの明確な説明
- 主要な entity、step、concept の特定
- 要素間の relationship または flow の理解

## Step-by-Step Workflow

### Step 1: Understand the Request

ユーザーの説明を分析し、次を判断します。
1. **Diagram type** (flowchart, relationship, mind map, architecture)
2. **Key elements** (entity、step、concept)
3. **Relationships** (flow、connection、hierarchy)
4. **Complexity** (要素数)

### Step 2: Choose the Appropriate Diagram Type

| User Intent | Diagram Type | Example Keywords |
|-------------|--------------|------------------|
| Process flow, steps, procedures | **Flowchart** | "workflow", "process", "steps", "procedure" |
| Connections, dependencies, associations | **Relationship Diagram** | "relationship", "connections", "dependencies", "structure" |
| Concept hierarchy, brainstorming | **Mind Map** | "mind map", "concepts", "ideas", "breakdown" |
| System design, components | **Architecture Diagram** | "architecture", "system", "components", "modules" |
| Data flow, transformation processes | **Data Flow Diagram (DFD)** | "data flow", "data processing", "data transformation" |
| Cross-functional processes, actor responsibilities | **Business Flow (Swimlane)** | "business process", "swimlane", "actors", "responsibilities" |
| Object-oriented design, class structures | **Class Diagram** | "class", "inheritance", "OOP", "object model" |
| Interaction sequences, message flows | **Sequence Diagram** | "sequence", "interaction", "messages", "timeline" |
| Database design, entity relationships | **ER Diagram** | "database", "entity", "relationship", "data model" |

### Step 3: Extract Structured Information

**For Flowcharts:**
- 順番に並ぶ step の list
- decision point (あれば)
- start と end point

**For Relationship Diagrams:**
- entity / node (name + optional description)
- entity 間 relationship (`from → to` と label)

**For Mind Maps:**
- central topic
- main branch (推奨 3-6)
- 各 branch の sub-topic (optional)

**For Data Flow Diagrams (DFD):**
- data source と destination (external entity)
- process (data transformation)
- data store (database、file)
- data flow (data movement を示す arrow。左から右、または左上から右下)
- **Important**: process の順序ではなく、data flow だけを表現すること

**For Business Flow (Swimlane):**
- actor / role (department、system、人) - header column として表示
- process lane (各 actor の下に並ぶ縦 lane)
- process box (各 lane 内の activity)
- flow arrow (process box を接続。lane をまたぐ handoff も含む)

**For Class Diagrams:**
- class 名付き class
- visibility (`+`, `-`, `#`) 付き attribute
- visibility と parameter 付き method
- relationship: inheritance (solid line + white triangle), implementation (dashed line + white triangle), association (solid line), dependency (dashed line), aggregation (solid line + white diamond), composition (solid line + filled diamond)
- multiplicity notation (`1`, `0..1`, `1..*`, `*`)

**For Sequence Diagrams:**
- object / actor (上部に横並び)
- lifeline (各 object から下に伸びる縦線)
- message (lifeline 間の水平 arrow)
- synchronous message (solid arrow)、asynchronous message (dashed arrow)
- return value (dashed arrow)
- activation box (実行中の lifeline 上の rectangle)
- 時間は上から下へ流れる

**For ER Diagrams:**
- entity (entity 名の入った rectangle)
- attribute (entity 内に列挙)
- primary key (下線または PK で表示)
- foreign key (FK で表示)
- relationship (entity をつなぐ line)
- cardinality: 1:1 (one-to-one), 1:N (one-to-many), N:M (many-to-many)
- many-to-many relationship 用の junction / associative entity (破線 rectangle)

### Step 4: Generate the Excalidraw JSON

適切な element を使って `.excalidraw` file を作成します。

**Available element types:**
- `rectangle`: entity、step、concept 用の box
- `ellipse`: 強調や terminal 用の別 shape
- `diamond`: decision point
- `arrow`: 方向付き connection
- `text`: label と annotation

**Key properties to set:**
- **Position**: `x`, `y` coordinate
- **Size**: `width`, `height`
- **Style**: `strokeColor`, `backgroundColor`, `fillStyle`
- **Font**: `fontFamily: 5` (Excalifont - **すべての text element で必須**)
- **Text**: label 用 embedded text
- **Connections**: arrow 用 `points` array

**Important**: すべての text element は、見た目を揃えるため `fontFamily: 5` (Excalifont) を使うこと。

### Step 5: Format the Output

完全な Excalidraw file は次の形にします。

```json
{
  "type": "excalidraw",
  "version": 2,
  "source": "https://excalidraw.com",
  "elements": [
    // Array of diagram elements
  ],
  "appState": {
    "viewBackgroundColor": "#ffffff",
    "gridSize": 20
  },
  "files": {}
}
```

### Step 6: Save and Provide Instructions

1. `<descriptive-name>.excalidraw` として保存
2. 開き方を伝える:
   - https://excalidraw.com へアクセス
   - "Open" をクリック、または file を drag-and-drop
   - あるいは Excalidraw VS Code extension を使う

## Best Practices

### Element Count Guidelines

| Diagram Type | Recommended Count | Maximum |
|--------------|-------------------|---------|
| Flowchart steps | 3-10 | 15 |
| Relationship entities | 3-8 | 12 |
| Mind map branches | 4-6 | 8 |
| Mind map sub-topics per branch | 2-4 | 6 |

### Layout Tips

1. **Start positions**: 重要な要素は中央に置き、spacing を一貫させる
2. **Spacing**:
   - Horizontal gap: 要素間 200-300px
   - Vertical gap: row 間 100-150px
3. **Colors**: 一貫した color scheme を使う
   - Primary elements: Light blue (`#a5d8ff`)
   - Secondary elements: Light green (`#b2f2bb`)
   - Important/Central: Yellow (`#ffd43b`)
   - Alerts/Warnings: Light red (`#ffc9c9`)
4. **Text sizing**: 可読性のため 16-24px
5. **Font**: すべての text element に `fontFamily: 5` (Excalifont) を使う
6. **Arrow style**: 単純な flow には straight arrow、複雑な relationship には curved を使う

### Complexity Management

**ユーザー依頼に要素が多すぎる場合:**
- 複数 diagram に分割することを提案する
- まず主要要素に絞る
- 詳細 sub-diagram を後から作る提案をする

**Example response:**
```
"Your request includes 15 components. For clarity, I recommend:
1. High-level architecture diagram (6 main components)
2. Detailed diagram for each subsystem

Would you like me to start with the high-level view?"
```

## Example Prompts and Responses

### Example 1: Simple Flowchart

**User:** "Create a flowchart for user registration"

**Agent generates:**
1. step を抽出: "Enter email" → "Verify email" → "Set password" → "Complete"
2. 4 つの rectangle + 3 つの arrow で flowchart を作成
3. `user-registration-flow.excalidraw` として保存

### Example 2: Relationship Diagram

**User:** "Diagram the relationship between User, Post, and Comment entities"

**Agent generates:**
1. Entity: User, Post, Comment
2. Relationship: User → Post ("creates"), User → Comment ("writes"), Post → Comment ("contains")
3. `user-content-relationships.excalidraw` として保存

### Example 3: Mind Map

**User:** "Mind map about machine learning concepts"

**Agent generates:**
1. 中心: "Machine Learning"
2. Branch: Supervised Learning, Unsupervised Learning, Reinforcement Learning, Deep Learning
3. 各 branch の sub-topic
4. `machine-learning-mindmap.excalidraw` として保存

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Elements overlap | coordinate 間の spacing を広げる |
| Text doesn't fit in boxes | box width を広げるか font size を下げる |
| Too many elements | 複数 diagram に分割する |
| Unclear layout | grid layout (row/column) または radial layout (mind map) を使う |
| Colors inconsistent | element type ごとに先に color palette を決める |

## Advanced Techniques

### Grid Layout (for Relationship Diagrams)
```javascript
const columns = Math.ceil(Math.sqrt(entityCount));
const x = startX + (index % columns) * horizontalGap;
const y = startY + Math.floor(index / columns) * verticalGap;
```

### Radial Layout (for Mind Maps)
```javascript
const angle = (2 * Math.PI * index) / branchCount;
const x = centerX + radius * Math.cos(angle);
const y = centerY + radius * Math.sin(angle);
```

### Auto-generated IDs

timestamp + random string で unique ID を作ります。
```javascript
const id = Date.now().toString(36) + Math.random().toString(36).substr(2);
```

## Output Format

常に次を含めてください。
1. ✅ 完全な `.excalidraw` JSON file
2. 📊 何を作成したかの summary
3. 📝 element count
4. 💡 開き方 / 編集方法

**Example summary:**
```
Created: user-workflow.excalidraw
Type: Flowchart
Elements: 7 rectangles, 6 arrows, 1 title text
Total: 14 elements

To view:
1. Visit https://excalidraw.com
2. Drag and drop user-workflow.excalidraw
3. Or use File → Open in Excalidraw VS Code extension
```

## Validation Checklist

渡す前に確認:
- [ ] すべての element に unique ID がある
- [ ] coordinate が重なりを防いでいる
- [ ] text が読める (`font size 16+`)
- [ ] **すべての text element が `fontFamily: 5` (Excalifont) を使っている**
- [ ] arrow が論理的につながっている
- [ ] color scheme が一貫している
- [ ] file が valid JSON である
- [ ] element count が妥当 (<20、可読性重視)

## Icon Libraries (Optional Enhancement)

専門的な diagram (例: AWS/GCP/Azure architecture diagram) では、Excalidraw の既製 icon library を使えます。基本 shape ではなく、標準化された professional icon を使えるようになります。

### When User Requests Icons

**ユーザーが AWS / cloud architecture diagram や specific icon の利用を求めた場合:**

1. **library があるか確認**: `libraries/<library-name>/reference.md` を探す
2. **library がある場合**: icon を使って進める (下の AI Assistant Workflow 参照)
3. **library がない場合**: 次の setup instruction を返す

   ```
   To use [AWS/GCP/Azure/etc.] architecture icons, please follow these steps:

   1. Visit https://libraries.excalidraw.com/
   2. Search for "[AWS Architecture Icons/etc.]" and download the .excalidrawlib file
   3. Create directory: skills/excalidraw-diagram-generator/libraries/[icon-set-name]/
   4. Place the downloaded file in that directory
   5. Run the splitter script:
      python skills/excalidraw-diagram-generator/scripts/split-excalidraw-library.py skills/excalidraw-diagram-generator/libraries/[icon-set-name]/

   This will split the library into individual icon files for efficient use.
   After setup is complete, I can create your diagram using the actual AWS/cloud icons.

   Alternatively, I can create the diagram now using simple shapes (rectangles, ellipses)
   which you can later replace with icons manually in Excalidraw.
   ```

### User Setup Instructions (Detailed)

**Step 1: Create Library Directory**
```bash
mkdir -p skills/excalidraw-diagram-generator/libraries/aws-architecture-icons
```

**Step 2: Download Library**
- Visit: https://libraries.excalidraw.com/
- 使いたい icon set を検索する (例: "AWS Architecture Icons")
- ダウンロードして `.excalidrawlib` file を取得する
- 例となる category (availability は変わるため、site 上で確認すること):
   - Cloud service icons
   - UI/Material icons
   - Flowchart symbols

**Step 3: Place Library File**
- ダウンロードした file を directory 名に合わせて rename する (例: `aws-architecture-icons.excalidrawlib`)
- Step 1 で作った directory に置く

**Step 4: Run Splitter Script**
```bash
python skills/excalidraw-diagram-generator/scripts/split-excalidraw-library.py skills/excalidraw-diagram-generator/libraries/aws-architecture-icons/
```

**Step 5: Verify Setup**
script 実行後、次の構造があることを確認します。
```
skills/excalidraw-diagram-generator/libraries/aws-architecture-icons/
  aws-architecture-icons.excalidrawlib  (original)
  reference.md                          (generated - icon lookup table)
  icons/                                (generated - individual icon files)
    API-Gateway.json
    CloudFront.json
    EC2.json
    Lambda.json
    RDS.json
    S3.json
    ...
```

### AI Assistant Workflow

**`libraries/` に icon library がある場合:**

**RECOMMENDED APPROACH: Python Script を使う (効率的で信頼できる)**

この repository には、icon integration を自動で処理する Python script が含まれています。

1. **Create base diagram structure**:
   - title、box、region を含む `.excalidraw` file を作る
   - まず canvas と全体構造を確立する

2. **Add icons using Python script**:
   ```bash
   python skills/excalidraw-diagram-generator/scripts/add-icon-to-diagram.py \
     <diagram-path> <icon-name> <x> <y> [--label "Text"] [--library-path PATH]
   ```
   - editor の overwrite issue を避けるため、`.excalidraw.edit` 経由での編集が既定で有効。無効化するには `--no-use-edit-suffix` を渡す。

   **Examples**:
   ```bash
   # Add EC2 icon at position (400, 300) with label
   python scripts/add-icon-to-diagram.py diagram.excalidraw EC2 400 300 --label "Web Server"

   # Add VPC icon at position (200, 150)
   python scripts/add-icon-to-diagram.py diagram.excalidraw VPC 200 150

   # Add icon from different library
   python scripts/add-icon-to-diagram.py diagram.excalidraw Compute-Engine 500 200 \
     --library-path libraries/gcp-icons --label "API Server"
   ```

3. **Add connecting arrows**:
   ```bash
   python skills/excalidraw-diagram-generator/scripts/add-arrow.py \
     <diagram-path> <from-x> <from-y> <to-x> <to-y> [--label "Text"] [--style solid|dashed|dotted] [--color HEX]
   ```
   - editor の overwrite issue を避けるため、`.excalidraw.edit` 経由での編集が既定で有効。無効化するには `--no-use-edit-suffix` を渡す。

   **Examples**:
   ```bash
   # Simple arrow from (300, 250) to (500, 300)
   python scripts/add-arrow.py diagram.excalidraw 300 250 500 300

   # Arrow with label
   python scripts/add-arrow.py diagram.excalidraw 300 250 500 300 --label "HTTPS"

   # Dashed arrow with custom color
   python scripts/add-arrow.py diagram.excalidraw 400 350 600 400 --style dashed --color "#7950f2"
   ```

4. **Workflow summary**:
   ```bash
   # Step 1: Create base diagram with title and structure
   # (Create .excalidraw file with initial elements)

   # Step 2: Add icons with labels
   python scripts/add-icon-to-diagram.py my-diagram.excalidraw "Internet-gateway" 200 150 --label "Internet Gateway"
   python scripts/add-icon-to-diagram.py my-diagram.excalidraw VPC 250 250
   python scripts/add-icon-to-diagram.py my-diagram.excalidraw ELB 350 300 --label "Load Balancer"
   python scripts/add-icon-to-diagram.py my-diagram.excalidraw EC2 450 350 --label "EC2 Instance"
   python scripts/add-icon-to-diagram.py my-diagram.excalidraw RDS 550 400 --label "Database"

   # Step 3: Add connecting arrows
   python scripts/add-arrow.py my-diagram.excalidraw 250 200 300 250  # Internet → VPC
   python scripts/add-arrow.py my-diagram.excalidraw 300 300 400 300  # VPC → ELB
   python scripts/add-arrow.py my-diagram.excalidraw 400 330 500 350  # ELB → EC2
   python scripts/add-arrow.py my-diagram.excalidraw 500 380 600 400  # EC2 → RDS
   ```

**Benefits of Python Script Approach**:
- ✅ **No token consumption**: icon JSON data (各 200-1000 行) を AI context に入れずに済む
- ✅ **Accurate transformations**: coordinate 計算を決定的に処理できる
- ✅ **ID management**: 自動 UUID 生成で conflict を防げる
- ✅ **Reliable**: coordinate miscalculation や ID collision の risk がない
- ✅ **Fast**: 直接 file を操作するため parsing overhead が少ない
- ✅ **Reusable**: 提供された任意の Excalidraw library で使える

**ALTERNATIVE: Manual Icon Integration (Not Recommended)**

Python script が使えない場合だけ使ってください。

1. **Check for libraries**:
   ```
   List directory: skills/excalidraw-diagram-generator/libraries/
   Look for subdirectories containing reference.md files
   ```

2. **Read reference.md**:
   ```
   Open: libraries/<library-name>/reference.md
   This is lightweight (typically <300 lines) and lists all available icons
   ```

3. **Find relevant icons**:
   ```
   Search the reference.md table for icon names matching diagram needs
   Example: For AWS diagram with EC2, S3, Lambda → Find "EC2", "S3", "Lambda" in table
   ```

4. **Load specific icon data** (WARNING: Large files):
   ```
   Read ONLY the needed icon files:
   - libraries/aws-architecture-icons/icons/EC2.json (200-300 lines)
   - libraries/aws-architecture-icons/icons/S3.json (200-300 lines)
   - libraries/aws-architecture-icons/icons/Lambda.json (200-300 lines)
   Note: Each icon file is 200-1000 lines - this consumes significant tokens
   ```

5. **Extract and transform elements**:
   ```
   Each icon JSON contains an "elements" array
   Calculate bounding box (min_x, min_y, max_x, max_y)
   Apply offset to all x/y coordinates
   Generate new unique IDs for all elements
   Update groupIds references
   Copy transformed elements into your diagram
   ```

6. **Position icons and add connections**:
   ```
   Adjust x/y coordinates to position icons correctly in the diagram
   Update IDs to ensure uniqueness across diagram
   Add connecting arrows and labels as needed
   ```

**Manual Integration Challenges**:
- ⚠️ token 消費が大きい (200-1000 行 / icon × icon 数)
- ⚠️ coordinate transformation 計算が複雑
- ⚠️ careful に扱わないと ID collision の risk がある
- ⚠️ icon 数が多い diagram では時間がかかる

### Example: Creating AWS Diagram with Icons

**Request**: "Create an AWS architecture diagram with Internet Gateway, VPC, ELB, EC2, and RDS"

**Recommended Workflow (using Python scripts)**:
**Request**: "Create an AWS architecture diagram with Internet Gateway, VPC, ELB, EC2, and RDS"

**Recommended Workflow (using Python scripts)**:

```bash
# Step 1: Create base diagram file with title
# Create my-aws-diagram.excalidraw with basic structure (title, etc.)

# Step 2: Check icon availability
# Read: libraries/aws-architecture-icons/reference.md
# Confirm icons exist: Internet-gateway, VPC, ELB, EC2, RDS

# Step 3: Add icons with Python script
python scripts/add-icon-to-diagram.py my-aws-diagram.excalidraw "Internet-gateway" 150 100 --label "Internet Gateway"
python scripts/add-icon-to-diagram.py my-aws-diagram.excalidraw VPC 200 200
python scripts/add-icon-to-diagram.py my-aws-diagram.excalidraw ELB 350 250 --label "Load Balancer"
python scripts/add-icon-to-diagram.py my-aws-diagram.excalidraw EC2 500 300 --label "Web Server"
python scripts/add-icon-to-diagram.py my-aws-diagram.excalidraw RDS 650 350 --label "Database"

# Step 4: Add connecting arrows
python scripts/add-arrow.py my-aws-diagram.excalidraw 200 150 250 200  # Internet → VPC
python scripts/add-arrow.py my-aws-diagram.excalidraw 265 230 350 250  # VPC → ELB
python scripts/add-arrow.py my-aws-diagram.excalidraw 415 280 500 300  # ELB → EC2
python scripts/add-arrow.py my-aws-diagram.excalidraw 565 330 650 350 --label "SQL" --style dashed

# Result: Complete diagram with professional AWS icons, labels, and connections
```

**Benefits**:
- 手計算で coordinate を出さなくてよい
- icon data に token を使わない
- 結果が決定的で信頼できる
- 位置調整や反復がしやすい

**Alternative Workflow (manual, if scripts unavailable)**:
1. Check: `libraries/aws-architecture-icons/reference.md` exists → Yes
2. Read reference.md → Internet-gateway, VPC, ELB, EC2, RDS の entry を探す
3. Load:
   - `icons/Internet-gateway.json` (298 lines)
   - `icons/VPC.json` (550 lines)
   - `icons/ELB.json` (363 lines)
   - `icons/EC2.json` (231 lines)
   - `icons/RDS.json` (similar size)
   **Total: ~2000+ lines of JSON to process**
4. 各 JSON から element を抽出
5. 各 icon の bounding box と offset を計算
6. 配置用にすべての coordinate (`x`, `y`) を変換
7. すべての element に unique ID を生成
8. data flow を示す arrow を追加
9. text label を追加
10. 最終 `.excalidraw` file を生成

**Challenges with manual approach**:
- 高い token 消費 (~2000-5000 行)
- 複雑な coordinate math
- ID conflict の risk

### Supported Icon Libraries (Examples — verify availability)

- この workflow は、提供された任意の valid `.excalidrawlib` file で動作します。
- https://libraries.excalidraw.com/ で見つかる可能性のある library category の例:
   - Cloud service icons
   - Kubernetes / infrastructure icons
   - UI / Material icons
   - Flowchart / diagram symbols
   - Network diagram icons
- availability や naming は変わるため、正確な library 名は site で確認してください。

### Fallback: No Icons Available

**icon library が未 setup の場合:**
- 基本 shape (rectangle、ellipse、arrow) で diagram を作る
- color coding と text label で component を区別する
- 後で icon を追加するか、将来のために library を setup できることをユーザーに伝える
- 見た目はやや簡素でも、diagram 自体は十分機能的で分かりやすい

## References

bundled reference:
- `references/excalidraw-schema.md` - 完全な Excalidraw JSON schema
- `references/element-types.md` - element type の詳細 specification
- `templates/flowchart-template.json` - 基本 flowchart starter
- `templates/relationship-template.json` - relationship diagram starter
- `templates/mindmap-template.json` - mind map starter
- `scripts/split-excalidraw-library.py` - `.excalidrawlib` file を分割する tool
- `scripts/README.md` - library tool の documentation
- `scripts/.gitignore` - local Python artifact の commit を防ぐ

## Limitations

- 複雑な curve は単純な直線 / 基本 curve に簡略化される
- hand-drawn roughness は default (`1`) に固定
- auto-generation では embedded image をサポートしない
- 推奨 element 数の上限: 1 diagram あたり 20
- automatic collision detection はない (spacing guideline を使うこと)

## Future Enhancements

今後の改善候補:
- Auto-layout optimization algorithm
- Mermaid / PlantUML syntax からの import
- Template library の拡張
- 生成後の interactive editing
