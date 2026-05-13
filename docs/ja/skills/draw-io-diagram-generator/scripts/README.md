# draw-io Scripts

cxp-bu-order-ms project で `.drawio` diagram file を扱うための utility script 集です。

## Requirements

- Python 3.8+
- 外部 dependency は不要 (standard library のみ使用: `xml.etree.ElementTree`, `argparse`, `json`, `sys`, `pathlib`)

## Scripts

### `validate-drawio.py`

`.drawio` file の XML 構造が、必要な制約を満たしているか検証します。

**Usage**

```bash
python scripts/validate-drawio.py <path-to-diagram.drawio>
```

**Examples**

```bash
# 1 file を検証
python scripts/validate-drawio.py docs/architecture.drawio

# directory 内のすべての drawio file を検証
for f in docs/**/*.drawio; do python scripts/validate-drawio.py "$f"; done
```

**Checks performed**

| Check | Description |
|-------|-------------|
| Root cells | すべての diagram page に id="0" と id="1" cell があることを確認 |
| Unique IDs | diagram 内のすべての `mxCell` id が一意であることを確認 |
| Edge connectivity | すべての edge が、既存 cell を指す有効な `source` / `target` attribute を持つことを確認 |
| Geometry | すべての vertex cell が `mxGeometry` child element を持つことを確認 |
| Parent chain | 各 cell の `parent` attribute が既存 cell id を参照していることを確認 |
| XML well-formedness | file が妥当な XML であることを確認 |

**Exit codes**

- `0` — 検証成功
- `1` — 1 件以上の検証エラーあり (error は stdout に出力)

---

### `add-shape.py`

既存 `.drawio` diagram file に新しい shape (vertex cell) を追加します。

**Usage**

```bash
python scripts/add-shape.py <diagram.drawio> <label> <x> <y> [options]
```

**Arguments**

| Argument | Required | Description |
|----------|----------|-------------|
| `diagram` | Yes | `.drawio` file への path |
| `label` | Yes | 新しい shape に付ける text label |
| `x` | Yes | X coordinate (左上からの pixel) |
| `y` | Yes | Y coordinate (左上からの pixel) |

**Options**

| Option | Default | Description |
|--------|---------|-------------|
| `--width` | `120` | shape width in pixels |
| `--height` | `60` | shape height in pixels |
| `--style` | `"rounded=1;whiteSpace=wrap;html=1;"` | draw.io style string |
| `--diagram-index` | `0` | diagram page の index (0-based) |
| `--dry-run` | false | file を変更せず、新しい cell XML を表示 |

**Examples**

```bash
# 基本の rounded box を追加
python scripts/add-shape.py docs/flowchart.drawio "New Step" 400 300

# custom style の shape を追加
python scripts/add-shape.py docs/flowchart.drawio "Decision" 400 400 \
  --width 160 --height 80 \
  --style "rhombus;whiteSpace=wrap;html=1;fillColor=#fff2cc;strokeColor=#d6b656;"

# 書き込まず preview
python scripts/add-shape.py docs/architecture.drawio "Service X" 600 200 --dry-run
```

**Output**

成功時は新しい cell id を出力します:
```
Added shape id="auto_abc123" to page 0 of docs/flowchart.drawio
```

---

## Common Workflows

### Commit 前に検証

```bash
# すべての diagram を検証
find . -name "*.drawio" -not -path "*/node_modules/*" | \
  xargs -I{} python scripts/validate-drawio.py {}
```

### placeholder node を素早く追加

```bash
python scripts/add-shape.py docs/architecture.drawio "TODO: Service" 800 400 \
  --style "rounded=1;whiteSpace=wrap;html=1;fillColor=#f8cecc;strokeColor=#b85450;"
```

### template が妥当か確認

```bash
python scripts/validate-drawio.py .github/skills/draw-io-diagram-generator/templates/flowchart.drawio
```
