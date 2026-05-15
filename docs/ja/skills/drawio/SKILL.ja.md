---
name: drawio
description: Generate draw.io diagrams as .drawio files and export to PNG/SVG/PDF with embedded XML
---

# Draw.io 図スキル

draw.io 図をネイティブ `.drawio` ファイルとして生成し、Word ドキュメントに埋め込むことができる PNG 画像にエクスポートします。

## 図の作成方法

1. **要求された図の `mxGraphModel` 形式でdraw.io XMLを生成します**
2. ファイル作成/編集ツールを使用して、**XML** を `.drawio` ファイルに書き込みます
3. **バンドルされたエクスポート スクリプトを使用して PNG にエクスポート**

## バンドルされたエクスポート スクリプト

このスキルには、2 つのレンダリング バックエンドを備えた Node.js エクスポート スクリプトである `drawio-to-png.mjs` が含まれています。

1. **draw.io CLI** (ピクセルパーフェクト、最速) —draw.io デスクトップがインストールされている場合に自動的に使用されます
2. **ヘッドレスブラウザの公式draw.ioビューア** (ピクセルパーフェクト、Chromium/Edgeが必要) — CLIが利用できない場合のフォールバック

### 使用法
```bash
# Install dependencies (one-time, from the scripts folder)
cd skills/drawio/scripts && npm install

# Export a single diagram
node skills/drawio/scripts/drawio-to-png.mjs <input.drawio> [output.png]

# Export all .drawio files in a directory
node skills/drawio/scripts/drawio-to-png.mjs --dir <directory>

# Force a specific renderer
node skills/drawio/scripts/drawio-to-png.mjs --renderer=cli|viewer|auto <input.drawio>
```

### スキルフォルダの内容

|ファイル |目的 |
|-----|----------|
| `SKILL.md` |この命令ファイル |
| `scripts/drawio-to-png.mjs` | Node.js エクスポート スクリプト (CLI + ブラウザー フォールバック) |
| `scripts/package.json` |依存関係 (`puppeteer-core`) |

## サポートされているエクスポート形式

|フォーマット | XML を埋め込む |メモ |
|----------|-----------|----------|
| `png` |はい |どこでも表示可能、draw.io で編集可能 |
| `svg` |はい |スケーラブル、draw.io で編集可能 |
| `pdf` |はい |印刷可能、draw.io で編集可能 |

## Draw.io XML スタイル規則

一貫性のあるプロフェッショナルな図を作成するには、次のスタイルを使用します。
```xml
<!-- Primary service (highlighted) -->
<mxCell style="rounded=1;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;strokeWidth=2;arcSize=12;shadow=1;" />

<!-- External system -->
<mxCell style="rounded=1;whiteSpace=wrap;html=1;fillColor=#f5f5f5;strokeColor=#666666;" />

<!-- Success/processing stage -->
<mxCell style="rounded=1;whiteSpace=wrap;html=1;fillColor=#d5e8d4;strokeColor=#82b366;" />

<!-- Warning/quality gate -->
<mxCell style="rounded=1;whiteSpace=wrap;html=1;fillColor=#fff2cc;strokeColor=#d6b656;" />

<!-- Error/failure path -->
<mxCell style="rounded=1;whiteSpace=wrap;html=1;fillColor=#f8cecc;strokeColor=#b85450;" />

<!-- Data store (cylinder) -->
<mxCell style="shape=cylinder3;whiteSpace=wrap;html=1;fillColor=#fff2cc;strokeColor=#d6b656;" />

<!-- Arrow -->
<mxCell style="edgeStyle=orthogonalEdgeStyle;rounded=1;strokeColor=#6c8ebf;strokeWidth=2;" />
```

## draw.io CLIの場所

最初に `drawio` を試してから (PATH 上で動作する)、その後フォールバックします。

- **Windows**: `"C:\Program Files\draw.io\draw.io.exe"`
- **macOS**: `/Applications/draw.io.app/Contents/MacOS/draw.io`
- **Linux**: `drawio` (snap/apt/ flatpak 経由)

### CLIエクスポートコマンド
```bash
drawio -x -f png -e -b 10 -o <output.png> <input.drawio>
```

フラグ: `-x` (エクスポート)、`-f` (フォーマット)、`-e` (埋め込み図 XML)、`-b` (境界線)、`-o` (出力パス)。