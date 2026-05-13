# Excalidraw JSON Schema Reference

この document は、diagram 生成用の Excalidraw `.excalidraw` file 構造を説明します。

## Top-Level Structure

```typescript
interface ExcalidrawFile {
  type: "excalidraw";
  version: number;           // Always 2
  source: string;            // "https://excalidraw.com"
  elements: ExcalidrawElement[];
  appState: AppState;
  files: Record<string, any>; // Usually empty {}
}
```

## AppState

```typescript
interface AppState {
  viewBackgroundColor: string; // Hex color, e.g., "#ffffff"
  gridSize: number;            // Typically 20
}
```

## ExcalidrawElement Base Properties

すべての element は次の共通 property を持ちます。

```typescript
interface BaseElement {
  id: string;                  // Unique identifier
  type: ElementType;           // See Element Types below
  x: number;                   // X coordinate (pixels from top-left)
  y: number;                   // Y coordinate (pixels from top-left)
  width: number;               // Width in pixels
  height: number;              // Height in pixels
  angle: number;               // Rotation angle in radians (usually 0)
  strokeColor: string;         // Hex color, e.g., "#1e1e1e"
  backgroundColor: string;     // Hex color or "transparent"
  fillStyle: "solid" | "hachure" | "cross-hatch";
  strokeWidth: number;         // 1-4 typically
  strokeStyle: "solid" | "dashed" | "dotted";
  roughness: number;           // 0-2, hand-drawn effect の強さ (1 = default)
  opacity: number;             // 0-100
  groupIds: string[];          // この element が属する group の ID
  frameId: null;               // Usually null
  index: string;               // Stacking order identifier
  roundness: Roundness | null;
  seed: number;                // Deterministic rendering のための random seed
  version: number;             // Element version (編集ごとに増やす)
  versionNonce: number;        // 編集時に変わる random number
  isDeleted: boolean;          // false にする
  boundElements: any;          // Usually null
  updated: number;             // Timestamp in milliseconds
  link: null;                  // External link (usually null)
  locked: boolean;             // element を lock するか
}
```

## Element Types

### Rectangle

```typescript
interface RectangleElement extends BaseElement {
  type: "rectangle";
  roundness: { type: 3 };      // 3 = rounded corners
  text?: string;               // Optional text inside
  fontSize?: number;           // Font size (16-32 typical)
  fontFamily?: number;         // 1 = Virgil, 2 = Helvetica, 3 = Cascadia
  textAlign?: "left" | "center" | "right";
  verticalAlign?: "top" | "middle" | "bottom";
}
```

**Example:**
```json
{
  "id": "rect1",
  "type": "rectangle",
  "x": 100,
  "y": 100,
  "width": 200,
  "height": 100,
  "strokeColor": "#1e1e1e",
  "backgroundColor": "#a5d8ff",
  "text": "My Box",
  "fontSize": 20,
  "textAlign": "center",
  "verticalAlign": "middle",
  "roundness": { "type": 3 }
}
```

### Ellipse

```typescript
interface EllipseElement extends BaseElement {
  type: "ellipse";
  text?: string;
  fontSize?: number;
  fontFamily?: number;
  textAlign?: "left" | "center" | "right";
  verticalAlign?: "top" | "middle" | "bottom";
}
```

### Diamond

```typescript
interface DiamondElement extends BaseElement {
  type: "diamond";
  text?: string;
  fontSize?: number;
  fontFamily?: number;
  textAlign?: "left" | "center" | "right";
  verticalAlign?: "top" | "middle" | "bottom";
}
```

### Arrow

```typescript
interface ArrowElement extends BaseElement {
  type: "arrow";
  points: [number, number][];  // element 基準の [x, y] coordinate array
  startBinding: Binding | null;
  endBinding: Binding | null;
  roundness: { type: 2 };      // 2 = curved arrow
}
```

**Example:**
```json
{
  "id": "arrow1",
  "type": "arrow",
  "x": 100,
  "y": 100,
  "width": 200,
  "height": 0,
  "points": [
    [0, 0],
    [200, 0]
  ],
  "roundness": { "type": 2 },
  "startBinding": null,
  "endBinding": null
}
```

**Points explanation:**
- 最初の point `[0, 0]` は `(x, y)` 基準
- それ以降の point は最初の point に対する相対値
- 水平の straight arrow: `[[0, 0], [width, 0]]`
- 垂直の straight arrow: `[[0, 0], [0, height]]`

### Line

```typescript
interface LineElement extends BaseElement {
  type: "line";
  points: [number, number][];
  startBinding: Binding | null;
  endBinding: Binding | null;
  roundness: { type: 2 } | null;
}
```

### Text

```typescript
interface TextElement extends BaseElement {
  type: "text";
  text: string;
  fontSize: number;
  fontFamily: number;          // 1-3
  textAlign: "left" | "center" | "right";
  verticalAlign: "top" | "middle" | "bottom";
  roundness: null;             // Text has no roundness
}
```

**Example:**
```json
{
  "id": "text1",
  "type": "text",
  "x": 100,
  "y": 100,
  "width": 150,
  "height": 25,
  "text": "Hello World",
  "fontSize": 20,
  "fontFamily": 1,
  "textAlign": "left",
  "verticalAlign": "top",
  "roundness": null
}
```

**Width/Height calculation:**
- Width ≈ `text.length * fontSize * 0.6`
- Height ≈ `fontSize * 1.2 * numberOfLines`

## Bindings

binding は arrow を shape に接続します。

```typescript
interface Binding {
  elementId: string;           // 接続先 element の ID
  focus: number;               // -1 〜 1、edge 上の位置
  gap: number;                 // element edge からの距離
}
```

## Common Colors

| Color Name | Hex Code | Use Case |
|------------|----------|----------|
| Black | `#1e1e1e` | default stroke |
| Light Blue | `#a5d8ff` | primary entity |
| Light Green | `#b2f2bb` | process step |
| Yellow | `#ffd43b` | important / central |
| Light Red | `#ffc9c9` | warning / error |
| Cyan | `#96f2d7` | secondary item |
| Transparent | `transparent` | fill なし |
| White | `#ffffff` | background |

## ID Generation

ID は unique string にします。よく使う pattern:

```javascript
// Timestamp-based
const id = Date.now().toString(36) + Math.random().toString(36).substr(2);

// Sequential
const id = "element-" + counter++;

// Descriptive
const id = "step-1", "entity-user", "arrow-1-to-2";
```

## Seed Generation

seed は hand-drawn effect の deterministic randomness に使います。

```javascript
const seed = Math.floor(Math.random() * 2147483647);
```

## Version and VersionNonce

```javascript
const version = 1;  // 編集時に increment
const versionNonce = Math.floor(Math.random() * 2147483647);
```

## Coordinate System

- origin `(0, 0)` は左上
- X は右に増える
- Y は下に増える
- 単位はすべて pixel

## Recommended Spacing

| Context | Spacing |
|---------|---------|
| Horizontal gap between elements | 200-300px |
| Vertical gap between rows | 100-150px |
| Minimum margin from edge | 50px |
| Arrow-to-box clearance | 20-30px |

## Font Families

| ID | Name | Description |
|----|------|-------------|
| 1 | Virgil | hand-drawn style (default) |
| 2 | Helvetica | clean sans-serif |
| 3 | Cascadia | monospace |

## Validation Rules

✅ **Required:**
- すべての ID が unique であること
- `type` が実際の element type と一致すること
- `version` が 1 以上の integer であること
- `opacity` が 0-100 であること

⚠️ **Recommended:**
- 一貫性のため `roughness` は 1 に保つ
- 見やすさのため `strokeWidth` は 2 を使う
- `isDeleted` は `false`
- `locked` は `false`
- `frameId`, `boundElements`, `link` は `null` にする

## Complete Minimal Example

```json
{
  "type": "excalidraw",
  "version": 2,
  "source": "https://excalidraw.com",
  "elements": [
    {
      "id": "box1",
      "type": "rectangle",
      "x": 100,
      "y": 100,
      "width": 200,
      "height": 100,
      "angle": 0,
      "strokeColor": "#1e1e1e",
      "backgroundColor": "#a5d8ff",
      "fillStyle": "solid",
      "strokeWidth": 2,
      "strokeStyle": "solid",
      "roughness": 1,
      "opacity": 100,
      "groupIds": [],
      "frameId": null,
      "index": "a0",
      "roundness": { "type": 3 },
      "seed": 1234567890,
      "version": 1,
      "versionNonce": 987654321,
      "isDeleted": false,
      "boundElements": null,
      "updated": 1706659200000,
      "link": null,
      "locked": false,
      "text": "Hello",
      "fontSize": 20,
      "fontFamily": 1,
      "textAlign": "center",
      "verticalAlign": "middle"
    }
  ],
  "appState": {
    "viewBackgroundColor": "#ffffff",
    "gridSize": 20
  },
  "files": {}
}
```
