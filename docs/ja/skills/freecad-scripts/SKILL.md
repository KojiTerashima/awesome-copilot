---
name: freecad-scripts
description: 'Expert skill for writing FreeCAD Python scripts, macros, and automation. Use when asked to create FreeCAD models, parametric objects, Part/Mesh/Sketcher scripts, workbench tools, GUI dialogs with PySide, Coin3D scenegraph manipulation, or any FreeCAD Python API task. Covers FreeCAD scripting basics, geometry creation, FeaturePython objects, interface tools, and macro development.'
---
# FreeCAD スクリプト

FreeCAD CAD アプリケーション用の製品品質の Python スクリプトを生成するための専門スキル。 3D モデリング タスクの短縮表現、準コード、および自然言語の説明を解釈し、それらを正しい FreeCAD Python API 呼び出しに変換します。

## このスキルを使用する場合

- FreeCAD の組み込みコンソールまたはマクロ システム用の Python スクリプトの作成
- 3D ジオメトリの作成または操作 (パーツ、メッシュ、スケッチャー、パス、FEM)
- カスタム プロパティを使用してパラメトリックな FeaturePython オブジェクトを構築する
- FreeCAD 内で PySide/Qt を使用した GUI ツールの開発
- Pivy 経由で Coin3D シーングラフを操作する
- カスタムワークベンチまたはGuiコマンドの作成
- 繰り返しのCAD操作をマクロで自動化
- メッシュ表現とソリッド表現の間の変換
- FEM 解析、レイトレーシング、または図面のエクスポートのスクリプト作成

## 前提条件

- FreeCAD がインストールされている (0.19 以降を推奨、最新 API の場合は 0.21 以降/1.0 以降)
- Python 3.x (FreeCAD にバンドルされている)
- GUI 作業の場合: PySide2 (FreeCAD にバンドルされている)
- シーングラフの場合: Pivy (FreeCAD にバンドルされている)

## FreeCAD Python 環境

FreeCAD には Python インタープリターが組み込まれています。スクリプトは、次の主要モジュールが利用可能な環境で実行されます。```python
import FreeCAD          # Core module (also aliased as 'App')
import FreeCADGui       # GUI module (also aliased as 'Gui') — only in GUI mode
import Part             # Part workbench — BRep/OpenCASCADE shapes
import Mesh             # Mesh workbench — triangulated meshes
import Sketcher         # Sketcher workbench — 2D constrained sketches
import Draft            # Draft workbench — 2D drawing tools
import Arch             # Arch/BIM workbench
import Path             # Path/CAM workbench
import FEM              # FEM workbench
import TechDraw         # TechDraw workbench (replaces Drawing)
import BOPTools         # Boolean operations
import CompoundTools    # Compound shape utilities
```### FreeCAD ドキュメント モデル```python
# Create or access a document
doc = FreeCAD.newDocument("MyDoc")
doc = FreeCAD.ActiveDocument

# Add objects
box = doc.addObject("Part::Box", "MyBox")
box.Length = 10.0
box.Width = 10.0
box.Height = 10.0

# Recompute
doc.recompute()

# Access objects
obj = doc.getObject("MyBox")
obj = doc.MyBox  # Attribute access also works

# Remove objects
doc.removeObject("MyBox")
```## コアコンセプト

### ベクトルと配置```python
import FreeCAD

# Vectors
v1 = FreeCAD.Vector(1, 0, 0)
v2 = FreeCAD.Vector(0, 1, 0)
v3 = v1.cross(v2)          # Cross product
d = v1.dot(v2)              # Dot product
v4 = v1 + v2                # Addition
length = v1.Length           # Magnitude
v_norm = FreeCAD.Vector(v1)
v_norm.normalize()           # In-place normalize

# Rotations
rot = FreeCAD.Rotation(FreeCAD.Vector(0, 0, 1), 45)  # axis, angle(deg)
rot = FreeCAD.Rotation(0, 0, 45)                       # Euler angles (yaw, pitch, roll)

# Placements (position + orientation)
placement = FreeCAD.Placement(
    FreeCAD.Vector(10, 20, 0),    # translation
    FreeCAD.Rotation(0, 0, 45),   # rotation
    FreeCAD.Vector(0, 0, 0)       # center of rotation
)
obj.Placement = placement

# Matrix (4x4 transformation)
import math
mat = FreeCAD.Matrix()
mat.move(FreeCAD.Vector(10, 0, 0))
mat.rotateZ(math.radians(45))
```### ジオメトリの作成と操作 (パーツ モジュール)

Part モジュールは OpenCASCADE をラップし、BRep ソリッド モデリングを提供します。```python
import FreeCAD
import Part

# --- Primitive Shapes ---
box = Part.makeBox(10, 10, 10)               # length, width, height
cyl = Part.makeCylinder(5, 20)               # radius, height
sphere = Part.makeSphere(10)                  # radius
cone = Part.makeCone(5, 2, 10)               # r1, r2, height
torus = Part.makeTorus(10, 2)                 # major_r, minor_r

# --- Wires and Edges ---
edge1 = Part.makeLine((0, 0, 0), (10, 0, 0))
edge2 = Part.makeLine((10, 0, 0), (10, 10, 0))
edge3 = Part.makeLine((10, 10, 0), (0, 0, 0))
wire = Part.Wire([edge1, edge2, edge3])

# Circles and arcs
circle = Part.makeCircle(5)                   # radius
arc = Part.makeCircle(5, FreeCAD.Vector(0, 0, 0),
                       FreeCAD.Vector(0, 0, 1), 0, 180)  # start/end angle

# --- Faces ---
face = Part.Face(wire)                        # From a closed wire

# --- Solids from Faces/Wires ---
extrusion = face.extrude(FreeCAD.Vector(0, 0, 10))       # Extrude
revolved = face.revolve(FreeCAD.Vector(0, 0, 0),
                         FreeCAD.Vector(0, 0, 1), 360)    # Revolve

# --- Boolean Operations ---
fused = box.fuse(cyl)           # Union
cut = box.cut(cyl)              # Subtraction
common = box.common(cyl)        # Intersection
fused_clean = fused.removeSplitter()  # Clean up seams

# --- Fillets and Chamfers ---
filleted = box.makeFillet(1.0, box.Edges)          # radius, edges
chamfered = box.makeChamfer(1.0, box.Edges)        # dist, edges

# --- Loft and Sweep ---
loft = Part.makeLoft([wire1, wire2], True)          # wires, solid
swept = Part.Wire([path_edge]).makePipeShell([profile_wire],
                                              True, False)  # solid, frenet

# --- BSpline Curves ---
from FreeCAD import Vector
points = [Vector(0,0,0), Vector(1,2,0), Vector(3,1,0), Vector(4,3,0)]
bspline = Part.BSplineCurve()
bspline.interpolate(points)
edge = bspline.toShape()

# --- Show in document ---
Part.show(box, "MyBox")    # Quick display (adds to active doc)
# Or explicitly:
doc = FreeCAD.ActiveDocument or FreeCAD.newDocument()
obj = doc.addObject("Part::Feature", "MyShape")
obj.Shape = box
doc.recompute()
```### トポロジカル探索```python
shape = obj.Shape

# Access sub-elements
shape.Vertexes    # List of Vertex objects
shape.Edges       # List of Edge objects
shape.Wires       # List of Wire objects
shape.Faces       # List of Face objects
shape.Shells      # List of Shell objects
shape.Solids      # List of Solid objects

# Bounding box
bb = shape.BoundBox
print(bb.XMin, bb.XMax, bb.YMin, bb.YMax, bb.ZMin, bb.ZMax)
print(bb.Center)

# Properties
shape.Volume
shape.Area
shape.Length       # For edges/wires
face.Surface       # Underlying geometric surface
edge.Curve         # Underlying geometric curve

# Shape type
shape.ShapeType    # "Solid", "Shell", "Face", "Wire", "Edge", "Vertex", "Compound"
```### メッシュモジュール```python
import Mesh

# Create mesh from vertices and facets
mesh = Mesh.Mesh()
mesh.addFacet(
    0.0, 0.0, 0.0,   # vertex 1
    1.0, 0.0, 0.0,   # vertex 2
    0.0, 1.0, 0.0    # vertex 3
)

# Import/Export
mesh = Mesh.Mesh("/path/to/file.stl")
mesh.write("/path/to/output.stl")

# Convert Part shape to Mesh
import Part
import MeshPart
shape = Part.makeBox(1, 1, 1)
mesh = MeshPart.meshFromShape(Shape=shape, LinearDeflection=0.1,
                                AngularDeflection=0.5)

# Convert Mesh to Part shape
shape = Part.Shape()
shape.makeShapeFromMesh(mesh.Topology, 0.05)  # tolerance
solid = Part.makeSolid(shape)
```### スケッチャーモジュール

# XY平面上にスケッチを作成
スケッチ = doc.addObject("スケッチャー::スケッチオブジェクト", "MySketch")
スケッチ.配置 = FreeCAD.配置(
    FreeCAD.Vector(0, 0, 0),
    FreeCAD.Rotation(0, 0, 0, 1)
）

# ジオメトリを追加 (ジオメトリ インデックスを返します)
idx_line =sketch.addGeometry(Part.LineSegment(
    FreeCAD.Vector(0, 0, 0), FreeCAD.Vector(10, 0, 0)))
idx_circle =sketch.addGeometry(Part.Circle(
    FreeCAD.Vector(5, 5, 0)、FreeCAD.Vector(0, 0, 1), 3))

# 制約を追加する
sketch.addConstraint(Sketcher.Constraint("一致", 0, 2, 1, 1))
スケッチ.addConstraint(Sketcher.Constraint("水平", 0))
スケッチ.addConstraint(Sketcher.Constraint("距離X", 0, 1, 0, 2, 10.0))
sketch.addConstraint(Sketcher.Constraint("半径", 1, 3.0))
sketch.addConstraint(Sketcher.Constraint("固定", 0, 1))
# 拘束タイプ: 一致、水平、垂直、平行、垂直、
# 接線、等価、対称、距離、距離X、距離Y、半径、角度、
# 固定 (ブロック)、InternalAlignment

doc.recompute()```

### Draft Module

```パイソン
ドラフトをインポート
FreeCAD をインポートする

# 2D 形状
line = Draft.makeLine(FreeCAD.Vector(0,0,0), FreeCAD.Vector(10,0,0))
サークル = Draft.makeCircle(5)
rect = Draft.makeRectangle(10, 5)
ポリ = Draft.makePolygon(6, radius=5) # 六角形

# 操作
移動 = Draft.move(obj, FreeCAD.Vector(10, 0, 0), copy=True)
回転 = Draft.rotate(obj, 45, FreeCAD.Vector(0,0,0),
                        軸=FreeCAD.Vector(0,0,1)、コピー=True)
スケール = Draft.scale(obj, FreeCAD.Vector(2,2,2), center=FreeCAD.Vector(0,0,0),
                      コピー=真)
offset = Draft.offset(obj, FreeCAD.Vector(1,0,0))
配列 = Draft.makeArray(obj, FreeCAD.Vector(15,0,0),
                         FreeCAD.Vector(0,15,0), 3, 3)```

## Creating Parametric Objects (FeaturePython)

FeaturePython objects are custom parametric objects with properties that trigger recomputation:

```パイソン
FreeCAD をインポートする
パーツのインポート

クラスMyBox:
    """カスタムパラメトリックボックス。"""

    def __init__(self, obj):
        obj.Proxy = 自己
        obj.addProperty("App::PropertyLength", "長さ", "寸法",
                         "ボックスの長さ").長さ = 10.0
        obj.addProperty("App::PropertyLength", "幅", "寸法",
                         "ボックスの幅").Width = 10.0
        obj.addProperty("App::PropertyLength", "高さ", "寸法",
                         "ボックスの高さ").高さ = 10.0

    def 実行(self, obj):
        """ドキュメントの再計算時に呼び出されます。"""
        obj.Shape = Part.makeBox(obj.Length, obj.Width, obj.Height)

    def onChanged(self, obj, prop):
        """プロパティが変更されると呼び出されます。"""
        パスする

    def __getstate__(self):
        なしを返す

    def __setstate__(self, state):
        なしを返す


クラス ViewProviderMyBox:
    """カスタム アイコンと表示設定のプロバイダーを表示します。"""

    def __init__(self, vobj):
        vobj.Proxy = 自己

    def getIcon(self):
        ":/icons/Part_Box.svg" を返します

    defattach(self, vobj):
        self.オブジェクト = vobj.オブジェクト

    def updateData(self, obj, prop):
        パスする

    def onChanged(self, vobj, prop):
        パスする

    def __getstate__(self):
        なしを返す

    def __setstate__(self, state):
        なしを返す


# --- 使用法 ---
doc = FreeCAD.ActiveDocument または FreeCAD.newDocument("Test")
obj = doc.addObject("Part::FeaturePython", "CustomBox")
MyBox(オブジェクト)
ViewProviderMyBox(obj.ViewObject)
doc.recompute()```

### Common Property Types

| Property Type | Python Type | Description |
|---|---|---|
| `App::PropertyBool` | `bool` | Boolean |
| `App::PropertyInteger` | `int` | Integer |
| `App::PropertyFloat` | `float` | Float |
| `App::PropertyString` | `str` | String |
| `App::PropertyLength` | `float` (units) | Length with units |
| `App::PropertyAngle` | `float` (deg) | Angle in degrees |
| `App::PropertyVector` | `FreeCAD.Vector` | 3D vector |
| `App::PropertyPlacement` | `FreeCAD.Placement` | Position + rotation |
| `App::PropertyLink` | object ref | Link to another object |
| `App::PropertyLinkList` | list of refs | Links to multiple objects |
| `App::PropertyEnumeration` | `list`/`str` | Dropdown selection |
| `App::PropertyFile` | `str` | File path |
| `App::PropertyColor` | `tuple` | RGB color (0.0-1.0) |
| `App::PropertyPythonObject` | any | Serializable Python object |

## Creating GUI Tools

### Gui Commands

```パイソン
FreeCAD をインポートする
FreeCADGui をインポートする

クラスMyCommand:
    """カスタム ツールバー/メニュー コマンド。"""

    def GetResources(self):
        戻り値 {
            "ピックスマップ": ":/icons/Part_Box.svg",
            "MenuText": "私のカスタム コマンド",
            "ツールチップ": "カスタム ボックスを作成します",
            「アクセル」：「Ctrl+Shift+B」
        }

    def IsActive(self):
        FreeCAD.ActiveDocument が None ではないことを返します

    def Activated(自身):
        # コマンドロジックはここにあります
        FreeCAD.Console.PrintMessage("コマンドがアクティブになりました\n")

FreeCADGui.addCommand("My_CustomCommand", MyCommand())```

### PySide Dialogs

```パイソン
PySide2 から QtWidgets、QtCore、QtGui をインポート

クラスMyDialog(QtWidgets.QDialog):
    def __init__(self,parent=None):
        super().__init__(親またはFreeCADGui.getMainWindow())
        self.setWindowTitle("マイツール")
        self.setMinimumWidth(300)

        レイアウト = QtWidgets.QVBoxLayout(self)

        # 入力フィールド
        self.label = QtWidgets.QLabel("長さ:")
        self.spinbox = QtWidgets.QDoubleSpinBox()
        self.spinbox.setRange(0.1, 1000.0)
        self.spinbox.setValue(10.0)
        self.spinbox.setSuffix(" mm")

        フォーム = QtWidgets.QFormLayout()
        form.addRow(self.label, self.spinbox)
        レイアウト.addLayout(フォーム)

        # ボタン
        btn_layout = QtWidgets.QHBoxLayout()
        self.btn_ok = QtWidgets.QPushButton("OK")
        self.btn_cancel = QtWidgets.QPushButton("キャンセル")
        btn_layout.addWidget(self.btn_ok)
        btn_layout.addWidget(self.btn_cancel)
        レイアウト.addLayout(btn_layout)

        self.btn_ok.clicked.connect(self.accept)
        self.btn_cancel.clicked.connect(self.reject)

# 使用法
ダイアログ = MyDialog()
if Dialog.exec_() == QtWidgets.QDialog.Accepted:
    長さ = ダイアログ.スピンボックス.値()
    FreeCAD.Console.PrintMessage(f"長さ: {長さ}\n")```

### Task Panel (Recommended for FreeCAD integration)

```パイソン
クラスMyTaskPanel:
    """タスクパネルは左側のサイドバーに表示されます。"""

    def __init__(自分自身):
        self.form = QtWidgets.QWidget()
        レイアウト = QtWidgets.QVBoxLayout(self.form)
        self.spinbox = QtWidgets.QDoubleSpinBox()
        self.spinbox.setValue(10.0)
        layout.addWidget(QtWidgets.QLabel("長さ:"))
        layout.addWidget(self.spinbox)

    def accept(self):
        # ユーザーが「OK」をクリックすると呼び出されます
        長さ = self.spinbox.value()
        FreeCAD.Console.PrintMessage(f"受け入れられました: {length}\n")
        FreeCADGui.Control.closeDialog()
        Trueを返す

    デフォルト拒否(自分自身):
        FreeCADGui.Control.closeDialog()
        Trueを返す

    def getStandardButtons(self):
        return int(QtWidgets.QDialogButtonBox.Ok |
                   QtWidgets.QDialogButtonBox.Cancel)

# パネルを表示する
パネル = MyTaskPanel()
FreeCADGui.Control.showDialog(パネル)```

## Coin3D Scenegraph (Pivy)

```パイソン
Pivy輸入コインから
FreeCADGui をインポートする

# シーングラフのルートにアクセスする
sg = FreeCADGui.ActiveDocument.ActiveView.getSceneGraph()

# 球を使用したカスタムセパレーターを追加します
sep = コイン.SoSeparator()
マット = コイン.SoMaterial()
mat.diffuseColor.setValue(1.0, 0.0, 0.0) # 赤
trans = コイン.SoTranslation()
trans.translation.setValue(10, 10, 10)
球 = コイン.SoSphere()
sphere.radius.setValue(2.0)
sep.addChild(マット)
sep.addChild(trans)
sep.addChild(球体)
sg.addChild(sep)

# 後で削除する
sg.removeChild(sep)```

## Custom Workbench Creation

```パイソン
FreeCADGui をインポートする

クラス MyWorkbench(FreeCADGui.Workbench):
    MenuText = "私のワークベンチ"
    ツールヒント = "カスタム ワークベンチ"
    アイコン = ":/icons/freecad.svg"

    def 初期化(自分自身):
        """ワークベンチのアクティブ化時に呼び出されます。"""
        import MyCommands # コマンド モジュールをインポートします
        self.appendToolbar("マイ ツール", ["My_CustomCommand"])
        self.appendMenu("マイメニュー", ["My_CustomCommand"])

    def Activated(自身):
        パスする

    def 非アクティブ化(自分自身):
        パスする

    def GetClassName(self):
        "Gui::PythonWorkbench" を返す

FreeCADGui.addWorkbench(MyWorkbench)```

## Macro Best Practices

```パイソン
# 標準マクロヘッダー
# -*- コーディング: utf-8 -*-
# FreeCAD マクロ: MyMacro
# 説明: マクロの動作の簡単な説明
# 著者: あなたの名前
# バージョン: 1.0
# 日付: 2026-04-07

FreeCAD をインポートする
パーツのインポート
FreeCADインポートベースより

# GUI の可用性を保護する
FreeCAD.GuiUp の場合:
    FreeCADGui をインポートする
    PySide2 から QtWidgets、QtCore をインポート

def main():
    doc = FreeCAD.ActiveDocument
    doc が None の場合:
        FreeCAD.Console.PrintError("アクティブなドキュメントがありません\n")
        戻る

    FreeCAD.GuiUp の場合:
        sel = FreeCADGui.Selection.getSelection()
        そうでない場合:
            FreeCAD.Console.PrintWarning("オブジェクトが選択されていません\n")

    # ...マクロロジック ...

    doc.recompute()
    FreeCAD.Console.PrintMessage("マクロが完了しました\n")

__name__ == "__main__"の場合:
    メイン()```

### Selection Handling

```パイソン
# 選択したオブジェクトを取得する
sel = FreeCADGui.Selection.getSelection() # オブジェクトのリスト
sel_ex = FreeCADGui.Selection.getSelectionEx() # 拡張 (サブ要素)

sel_ex の selobj の場合:
    obj = selobj.オブジェクト
    selobj.SubElementNames のサブの場合:
        print(f"{obj.Name}.{sub}")
        Shape = obj.getSubObject(sub) # サブシェイプを取得します

# プログラムで選択する
FreeCADGui.Selection.addSelection(doc.MyBox)
FreeCADGui.Selection.addSelection(doc.MyBox, "Face1")
FreeCADGui.Selection.clearSelection()```

### Console Output

```パイソン
FreeCAD.Console.PrintMessage("情報メッセージ\n")
FreeCAD.Console.PrintWarning("警告メッセージ\n")
FreeCAD.Console.PrintError("エラー メッセージ\n")
FreeCAD.Console.PrintLog("デバッグ/ログ メッセージ\n")```

## Common Patterns

### Parametric Pad from Sketch

```パイソン
doc = FreeCAD.ActiveDocument

# スケッチを作成する
スケッチ = doc.addObject("スケッチャー::スケッチオブジェクト", "スケッチ")
スケッチ.addGeometry(Part.LineSegment(FreeCAD.Vector(0,0,0), FreeCAD.Vector(10,0,0)))
スケッチ.addGeometry(Part.LineSegment(FreeCAD.Vector(10,0,0), FreeCAD.Vector(10,10,0)))
スケッチ.addGeometry(Part.LineSegment(FreeCAD.Vector(10,10,0), FreeCAD.Vector(0,10,0)))
スケッチ.addGeometry(Part.LineSegment(FreeCAD.Vector(0,10,0), FreeCAD.Vector(0,0,0)))
# 一致制約で閉じる
range(3) の i の場合:
    sketch.addConstraint(Sketcher.Constraint("一致", i, 2, i+1, 1))
スケッチ.addConstraint(Sketcher.Constraint("一致", 3, 2, 0, 1))

# パッド (パーツデザイン)
Pad = doc.addObject("PartDesign::Pad", "Pad")
パッド.プロファイル = スケッチ
パッドの長さ = 5.0
スケッチ.可視性 = False
doc.recompute()```

### Export Shapes

```パイソン
# STEPエクスポート
Part.export([doc.MyBox], "/path/to/output.step")

# STL エクスポート (メッシュ)
メッシュのインポート
Mesh.export([doc.MyBox], "/path/to/output.stl")

# IGES エクスポート
Part.export([doc.MyBox], "/path/to/output.iges")

# importlib による複数のフォーマット
インポートインポート
importlib.import_module("importOBJ").export([doc.MyBox], "/path/to/output.obj")```

### Units and Quantities

```パイソン
# FreeCAD は内部で mm を使用します
q = FreeCAD.Units.Quantity("10 mm")
q_inch = FreeCAD.Units.Quantity("1 インチ")
print(q_inch.getValueAs("mm")) # 25.4

# ユーザー入力を単位で解析する
q = FreeCAD.Units.parseQuantity("2.5 インチ")
value_mm = float(q) # mm 単位の値 (内部単位)
「」

## 報酬ルール (準コーダー統合)

FreeCAD スクリプトの短縮表現または疑似コードを解釈する場合:

1. **用語のマッピング**: 「ボックス」→ `Part.makeBox()`、「円柱」→ `Part.makeCylinder()`、「球」→ `Part.makeSphere()`、「マージ/結合/結合」→ `.fuse()`、「減算/カット/削除」→ `.cut()`、「交差」→ `.common()`、「ラウンド エッジ/フィレット」→ `.makeFillet()`、 「面取り・面取り」 → `.makeChamfer()`
2. **暗黙的なドキュメント**: ドキュメントの処理が記載されていない場合は、標準の `doc = FreeCAD.ActiveDocument or FreeCAD.newDocument()` で囲みます。
3. **単位の仮定**: 特に明記されていない限り、デフォルトはミリメートルです。
4. **再計算**: 変更後は常に `doc.recompute()` を呼び出します
5. **GUI ガード**: スクリプトがヘッドレスで実行される可能性がある場合、GUI に依存するコードを `if FreeCAD.GuiUp:` でラップします。
6. **Part.show()**: クイック表示には `Part.show(shape, "Name")` を使用し、名前付き永続オブジェクトには `doc.addObject("Part::Feature", "Name")` を使用します

## 参考文献

### プライマリリンク

- [Python コードの書き方](https://wiki.freecad.org/Manual:A_gentle_introduction#Writing_Python_code)
- [FreeCAD オブジェクトの操作](https://wiki.freecad.org/Manual:A_gentle_introduction#Manipulated_FreeCAD_objects)
- [ベクターと配置](https://wiki.freecad.org/Manual:A_gentle_introduction#Vectors_and_Placements)
- [ジオメトリの作成と操作](https://wiki.freecad.org/Manual:Creating_and_manipulator_geometry)
- [パラメトリック オブジェクトの作成](https://wiki.freecad.org/Manual:Creating_parametric_objects)
- [インターフェイス ツールの作成](https://wiki.freecad.org/Manual:Creating_interface_tools)
- [Python](https://en.wikipedia.org/wiki/Python_%28プログラミング_言語%29)
- [Python 入門](https://wiki.freecad.org/ Introduction_to_Python)
- [Python スクリプト チュートリアル](https://wiki.freecad.org/Python_scripting_tutorial)
- [FreeCAD スクリプトの基礎](https://wiki.freecad.org/FreeCAD_Scripting_Basics)
- [Gui コマンド](https://wiki.freecad.org/Gui_Command)

### 同梱の参考資料

トピック別にまとめられたガイドについては、[references/](references/) ディレクトリを参照してください。

1. [scripting-fundamentals.md](references/scripting-fundamentals.md) — コア スクリプト、ドキュメント モデル、コンソール
2. [geometry-and-shapes.md](references/geometry-and-shapes.md) — パーツ、メッシュ、スケッチャー、トポロジ
3. [parametric-objects.md](references/parametric-objects.md) — 機能Python、プロパティ、スクリプト化されたオブジェクト
4. [gui-and-interface.md](references/gui-and-interface.md) — PySide、ダイアログ、タスクパネル、Coin3D
5. [workbenches-and-advanced.md](references/workbenches-and-advanced.md) — ワークベンチ、マクロ、FEM、パス、レシピ