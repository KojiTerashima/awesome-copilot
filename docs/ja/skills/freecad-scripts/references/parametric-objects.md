# FreeCAD パラメトリック オブジェクト

FeaturePython オブジェクト、スクリプト化されたオブジェクト、プロパティ、ビュー プロバイダー、およびシリアル化を作成するためのリファレンス ガイド。

## 公式 Wiki リファレンス

- [パラメトリック オブジェクトの作成](https://wiki.freecad.org/Manual:Creating_parametric_objects)
- [FeaturePython オブジェクトの作成パート I](https://wiki.freecad.org/Create_a_FeaturePython_object_part_I)
- [FeaturePython オブジェクトの作成パート II](https://wiki.freecad.org/Create_a_FeaturePython_object_part_II)
- [スクリプト化されたオブジェクト](https://wiki.freecad.org/Scripted_objects)
- [スクリプト化されたオブジェクトの属性保存](https://wiki.freecad.org/Scripted_objects_ Saving_attributes)
- [スクリプト化されたオブジェクトの移行](https://wiki.freecad.org/Scripted_objects_migration)
- [アタッチメント付きのスクリプト化されたオブジェクト](https://wiki.freecad.org/Scripted_objects_with_attachment)
- [ビュープロバイダ](https://wiki.freecad.org/Viewprovider)
- [ツリービューのカスタムアイコン](https://wiki.freecad.org/Custom_icon_in_tree_view)
- [プロパティ](https://wiki.freecad.org/Property)
- [PropertyLink: InList と OutList](https://wiki.freecad.org/PropertyLink:_InList_and_OutList)
- [FeaturePython メソッド](https://wiki.freecad.org/FeaturePython_methods)

## FeaturePython オブジェクト — 完全なテンプレート```python
import FreeCAD
import Part

class MyParametricObject:
    """Proxy class for a custom parametric object."""

    def __init__(self, obj):
        """Initialize and add properties."""
        obj.Proxy = self
        self.Type = "MyParametricObject"

        # Add custom properties
        obj.addProperty("App::PropertyLength", "Length", "Dimensions",
                         "The length of the object").Length = 10.0
        obj.addProperty("App::PropertyLength", "Width", "Dimensions",
                         "The width of the object").Width = 10.0
        obj.addProperty("App::PropertyLength", "Height", "Dimensions",
                         "The height of the object").Height = 5.0
        obj.addProperty("App::PropertyBool", "Chamfered", "Options",
                         "Apply chamfer to edges").Chamfered = False
        obj.addProperty("App::PropertyLength", "ChamferSize", "Options",
                         "Size of chamfer").ChamferSize = 1.0

    def execute(self, obj):
        """Called when the document is recomputed. Build the shape here."""
        shape = Part.makeBox(obj.Length, obj.Width, obj.Height)
        if obj.Chamfered and obj.ChamferSize > 0:
            shape = shape.makeChamfer(obj.ChamferSize, shape.Edges)
        obj.Shape = shape

    def onChanged(self, obj, prop):
        """Called when any property changes."""
        if prop == "Chamfered":
            # Show/hide ChamferSize based on Chamfered toggle
            if obj.Chamfered:
                obj.setPropertyStatus("ChamferSize", "-Hidden")
            else:
                obj.setPropertyStatus("ChamferSize", "Hidden")

    def onDocumentRestored(self, obj):
        """Called when the document is loaded. Re-initialize if needed."""
        self.Type = "MyParametricObject"

    def __getstate__(self):
        """Serialize the proxy (for saving .FCStd)."""
        return {"Type": self.Type}

    def __setstate__(self, state):
        """Deserialize the proxy (for loading .FCStd)."""
        if state:
            self.Type = state.get("Type", "MyParametricObject")
```## ViewProvider — 完全なテンプレート```python
import FreeCADGui
from pivy import coin

class ViewProviderMyObject:
    """Controls how the object appears in the 3D view and tree."""

    def __init__(self, vobj):
        vobj.Proxy = self
        # Add view properties if needed
        # vobj.addProperty("App::PropertyColor", "Color", "Display", "Object color")

    def attach(self, vobj):
        """Called when the view provider is attached to the view object."""
        self.Object = vobj.Object
        self.standard = coin.SoGroup()
        vobj.addDisplayMode(self.standard, "Standard")

    def getDisplayModes(self, vobj):
        """Return available display modes."""
        return ["Standard"]

    def getDefaultDisplayMode(self):
        """Return the default display mode."""
        return "Standard"

    def setDisplayMode(self, mode):
        return mode

    def getIcon(self):
        """Return the icon path for the tree view."""
        return ":/icons/Part_Box.svg"
        # Or return an XPM string, or path to a .svg/.png file

    def updateData(self, obj, prop):
        """Called when the model object's data changes."""
        pass

    def onChanged(self, vobj, prop):
        """Called when a view property changes."""
        pass

    def doubleClicked(self, vobj):
        """Called on double-click in the tree."""
        # Open a task panel, for example
        return True

    def setupContextMenu(self, vobj, menu):
        """Add items to the right-click context menu."""
        action = menu.addAction("My Action")
        action.triggered.connect(lambda: self._myAction(vobj))

    def _myAction(self, vobj):
        FreeCAD.Console.PrintMessage("Context menu action triggered\n")

    def claimChildren(self):
        """Return list of child objects to show in tree hierarchy."""
        # return [self.Object.BaseFeature] if hasattr(self.Object, "BaseFeature") else []
        return []

    def __getstate__(self):
        return None

    def __setstate__(self, state):
        return None
```## オブジェクトの作成```python
def makeMyObject(name="MyObject"):
    """Factory function to create the parametric object."""
    doc = FreeCAD.ActiveDocument
    if doc is None:
        doc = FreeCAD.newDocument()

    obj = doc.addObject("Part::FeaturePython", name)
    MyParametricObject(obj)

    if FreeCAD.GuiUp:
        ViewProviderMyObject(obj.ViewObject)

    doc.recompute()
    return obj

# Usage
obj = makeMyObject("ChamferedBlock")
obj.Length = 20.0
obj.Chamfered = True
FreeCAD.ActiveDocument.recompute()
```## 完全なプロパティ タイプ リファレンス

### 数値プロパティ

|タイプ |パイソン |メモ |
|---|---|---|
| `App::PropertyInteger` | `int` |標準整数 |
| `App::PropertyFloat` | `float` |標準フロート |
| `App::PropertyLength` | `float` |長さの単位 (mm) |
| `App::PropertyDistance` | `float` |距離 (負の値も可能) |
| `App::PropertyAngle` | `float` |角度 (度) |
| `App::PropertyArea` | `float` |ユニットのあるエリア |
| `App::PropertyVolume` | `float` |単位付きの体積 |
| `App::PropertySpeed` | `float` |単位付き速度 |
| `App::PropertyAcceleration` | `float` |加速 |
| `App::PropertyForce` | `float` |力 |
| `App::PropertyPressure` | `float` |圧力 |
| `App::PropertyPercent` | `int` | 0 ～ 100 の整数 |
| `App::PropertyQuantity` | `Quantity` |一般的な単位を意識した値 |
| `App::PropertyIntegerConstraint` | `(val,min,max,step)` |有界整数 |
| `App::PropertyFloatConstraint` | `(val,min,max,step)` |有界浮動小数点 |

### 文字列/パスのプロパティ

|タイプ |パイソン |メモ |
|---|---|---|
| `App::PropertyString` | `str` |テキスト文字列 |
| `App::PropertyFont` | `str` |フォント名 |
| `App::PropertyFile` | `str` |ファイルパス |
| `App::PropertyFileIncluded` | `str` |埋め込みファイル |
| `App::PropertyPath` | `str` |ディレクトリパス |

### ブール値と列挙型

|タイプ |パイソン |メモ |
|---|---|---|
| `App::PropertyBool` | `bool` |真/偽 |
| `App::PropertyEnumeration` | `list`/`str` |落ちる;リストを設定してから値を設定する |```python
# Enumeration usage
obj.addProperty("App::PropertyEnumeration", "Style", "Options", "Style choice")
obj.Style = ["Solid", "Wireframe", "Points"]  # set choices FIRST
obj.Style = "Solid"                              # then set value
```### 幾何学的特性

|タイプ |パイソン |メモ |
|---|---|---|
| `App::PropertyVector` | `FreeCAD.Vector` | 3D ベクトル |
| `App::PropertyVectorList` | `[Vector,...]` |ベクトルのリスト |
| `App::PropertyPlacement` | `Placement` |位置 + 回転 |
| `App::PropertyMatrix` | `Matrix` | 4x4 マトリックス |
| `App::PropertyVectorDistance` | `Vector` |単位を持つベクトル |
| `App::PropertyPosition` | `Vector` |単位付きの位置 |
| `App::PropertyDirection` | `Vector` |方向ベクトル |

### リンクのプロパティ

|タイプ |パイソン |メモ |
|---|---|---|
| `App::PropertyLink` |オブジェクト参照 | 1 つのオブジェクトへのリンク |
| `App::PropertyLinkList` | `[obj,...]` |複数のオブジェクトへのリンク |
| `App::PropertyLinkSub` | `(obj, [subs])` |サブ要素とのリンク |
| `App::PropertyLinkSubList` | `[(obj,[subs]),...]` |複数のリンク+サブ |
| `App::PropertyLinkChild` |オブジェクト参照 |子リンクが要求されました |
| `App::PropertyLinkListChild` | `[obj,...]` |複数の子供を主張 |

### 形状と材質

|タイプ |パイソン |メモ |
|---|---|---|
| `Part::PropertyPartShape` | `Part.Shape` |フルシェイプ |
| `App::PropertyColor` | `(r,g,b)` |カラー (0.0-1.0) |
| `App::PropertyColorList` | `[(r,g,b),...]` |要素ごとの色 |
| `App::PropertyMaterial` | `Material` |材料の定義 |

### コンテナのプロパティ

|タイプ |パイソン |メモ |
|---|---|---|
| `App::PropertyPythonObject` |任意 |シリアル化可能な Python オブジェクト |
| `App::PropertyIntegerList` | `[int,...]` |整数のリスト |
| `App::PropertyFloatList` | `[float,...]` |フロートのリスト |
| `App::PropertyStringList` | `[str,...]` |文字列のリスト |
| `App::PropertyBoolList` | `[bool,...]` |ブール値のリスト |
| `App::PropertyMap` | `{str:str}` |文字列辞書 |

## オブジェクトの依存関係の追跡```python
# InList: objects that reference this object
obj.InList          # [objects referencing obj]
obj.InListRecursive # all ancestors

# OutList: objects this object references
obj.OutList         # [objects obj references]
obj.OutListRecursive # all descendants
```## バージョン間の移行```python
class MyParametricObject:
    # ... existing code ...

    def onDocumentRestored(self, obj):
        """Handle version migration when document loads."""
        # Add properties that didn't exist in older versions
        if not hasattr(obj, "NewProp"):
            obj.addProperty("App::PropertyFloat", "NewProp", "Group", "Tip")
            obj.NewProp = default_value

        # Rename properties (copy value, remove old)
        if hasattr(obj, "OldPropName"):
            if not hasattr(obj, "NewPropName"):
                obj.addProperty("App::PropertyFloat", "NewPropName", "Group", "Tip")
                obj.NewPropName = obj.OldPropName
            obj.removeProperty("OldPropName")
```## 添付ファイルのサポート```python
import Part

class MyAttachableObject:
    def __init__(self, obj):
        obj.Proxy = self
        obj.addExtension("Part::AttachExtensionPython")

    def execute(self, obj):
        # The attachment sets the Placement automatically
        if not obj.MapPathParameter:
            obj.positionBySupport()
        # Build your shape at the origin; Placement handles positioning
        obj.Shape = Part.makeBox(10, 10, 10)
```
