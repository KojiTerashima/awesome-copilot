# Excalidraw Element Types Guide

各 Excalidraw element type の詳細 specification、見た目の例、use case をまとめた guide です。

## Element Type Overview

| Type | Visual | Primary Use | Text Support |
|------|--------|-------------|--------------|
| `rectangle` | □ | box、container、process step | ✅ Yes |
| `ellipse` | ○ | 強調、terminal、state | ✅ Yes |
| `diamond` | ◇ | decision point、choice | ✅ Yes |
| `arrow` | → | directional flow、relationship | ❌ No (separate text を使う) |
| `line` | — | connection、divider | ❌ No |
| `text` | A | label、annotation、title | ✅ (それ自体が目的) |

---

## Rectangle

**Best for:** process step、entity、data store、component

### Properties

```typescript
{
  type: "rectangle",
  roundness: { type: 3 },  // Rounded corners
  text: "Step Name",       // Optional embedded text
  fontSize: 20,
  textAlign: "center",
  verticalAlign: "middle"
}
```

### Use Cases

| Scenario | Configuration |
|----------|---------------|
| **Process step** | Green background (`#b2f2bb`)、centered text |
| **Entity/Object** | Blue background (`#a5d8ff`)、medium size |
| **System component** | Light color、descriptive text |
| **Data store** | Gray/white、database らしい label |

### Size Guidelines

| Content | Width | Height |
|---------|-------|--------|
| Single word | 120-150px | 60-80px |
| Short phrase (2-4 words) | 180-220px | 80-100px |
| Sentence | 250-300px | 100-120px |

### Example

```json
{
  "type": "rectangle",
  "x": 100,
  "y": 100,
  "width": 200,
  "height": 80,
  "backgroundColor": "#b2f2bb",
  "text": "Validate Input",
  "fontSize": 20,
  "textAlign": "center",
  "verticalAlign": "middle",
  "roundness": { "type": 3 }
}
```

---

## Ellipse

**Best for:** start/end point、state、強調 circle

### Properties

```typescript
{
  type: "ellipse",
  text: "Start",
  fontSize: 18,
  textAlign: "center",
  verticalAlign: "middle"
}
```

### Use Cases

| Scenario | Configuration |
|----------|---------------|
| **Flow start** | Light green、"Start" text |
| **Flow end** | Light red、"End" text |
| **State** | soft color、state 名 |
| **Highlight** | bright color、強調 text |

### Size Guidelines

円形 shape では `width === height` を使います。

| Content | Diameter |
|---------|----------|
| Icon/Symbol | 60-80px |
| Short text | 100-120px |
| Longer text | 150-180px |

### Example

```json
{
  "type": "ellipse",
  "x": 100,
  "y": 100,
  "width": 120,
  "height": 120,
  "backgroundColor": "#d0f0c0",
  "text": "Start",
  "fontSize": 18,
  "textAlign": "center",
  "verticalAlign": "middle"
}
```

---

## Diamond

**Best for:** decision point、conditional branch

### Properties

```typescript
{
  type: "diamond",
  text: "Valid?",
  fontSize: 18,
  textAlign: "center",
  verticalAlign: "middle"
}
```

### Use Cases

| Scenario | Text Example |
|----------|--------------|
| **Yes/No decision** | "Is Valid?", "Exists?" |
| **Multiple choice** | "Type?", "Status?" |
| **Conditional** | "Score > 50?" |

### Size Guidelines

diamond は同じ text でも rectangle より余分な space が必要です。

| Content | Width | Height |
|---------|-------|--------|
| Yes/No | 120-140px | 120-140px |
| Short question | 160-180px | 160-180px |
| Longer question | 200-220px | 200-220px |

### Example

```json
{
  "type": "diamond",
  "x": 100,
  "y": 100,
  "width": 150,
  "height": 150,
  "backgroundColor": "#ffe4a3",
  "text": "Valid?",
  "fontSize": 18,
  "textAlign": "center",
  "verticalAlign": "middle"
}
```

---

## Arrow

**Best for:** flow direction、relationship、dependency

### Properties

```typescript
{
  type: "arrow",
  points: [[0, 0], [endX, endY]],  // Relative coordinates
  roundness: { type: 2 },          // Curved
  startBinding: null,              // Or { elementId, focus, gap }
  endBinding: null
}
```

### Arrow Directions

#### Horizontal (Left to Right)

```json
{
  "x": 100,
  "y": 150,
  "width": 200,
  "height": 0,
  "points": [[0, 0], [200, 0]]
}
```

#### Vertical (Top to Bottom)

```json
{
  "x": 200,
  "y": 100,
  "width": 0,
  "height": 150,
  "points": [[0, 0], [0, 150]]
}
```

#### Diagonal

```json
{
  "x": 100,
  "y": 100,
  "width": 200,
  "height": 150,
  "points": [[0, 0], [200, 150]]
}
```

### Arrow Styles

| Style | `strokeStyle` | `strokeWidth` | Use Case |
|-------|---------------|---------------|----------|
| **Normal flow** | `"solid"` | 2 | 標準 connection |
| **Optional/Weak** | `"dashed"` | 2 | optional path |
| **Important** | `"solid"` | 3-4 | 強調した flow |
| **Dotted** | `"dotted"` | 2 | indirect relationship |

### Adding Arrow Labels

arrow midpoint 付近に separate text element を置きます。

```json
[
  {
    "type": "arrow",
    "id": "arrow1",
    "x": 100,
    "y": 150,
    "points": [[0, 0], [200, 0]]
  },
  {
    "type": "text",
    "x": 180,      // Near midpoint
    "y": 130,      // Above arrow
    "text": "sends",
    "fontSize": 14
  }
]
```

---

## Line

**Best for:** 非方向 connection、divider、border

### Properties

```typescript
{
  type: "line",
  points: [[0, 0], [x2, y2], [x3, y3], ...],
  roundness: null  // Or { type: 2 } for curved
}
```

### Use Cases

| Scenario | Configuration |
|----------|---------------|
| **Divider** | horizontal、thin stroke |
| **Border** | closed path (polygon) |
| **Connection** | multi-point path |
| **Underline** | short horizontal line |

### Multi-Point Line Example

```json
{
  "type": "line",
  "x": 100,
  "y": 100,
  "points": [
    [0, 0],
    [100, 50],
    [200, 0]
  ]
}
```

---

## Text

**Best for:** label、title、annotation、standalone text

### Properties

```typescript
{
  type: "text",
  text: "Label text",
  fontSize: 20,
  fontFamily: 1,        // 1=Virgil, 2=Helvetica, 3=Cascadia
  textAlign: "left",
  verticalAlign: "top"
}
```

### Font Sizes by Purpose

| Purpose | Font Size |
|---------|-----------|
| **Main title** | 28-36 |
| **Section header** | 24-28 |
| **Element label** | 18-22 |
| **Annotation** | 14-16 |
| **Small note** | 12-14 |

### Width/Height Calculation

```javascript
// Approximate width
const width = text.length * fontSize * 0.6;

// Approximate height (single line)
const height = fontSize * 1.2;

// Multi-line
const lines = text.split('\n').length;
const height = fontSize * 1.2 * lines;
```

### Text Positioning

| Position | textAlign | verticalAlign | Use Case |
|----------|-----------|---------------|----------|
| **Top-left** | `"left"` | `"top"` | default label |
| **Centered** | `"center"` | `"middle"` | title |
| **Bottom-right** | `"right"` | `"bottom"` | footnote |

### Example: Title

```json
{
  "type": "text",
  "x": 100,
  "y": 50,
  "width": 400,
  "height": 40,
  "text": "System Architecture",
  "fontSize": 32,
  "fontFamily": 2,
  "textAlign": "center",
  "verticalAlign": "top"
}
```

### Example: Annotation

```json
{
  "type": "text",
  "x": 150,
  "y": 200,
  "width": 100,
  "height": 20,
  "text": "User input",
  "fontSize": 14,
  "fontFamily": 1,
  "textAlign": "left",
  "verticalAlign": "top"
}
```

---

## Combining Elements

### Pattern: Labeled Box

```json
[
  {
    "type": "rectangle",
    "id": "box1",
    "x": 100,
    "y": 100,
    "width": 200,
    "height": 100,
    "text": "Component",
    "textAlign": "center",
    "verticalAlign": "middle"
  }
]
```

### Pattern: Connected Boxes

```json
[
  {
    "type": "rectangle",
    "id": "box1",
    "x": 100,
    "y": 100,
    "width": 180,
    "height": 80,
    "text": "Source"
  },
  {
    "type": "rectangle",
    "id": "box2",
    "x": 400,
    "y": 100,
    "width": 180,
    "height": 80,
    "text": "Target"
  },
  {
    "type": "arrow",
    "id": "arrow1",
    "x": 280,
    "y": 140,
    "width": 120,
    "height": 0,
    "points": [[0, 0], [120, 0]]
  }
]
```
