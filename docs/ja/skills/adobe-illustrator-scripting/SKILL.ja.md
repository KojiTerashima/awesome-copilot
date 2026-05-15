---
name: adobe-illustrator-scripting
description: 'ExtendScript (JavaScript/JSX) を使用して、Adobe Illustrator オートメーション スクリプトを作成、デバッグ、最適化します。ドキュメント、レイヤー、パス、テキスト フレーム、カラー、シンボル、アートボード、または Illustrator DOM オブジェクトを操作するスクリプトを作成または変更するときに使用します。完全な JavaScript オブジェクト モデル、座標系、測定単位、エクスポート ワークフロー、およびスクリプトのベスト プラクティスをカバーします。'
---

# Adobe Illustrator スクリプト

ExtendScript (JavaScript/JSX) を通じて Adob​​e Illustrator を自動化するための専門家によるガイダンス。このスキルは、Illustrator のスクリプト オブジェクト モデル、すべての主要な API オブジェクト、コード パターン、および製品品質の `.jsx` スクリプトを作成するためのベスト プラクティスをカバーします。

## バンドルされたアセット

- [`references/object-model-quick-reference.md`](references/object-model-quick-reference.md): これは、スクリプトの作成またはデバッグ中に、Illustrator スクリプト オブジェクト モデル、一般的なドキュメントおよびページ項目タイプ、および関連する DOM 概念のクイックルックアップとして使用します。
- `scripts/`: ドキュメント操作、エクスポート、バッチ処理、DOM の使用などの一般的なタスクの開始点または実装パターンとして使用できる、Illustrator 自動化スクリプトの例が含まれています。機能する JSX パターンが必要な場合、またはデバッグ中に動作を比較したい場合は、これらの例を確認して調整してください。
## このスキルをいつ使用するか

- 新しい Illustrator 自動化スクリプトの作成 (`.jsx` または `.js` ファイル)
- 既存の Illustrator ExtendScript コードのデバッグまたは修正
- ドキュメント、レイヤー、ページアイテム、パス、テキスト、またはカラーをプログラムで操作する
- Illustrator ファイルのバッチ処理またはデータからのアートワークの生成
- ドキュメントをさまざまな形式 (PDF、SVG、PNG、EPS など) にエクスポートします。
- Illustrator DOM の操作 (アプリケーション、ドキュメント、レイヤー、PathItem、TextFrame など)
- 変数とデータセットを使用したデータ駆動型グラフィックスの作成
- スクリプト化された印刷オプションによる印刷ワークフローの自動化

## 前提条件

- Adobe Illustrator CC以降がインストールされている
- JavaScript の基本的な知識 (ExtendScript は Adob​​e 拡張機能を備えた ES3 ベースです)
- スクリプトは、[ファイル] > [スクリプト] > [その他のスクリプト]、[スクリプト] メニューから実行するか、[起動スクリプト] フォルダーに配置して実行されます。
- ExtendScript Toolkit (ESTK) または任意のテキスト エディタを使用して、`.jsx` ファイルを作成できます。

## スクリプト環境

### 言語とファイル拡張子

|言語 |拡張子 |プラットフォーム |
|---|---|---|
|拡張スクリプト/JavaScript | `.jsx`、`.js` | Windows、macOS |
|アップルスクリプト | `.scpt` | macOS のみ |
| VBスクリプト | `.vbs` | Windows のみ |

**このスキルは、クロスプラットフォームで最も広く使用されているオプションとして ExtendScript/JavaScript** に焦点を当てています。

### スクリプトの実行

- **スクリプト メニュー**: [ファイル] > [スクリプト] には、アプリケーション スクリプト フォルダーのスクリプトがリストされます。
- **その他のスクリプト**: `.jsx` ファイルを参照して実行するには、[ファイル] > [スクリプト] > [その他のスクリプト]
- **スタートアップ スクリプト**: 起動時に自動的に実行されるように、スクリプトをスタートアップ スクリプト フォルダーに配置します。
- **ターゲット ディレクティブ**: ESTK または外部ツールから実行する場合は `#target illustrator` でスクリプトを開始します
- **`#targetengine` ディレクティブ**: `#targetengine "session"` を使用して、スクリプト実行全体で変数を保持します

### 命名規則 (JavaScript)

- オブジェクトとプロパティは **camelCase** を使用します: `activeDocument`、`pathItems`、`textFrames`
- `app` グローバルは `Application` オブジェクトを参照します
- コレクションのインデックスは **0 から始まります**: `documents[0]` が最前面のドキュメントです
- `typename` プロパティを使用して実行時にオブジェクト タイプを識別します

## オブジェクトモデルの概要

Illustrator DOM は、厳密な包含階層に従います。
```
Application (app)
├── activeDocument / documents[]
│   ├── layers[]
│   │   ├── pageItems[] (all artwork)
│   │   ├── pathItems[]
│   │   ├── compoundPathItems[]
│   │   ├── textFrames[]
│   │   ├── placedItems[]
│   │   ├── rasterItems[]
│   │   ├── meshItems[]
│   │   ├── pluginItems[]
│   │   ├── graphItems[]
│   │   ├── symbolItems[]
│   │   ├── nonNativeItems[]
│   │   ├── legacyTextItems[]
│   │   └── groupItems[]
│   ├── artboards[]
│   ├── views[]
│   ├── selection (array of selected items)
│   ├── swatches[], spots[], gradients[], patterns[]
│   ├── graphicStyles[], brushes[], symbols[]
│   ├── textFonts[] (via app.textFonts)
│   ├── stories[], characterStyles[], paragraphStyles[]
│   ├── variables[], datasets[]
│   └── inkList[], printOptions
├── preferences
├── printerList[]
└── textFonts[]
```

### 最上位オブジェクト

- **アプリケーション** (`app`): ルート オブジェクト。ドキュメント、環境設定、フォント、プリンターへのアクセスを提供します。主要なプロパティ: `activeDocument`、`documents`、`textFonts`、`printerList`、`userInteractionLevel`、`version`。
- **ドキュメント**: 開いている `.ai` ファイルを表します。主要なプロパティ: `layers`、`pageItems`、`selection`、`activeLayer`、`width`、`height`、`rulerOrigin`、`documentColorSpace`。主なメソッド: `saveAs()`、`exportFile()`、`close()`、`print()`。
- **レイヤー**: 描画レイヤー。主要なプロパティ: `pageItems`、`pathItems`、`textFrames`、`visible`、`locked`、`opacity`、`name`、`zOrderPosition`、`color`。

## 測定単位と座標

### 単位

すべてのスクリプト API 値は **ポイント** (72 ポイント = 1 インチ) を使用します。他の単位を変換するには:

|単位 |変換 |
|---|---|
|インチ | 72 を掛ける |
|センチメートル | 28.346 を掛ける |
|ミリメートル | 2.834645 を掛ける |
|ピカス | 12 を掛ける |

カーニング、トラッキング、および `aki` プロパティは **em 単位** (em の 1,000 分の 1、フォント サイズに比例) を使用します。

### 座標系

- **スクリプトドキュメント**の場合、原点`(0,0)`はアートボードの**左下**にあります
- X は左から右に増加します。 Y は下から上に増加します
- ページ項目の `position` プロパティは、`[x, y]` のように、その境界ボックスの **左上隅** にあります。
- ページ項目の最大幅/高さ: 16348 ポイント

### アートアイテムの境界

すべてのページ項目には 3 つの境界四角形があります。

- `geometricBounds`: ストローク幅を除きます `[left, top, right, bottom]`
- `visibleBounds`: ストローク幅を含む
- `controlBounds`: 制御点/方向点を含む

## ドキュメントの操作

### 作成して開く
```javascript
// Create a new document
var doc = app.documents.add();

// Create with a preset
var preset = new DocumentPreset();
preset.width = 612;  // 8.5 inches
preset.height = 792; // 11 inches
preset.colorMode = DocumentColorSpace.CMYK;
var doc = app.documents.addDocument("Print", preset);

// Open an existing file
var fileRef = new File("/path/to/file.ai");
var doc = app.open(fileRef);
```

### 保存とエクスポート
```javascript
// Save as Illustrator format
var saveOpts = new IllustratorSaveOptions();
saveOpts.compatibility = Compatibility.ILLUSTRATOR17; // CC
doc.saveAs(new File("/path/to/output.ai"), saveOpts);

// Export as PDF
var pdfOpts = new PDFSaveOptions();
pdfOpts.compatibility = PDFCompatibility.ACROBAT7;
pdfOpts.preserveEditability = false;
doc.saveAs(new File("/path/to/output.pdf"), pdfOpts);

// Export as PNG
var pngOpts = new ExportOptionsPNG24();
pngOpts.horizontalScale = 300;
pngOpts.verticalScale = 300;
pngOpts.transparency = true;
doc.exportFile(new File("/path/to/output.png"), ExportType.PNG24, pngOpts);

// Export as SVG
var svgOpts = new ExportOptionsSVG();
svgOpts.fontType = SVGFontType.OUTLINEFONT;
doc.exportFile(new File("/path/to/output.svg"), ExportType.SVG, svgOpts);
```

## パスとシェイプの操作

### 組み込みの形状メソッド

`pathItems` コレクションは、一般的な図形に便利なメソッドを提供します。
```javascript
var doc = app.activeDocument;
var layer = doc.activeLayer;

// Rectangle: rectangle(top, left, width, height)
var rect = layer.pathItems.rectangle(500, 100, 200, 150);

// Rounded rectangle: roundedRectangle(top, left, width, height, hRadius, vRadius)
var rrect = layer.pathItems.roundedRectangle(500, 100, 200, 150, 20, 20);

// Ellipse: ellipse(top, left, width, height)
var oval = layer.pathItems.ellipse(400, 200, 100, 100);

// Polygon: polygon(centerX, centerY, radius, sides)
var hex = layer.pathItems.polygon(300, 300, 50, 6);

// Star: star(centerX, centerY, radius, innerRadius, points)
var star = layer.pathItems.star(300, 300, 50, 25, 5);
```

### 座標配列を使用した自由形式のパス
```javascript
var doc = app.activeDocument;
var path = doc.pathItems.add();
path.setEntirePath([[100, 100], [200, 200], [300, 100]]);
path.closed = false;
path.stroked = true;
path.strokeWidth = 2;
```

### PathPoint オブジェクトを使用したフリーフォーム パス
```javascript
var doc = app.activeDocument;
var path = doc.pathItems.add();

var point1 = path.pathPoints.add();
point1.anchor = [100, 100];
point1.leftDirection = [100, 100];
point1.rightDirection = [150, 150];
point1.pointType = PointType.SMOOTH;

var point2 = path.pathPoints.add();
point2.anchor = [300, 100];
point2.leftDirection = [250, 150];
point2.rightDirection = [300, 100];
point2.pointType = PointType.SMOOTH;

path.closed = false;
```

### パスのプロパティ
```javascript
var item = doc.pathItems[0];
item.filled = true;
item.stroked = true;
item.strokeWidth = 1.5;
item.strokeCap = StrokeCap.ROUNDENDCAP;
item.strokeJoin = StrokeJoin.ROUNDENDJOIN;
item.opacity = 80;
item.closed = true;
```

## 色の操作

### カラーオブジェクト
```javascript
// RGB Color (values 0-255)
var red = new RGBColor();
red.red = 255;
red.green = 0;
red.blue = 0;

// CMYK Color (values 0-100)
var cyan = new CMYKColor();
cyan.cyan = 100;
cyan.magenta = 0;
cyan.yellow = 0;
cyan.black = 0;

// Grayscale (0-100, 0 = black)
var gray = new GrayColor();
gray.gray = 50;

// Lab Color
var lab = new LabColor();
lab.l = 50;
lab.a = 20;
lab.b = -30;

// No color (transparent)
var none = new NoColor();
```

### 色の適用
```javascript
var item = doc.pathItems[0];
item.fillColor = red;
item.strokeColor = cyan;

// Gradient fill
var gradient = doc.gradients.add();
gradient.type = GradientType.LINEAR;
gradient.gradientStops[0].color = red;
gradient.gradientStops[1].color = cyan;

var gradColor = new GradientColor();
gradColor.gradient = gradient;
item.fillColor = gradColor;
```

### スポットカラーとスウォッチ
```javascript
// Create a spot color
var spot = doc.spots.add();
spot.name = "My Spot Color";
spot.color = red; // Base color definition

var spotColor = new SpotColor();
spotColor.spot = spot;
spotColor.tint = 100;

item.fillColor = spotColor;

// Access a swatch by name
var swatch = doc.swatches.getByName("PANTONE 185 C");
item.fillColor = swatch.color;
```

## テキストの操作

### テキストフレームの種類
```javascript
var doc = app.activeDocument;

// Point text
var pointText = doc.textFrames.add();
pointText.contents = "Hello World!";
pointText.position = [100, 500];

// Area text (text inside a path)
var rectPath = doc.pathItems.rectangle(500, 100, 200, 100);
var areaText = doc.textFrames.areaText(rectPath);
areaText.contents = "Text inside a rectangle shape.";

// Path text (text along a path)
var curvePath = doc.pathItems.add();
curvePath.setEntirePath([[50, 300], [150, 400], [250, 300]]);
var pathText = doc.textFrames.pathText(curvePath);
pathText.contents = "Text on a path";
```

### 文字と段落の書式設定
```javascript
var tf = doc.textFrames[0];
var textRange = tf.textRange;

// Character attributes
var charAttr = textRange.characterAttributes;
charAttr.size = 24;           // Font size in points
charAttr.textFont = app.textFonts.getByName("ArialMT");
charAttr.fillColor = red;
charAttr.tracking = 50;       // Em units
charAttr.horizontalScale = 100;
charAttr.verticalScale = 100;
charAttr.baselineShift = 0;

// Paragraph attributes
var paraAttr = textRange.paragraphAttributes;
paraAttr.justification = Justification.CENTER;
paraAttr.firstLineIndent = 0;
paraAttr.leftIndent = 0;
paraAttr.spaceBefore = 0;
paraAttr.spaceAfter = 0;
```

### テキストコンテンツへのアクセス
```javascript
var tf = doc.textFrames[0];

// Access sub-ranges
var firstChar = tf.characters[0];
var firstWord = tf.words[0];
var firstPara = tf.paragraphs[0];
var firstLine = tf.lines[0];

// Modify specific ranges
tf.words[0].characterAttributes.size = 36;
tf.paragraphs[0].paragraphAttributes.justification = Justification.LEFT;
```

### テキストフレームのスレッド化
```javascript
var frame1 = doc.textFrames.areaText(path1);
var frame2 = doc.textFrames.areaText(path2);

// Link frames so text flows from frame1 to frame2
frame1.nextFrame = frame2;

// Stories represent the full text across threaded frames
var storyCount = doc.stories.length;
var fullText = doc.stories[0].textRange.contents;
```

## レイヤーの操作
```javascript
var doc = app.activeDocument;

// Create a layer
var newLayer = doc.layers.add();
newLayer.name = "Background";
newLayer.visible = true;
newLayer.locked = false;
newLayer.opacity = 100;

// Access existing layers
var topLayer = doc.layers[0];
var layerByName = doc.layers.getByName("Background");

// Move items between layers
var item = doc.pathItems[0];
item.move(newLayer, ElementPlacement.PLACEATBEGINNING);

// Reorder layers
newLayer.zOrder(ZOrderMethod.SENDTOBACK);
```

## 選択範囲の操作
```javascript
// Get current selection
var sel = app.activeDocument.selection;

// Iterate selected items
for (var i = 0; i < sel.length; i++) {
    var item = sel[i];
    // Check type using typename
    if (item.typename === "PathItem") {
        item.fillColor = red;
    } else if (item.typename === "TextFrame") {
        item.contents = "Modified";
    }
}

// Select an item programmatically
doc.pathItems[0].selected = true;

// Deselect all
doc.selection = null;
```

## シンボルの操作
```javascript
// Place a symbol instance
var sym = doc.symbols.getByName("MySymbol");
var instance = doc.symbolItems.add(sym);
instance.position = [200, 400];

// Access symbol definition
var symDef = instance.symbol;

// Break link to symbol (expand to regular art)
instance.breakLink();
```

## 変換
```javascript
var item = doc.pathItems[0];

// Rotate 45 degrees around center
item.rotate(45);

// Scale to 50% width, 75% height
item.resize(50, 75);

// Translate (move) by 100 points right and 50 points up
item.translate(100, 50);

// Using a transformation matrix
var matrix = app.getIdentityMatrix();
matrix = app.concatenateRotationMatrix(matrix, 30);
matrix = app.concatenateScaleMatrix(matrix, 150, 150);
item.transform(matrix);
```

## アートボードの操作
```javascript
var doc = app.activeDocument;

// Access artboards
var ab = doc.artboards[0];
var rect = ab.artboardRect; // [left, top, right, bottom]

// Create a new artboard
var newAB = doc.artboards.add([0, 0, 612, 792]); // Letter size
newAB.name = "Page 2";

// Set active artboard
doc.artboards.setActiveArtboardIndex(1);
```

## データ駆動型グラフィックス (変数とデータセット)
```javascript
// Variables link document items to data fields
var v = doc.variables.add();
v.kind = VariableKind.TEXTUAL;
v.name = "headline";

// Link a text frame to the variable
var tf = doc.textFrames[0];
tf.contentVariable = v;

// Create datasets for batch content
var ds = doc.dataSets.add();
ds.name = "Version 1";
// Dataset captures current variable bindings

// Switch datasets to swap content
doc.dataSets[0].display();
```

## 印刷
```javascript
var doc = app.activeDocument;
var opts = new PrintOptions();

opts.printPreset = "Default";

// Paper options
var paperOpts = new PrintPaperOptions();
paperOpts.name = "Letter";
opts.paperOptions = paperOpts;

// Job options
var jobOpts = new PrintJobOptions();
jobOpts.copies = 1;
jobOpts.designation = PrintArtworkDesignation.VISIBLELAYERS;
opts.jobOptions = jobOpts;

doc.print(opts);
```

## ユーザーインタラクションレベル

スクリプトの実行中に Illustrator がダイアログを表示するかどうかを制御します。
```javascript
// Suppress all dialogs
app.userInteractionLevel = UserInteractionLevel.DONTDISPLAYALERTS;

// Perform operations that might prompt dialogs...
doc.close(SaveOptions.DONOTSAVECHANGES);

// Restore dialog display
app.userInteractionLevel = UserInteractionLevel.DISPLAYALERTS;
```

## メソッドの操作 (JavaScript 固有)

複数のオプションのパラメータを指定してメソッドを呼び出す場合は、`undefined` を使用して中間のパラメータをスキップします。
```javascript
// rotate(angle, [changePositions], [changeFillPatterns], [changeFillGradients], ...)
item.rotate(30, undefined, undefined, true);
```

## よくあるパターン

### ドキュメント内のすべてのページ項目を反復する
```javascript
function processAllItems(doc) {
    for (var i = 0; i < doc.pageItems.length; i++) {
        var item = doc.pageItems[i];
        // Process based on type
        switch (item.typename) {
            case "PathItem":
                // handle path
                break;
            case "TextFrame":
                // handle text
                break;
            case "GroupItem":
                // handle group (may contain nested items)
                break;
        }
    }
}
```

### フォルダー内のファイルをバッチ処理する
```javascript
var folder = Folder.selectDialog("Select folder of .ai files");
if (folder) {
    var files = folder.getFiles("*.ai");
    for (var i = 0; i < files.length; i++) {
        var doc = app.open(files[i]);
        // Process each document...
        doc.close(SaveOptions.DONOTSAVECHANGES);
    }
}
```

### エラー処理
```javascript
try {
    var doc = app.activeDocument;
    var layer = doc.layers.getByName("NonExistentLayer");
} catch (e) {
    alert("Error: " + e.message);
    // e.message, e.line, e.fileName available
}
```

## トラブルシューティング

- **「未定義はオブジェクトではありません」**: 通常、コレクションが空であるか、インデックスが範囲外であることを意味します。項目にアクセスする前に `.length` を確認してください。
- **スクリプトは実行されますが、見た目には何も変わりません**: `app.redraw()` を呼び出して、変更後に画面を強制的に更新します。
- **カラー モードの不一致**: ドキュメントのカラー スペース (RGB 対 CMYK) はカラー オブジェクトと一致する必要があります。 `doc.documentColorSpace` を使用して確認してください。
- **位置が間違っているようです**: スクリプト化されたドキュメントでは、Y が上向きに増加する左下の原点を使用することに注意してください。 `position` プロパティは境界ボックスの左上にあります。
- **テキストが表示されない**: テキスト フレームのサイズがゼロ以外であることを確認してください。ポイントテキストの場合は、`position` を設定します。エリアテキストの場合は、`areaText()` への有効なパスを指定します。
- **Windows 上のファイル パス**: パス文字列でスラッシュ (`/`) または二重バックスラッシュ (`\\`) を使用するか、`File` オブジェクト コンストラクターを使用します。
- **バッチ スクリプトを中断するダイアログ ボックス**: バッチ操作の前に `app.userInteractionLevel = UserInteractionLevel.DONTDISPLAYALERTS` を設定します。
- **コレクションは `getByName()` を使用します**: 多くのコレクション オブジェクトは `getByName("name")` をサポートしており、見つからない場合はエラーがスローされます。 try/catch でラップします。

## スクリプト定数のリファレンス

API 全体で使用される共通の列挙定数:

|カテゴリー |定数 |
|---|---|
| **色空間** | `DocumentColorSpace.RGB`、`DocumentColorSpace.CMYK` |
| **正当化** | `Justification.LEFT`、`Justification.CENTER`、`Justification.RIGHT`、`Justification.FULLJUSTIFY` |
| **ポイントタイプ** | `PointType.SMOOTH`、`PointType.CORNER` |
| **ストロークキャップ** | `StrokeCap.BUTTENDCAP`、`StrokeCap.ROUNDENDCAP`、`StrokeCap.PROJECTINGENDCAP` |
| **ストローク結合** | `StrokeJoin.MITERENDJOIN`、`StrokeJoin.ROUNDENDJOIN`、`StrokeJoin.BEVELENDJOIN` |
| **ブレンドモード** | `BlendModes.NORMAL`、`BlendModes.MULTIPLY`、`BlendModes.SCREEN`、`BlendModes.OVERLAY` |
| **保存オプション** | `SaveOptions.SAVECHANGES`、`SaveOptions.DONOTSAVECHANGES`、`SaveOptions.PROMPTTOSAVECHANGES` |
| **エクスポート タイプ** | `ExportType.PNG24`、`ExportType.PNG8`、`ExportType.JPEG`、`ExportType.SVG`、`ExportType.TIFF`、`ExportType.PHOTOSHOP`、`ExportType.AUTOCAD`、`ExportType.FLASH` |
| **要素の配置** | `ElementPlacement.PLACEATBEGINNING`、`ElementPlacement.PLACEATEND`、`ElementPlacement.PLACEBEFORE`、`ElementPlacement.PLACEAFTER`、`ElementPlacement.INSIDE` |
| **Z オーダー** | `ZOrderMethod.BRINGTOFRONT`、`ZOrderMethod.SENDTOBACK`、`ZOrderMethod.BRINGFORWARD`、`ZOrderMethod.SENDBACKWARD` |
| **グラデーションタイプ** | `GradientType.LINEAR`、`GradientType.RADIAL` |
| **テキストフレームの種類** | `TextType.POINTTEXT`、`TextType.AREATEXT`、`TextType.PATHTEXT` |
| **変数の種類** | `VariableKind.TEXTUAL`、`VariableKind.IMAGE`、`VariableKind.VISIBILITY`、`VariableKind.GRAPH` |
| **ユーザー インタラクション** | `UserInteractionLevel.DISPLAYALERTS`、`UserInteractionLevel.DONTDISPLAYALERTS` |
| **互換性** | `Compatibility.ILLUSTRATOR10` から `Compatibility.ILLUSTRATOR24` |

## JavaScript オブジェクト リファレンス (完全な API オブジェクト リスト)

Illustrator JavaScript API には、カテゴリ別にグループ化された次のオブジェクトが含まれています。

### コアオブジェクト

`Application`、`Document`、`Documents`、`DocumentPreset`、`Layer`、`Layers`、`PageItem`、`PageItems`、`View`、`Views`、`Preferences`

### パスとシェイプオブジェクト

`PathItem`、`PathItems`、`PathPoint`、`PathPoints`、`CompoundPathItem`、`CompoundPathItems`、`GroupItem`、`GroupItems`

### テキストオブジェクト

`TextFrame`、`TextRange`、`TextRanges`、`TextPath`、`Characters`、`Words`、`Paragraphs`、`Lines`、`InsertionPoint`、`InsertionPoints`、`Story`、`Stories`、`CharacterAttributes`、`ParagraphAttributes`、 `CharacterStyle`、`CharacterStyles`、`ParagraphStyle`、`ParagraphStyles`、`TextFont`、`TextFonts`、`TabStopInfo`

### カラーオブジェクト

`RGBColor`、`CMYKColor`、`GrayColor`、`LabColor`、`NoColor`、`SpotColor`、`Spot`、`Spots`、`PatternColor`、`GradientColor`、`Color`、`Gradient`、`Gradients`、`GradientStop`、 `GradientStops`

### スウォッチとスタイルオブジェクト

`Swatch`、`Swatches`、`SwatchGroup`、`SwatchGroups`、`GraphicStyle`、`GraphicStyles`、`Pattern`、`Patterns`、`Brush`、`Brushes`

### シンボルオブジェクト

`Symbol`、`Symbols`、`SymbolItem`、`SymbolItems`

### アートボードオブジェクト

`Artboard`、`Artboards`

### 配置されたラスター オブジェクト

`PlacedItem`、`PlacedItems`、`RasterItem`、`RasterItems`、`MeshItem`、`MeshItems`、`GraphItem`、`GraphItems`、`PluginItem`、`PluginItems`、`NonNativeItem`、`NonNativeItems`、`LegacyTextItem`、`LegacyTextItems`

### データ駆動型オブジェクト

`Variable`、`Variables`、`Dataset`、`Datasets`

### マトリックスと変換オブジェクト

@@コード0@@

### タグオブジェクト

`Tag`、`Tags`

### オブジェクトのトレース

`TracingObject`、`TracingOptions`

### 保存およびエクスポートのオプション

`IllustratorSaveOptions`、`EPSSaveOptions`、`PDFSaveOptions`、`FXGSaveOptions`、`ExportOptionsAutoCAD`、`ExportOptionsFlash`、`ExportOptionsGIF`、`ExportOptionsJPEG`、`ExportOptionsPhotoshop`、`ExportOptionsPNG8`、`ExportOptionsPNG24`、`ExportOptionsSVG`、`ExportOptionsTIFF`

### オープンオプション

`OpenOptions`、`OpenOptionsAutoCAD`、`OpenOptionsFreeHand`、`OpenOptionsPhotoshop`、`PDFFileOptions`、`PhotoshopFileOptions`

### 印刷オブジェクト

`PrintOptions`、`PrintJobOptions`、`PrintPaperOptions`、`PrintColorManagementOptions`、`PrintColorSeparationOptions`、`PrintCoordinateOptions`、`PrintFlattenerOptions`、`PrintFontOptions`、`PrintPageMarksOptions`、`PrintPostScriptOptions`、`Printer`、`PrinterInfo`、`Paper`、`PaperInfo`、 `PPDFile`、`PPDFileInfo`、`Ink`、`InkInfo`、`Screen`、`ScreenInfo`、`ScreenSpotFunction`

### 画像とラスタライズのオプション

`ImageCaptureOptions`、`RasterEffectOptions`、`RasterizeOptions`

## 参考文献

- [変更ログ](https://ai-scripting.docsforadobe.dev/introduction/changelog/) - 最近のスクリプト API の変更 (CC 2020 では `Document.getPageItemFromUuid` および `PageItem.uuid` が追加されました。CC 2017 では `Application.getIsFileOpen` が追加されました)
- [Illustrator スクリプト ガイド](https://ai-scripting.docsforadobe.dev/) - コミュニティが管理する完全なドキュメント