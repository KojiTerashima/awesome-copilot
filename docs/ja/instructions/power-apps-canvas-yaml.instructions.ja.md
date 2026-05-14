---
description: 'Microsoft Power Apps YAML schema v3.0 に基づく Power Apps Canvas Apps YAML structure の包括的ガイド。Power Fx formula、control structure、data type、source control のベスト プラクティスを扱います。'
applyTo: '**/*.{yaml,yml,md,pa.yaml}'
---

# Power Apps Canvas Apps YAML 構造ガイド

## 概要
この文書は、公式 Microsoft Power Apps YAML schema (v3.0) と Power Fx documentation に基づいて、Power Apps canvas app の YAML code を扱うための包括的なガイドを提供します。

**公式 Schema Source**: https://raw.githubusercontent.com/microsoft/PowerApps-Tooling/refs/heads/master/schemas/pa-yaml/v3.0/pa.schema.yaml

## Power Fx の設計原則
Power Fx は Power Apps canvas app 全体で使われる formula language です。次の中核原則に従います。

### 設計原則
- **Simple**: Excel formula に馴染みのある概念を使う
- **Excel Consistency**: Excel formula の syntax と behavior に整合する
- **Declarative**: どう実現するかではなく、何をしたいかを記述する
- **Functional**: 副作用を避ける。ほとんどの function は pure
- **Composition**: より単純な function を組み合わせて複雑な logic を構築する
- **Strongly Typed**: type system が data integrity を保証する
- **Integrated**: Power Platform 全体で自然に連携する

### 言語哲学
Power Fx は次を促進します:
- 馴染みのある Excel 風 formula による low-code 開発
- dependency 変更時の自動再計算
- compile time checking を備えた型安全性
- functional programming pattern

## ルート構造
すべての Power Apps YAML file は次の top-level structure に従います。

```yaml
App:
  Properties:
    # App-level properties and formulas
    StartScreen: =Screen1

Screens:
  # Screen definitions

ComponentDefinitions:
  # Custom component definitions

DataSources:
  # Data source configurations

EditorState:
  # Editor metadata (screen order, etc.)
```

## 1. App セクション
`App` section は application level の property と configuration を定義します。

```yaml
App:
  Properties:
    StartScreen: =Screen1
    BackEnabled: =false
    # Other app properties with Power Fx formulas
```

### 重要ポイント:
- application 全体の設定を含みます
- property は Power Fx formula (`=` で始まる) を使います
- `StartScreen` property はよく指定されます

## 2. Screens セクション
application 内のすべての screen を unordered map として定義します。

```yaml
Screens:
  Screen1:
    Properties:
      # Screen properties
    Children:
      - Label1:
          Control: Label
          Properties:
            Text: ="Hello World"
            X: =10
            Y: =10
      - Button1:
          Control: Button
          Properties:
            Text: ="Click Me"
            X: =10
            Y: =100
```

### Screen 構造:
- **Properties**: screen level の property と formula
- **Children**: screen 上の control の配列 (z-index 順)

### Control 定義フォーマット:
```yaml
ControlName:
  Control: ControlType      # Required: Control type identifier
  Properties:
    PropertyName: =PowerFxFormula
  # Optional properties:
  Group: GroupName          # For organizing controls in Studio
  Variant: VariantName      # Control variant (affects default properties)
  MetadataKey: Key          # Metadata identifier for control
  Layout: LayoutName        # Layout configuration
  IsLocked: true/false      # Whether control is locked in editor
  Children: []              # For container controls (ordered by z-index)
```

### Control のバージョン指定:
`@` operator を使って control version を指定できます。
```yaml
MyButton:
  Control: Button@2.1.0     # Specific version
  Properties:
    Text: ="Click Me"

MyLabel:
  Control: Label            # Uses latest version by default
  Properties:
    Text: ="Hello World"
```

## 3. Control Type

### 標準 Control
一般的な first-party control は次のとおりです。
- **Basic Controls**: `Label`, `Button`, `TextInput`, `HTMLText`
- **Input Controls**: `Slider`, `Toggle`, `Checkbox`, `Radio`, `Dropdown`, `Combobox`, `DatePicker`, `ListBox`
- **Display Controls**: `Image`, `Icon`, `Video`, `Audio`, `PDF viewer`, `Barcode scanner`
- **Layout Controls**: `Container`, `Rectangle`, `Circle`, `Gallery`, `DataTable`, `Form`
- **Chart Controls**: `Column chart`, `Line chart`, `Pie chart`
- **Advanced Controls**: `Timer`, `Camera`, `Microphone`, `Add picture`, `Import`, `Export`

### Container と Layout Control
container control とその child には特に注意してください。
```yaml
MyContainer:
  Control: Container
  Properties:
    Width: =300
    Height: =200
    Fill: =RGBA(240, 240, 240, 1)
  Children:
    - Label1:
        Control: Label
        Properties:
          Text: ="Inside Container"
          X: =10         # Relative to container
          Y: =10         # Relative to container
    - Button1:
        Control: Button
        Properties:
          Text: ="Container Button"
          X: =10
          Y: =50
```

### Custom Component
```yaml
MyCustomControl:
  Control: Component
  ComponentName: MyComponent
  Properties:
    X: =10
    Y: =10
    # Custom component properties
```

### Code Component (PCF)
```yaml
MyPCFControl:
  Control: CodeComponent
  ComponentName: publisherprefix_namespace.classname
  Properties:
    X: =10
    Y: =10
```

## 4. Component Definitions
再利用可能な custom component を定義します。

```yaml
ComponentDefinitions:
  MyComponent:
    DefinitionType: CanvasComponent
    Description: "A reusable component"
    AllowCustomization: true
    AccessAppScope: false
    CustomProperties:
      InputText:
        PropertyKind: Input
        DataType: Text
        Description: "Input text property"
        Default: ="Default Value"
      OutputValue:
        PropertyKind: Output
        DataType: Number
        Description: "Output number value"
    Properties:
      Fill: =RGBA(255, 255, 255, 1)
      Height: =100
      Width: =200
    Children:
      - Label1:
          Control: Label
          Properties:
            Text: =Parent.InputText
```

### Custom Property Type:
- **Input**: 親から値を受け取る
- **Output**: 親へ値を返す
- **InputFunction**: 親から呼ばれる function
- **OutputFunction**: component 内で定義される function
- **Event**: 親へ event を発火する
- **Action**: 副作用を伴う function

### Data Type:
- `Text`, `Number`, `Boolean`
- `DateAndTime`, `Color`, `Currency`
- `Record`, `Table`, `Image`
- `VideoOrAudio`, `Screen`

## 5. DataSources
data connection を構成します。

```yaml
DataSources:
  MyTable:
    Type: Table
    Parameters:
      TableLogicalName: account

  MyActions:
    Type: Actions
    ConnectorId: shared_office365users
    Parameters:
      # Additional connector parameters
```

### DataSource Type:
- **Table**: Dataverse table またはその他の tabular data
- **Actions**: connector action と flow

## 6. EditorState
editor 上の整理状態を保持します。

```yaml
EditorState:
  ScreensOrder:
    - Screen1
    - Screen2
    - Screen3
  ComponentDefinitionsOrder:
    - MyComponent
    - AnotherComponent
```

## Power Fx Formula ガイドライン

### Formula の構文:
- すべての formula は `=` で始めます
- expression には Power Fx syntax を使います
- null value は `null` (引用符なし) で表現できます
- 例:
  ```yaml
  Text: ="Hello World"
  X: =10
  Visible: =Toggle1.Value
  OnSelect: =Navigate(Screen2, ScreenTransition.Fade)
  OptionalProperty: null    # Represents no value
  ```

### よくある Formula パターン:
```yaml
# Static values
Text: ="Static Text"
X: =50
Visible: =true

# Control references
Text: =TextInput1.Text
Visible: =Toggle1.Value

# Parent references (for controls in containers/galleries)
Width: =Parent.Width - 20
Height: =Parent.TemplateHeight    # In gallery templates

# Functions
OnSelect: =Navigate(NextScreen, ScreenTransition.Slide)
Text: =Concatenate("Hello ", User().FullName)

# Conditional logic
Visible: =If(Toggle1.Value, true, false)
Fill: =If(Button1.Pressed, RGBA(255,0,0,1), RGBA(0,255,0,1))

# Data operations
Items: =Filter(DataSource, Status = "Active")
Text: =LookUp(Users, ID = 123).Name
```

### Z-Index と Control の並び順:
- `Children` 配列内の control は z-index 順に並びます
- 配列の最初の control = 最下層 (z-index 1)
- 配列の最後の control = 最上層 (最大 z-index)
- すべての control は 1 からの昇順を使います

## 命名規則

### Entity 名:
- screen 名: わかりやすく一意にする
- control 名: TypeName + Number (例: `Button1`, `Label2`)
- component 名: PascalCase

### Property 名:
- 標準 property: schema の厳密な casing を使う
- custom property: PascalCase 推奨

## ベスト プラクティス

### 1. 構造の整理:
- screen を論理的に整理します
- 関連する control は `Group` property でまとめます
- すべての entity に意味のある名前を付けます

### 2. Formula の記述:
- formula は読みやすく整形します
- 複雑な formula では可能であれば comment を使います
- 過度に複雑な入れ子 expression は避けます

### 3. Component 設計:
- 再利用しやすい component を設計します
- custom property には明確な description を付けます
- 適切な property kind (Input / Output) を使います

### 4. DataSource 管理:
- data source には説明的な名前を付けます
- connection 要件を document 化します
- data source の構成は最小限に保ちます

## 検証ルール

### 必須 Property:
- すべての control は `Control` property を持つ必要があります
- component definition は `DefinitionType` を持つ必要があります
- data source は `Type` を持つ必要があります

### 命名パターン:
- entity 名: 1 文字以上、英数字
- control type ID: `^([A-Z][a-zA-Z0-9]*/)?[A-Z][a-zA-Z0-9]*(@\d+\.\d+\.\d+)?$` に従う
- code component 名: `^([a-z][a-z0-9]{1,7})_([a-zA-Z0-9]\.)+[a-zA-Z0-9]+$` に従う

## よくある問題と解決策

### 1. 無効な Control Type:
- control type の綴りが正しいことを確認します
- casing が正しいことを確認します
- schema でサポートされている control type か確認します

### 2. Formula Error:
- すべての formula は `=` で始めます
- 正しい Power Fx syntax を使います
- property reference が正しいか確認します

### 3. 構造の検証:
- YAML indentation を正しく保ちます
- 必須 property が存在することを確認します
- schema structure に厳密に従います

### 4. Custom Component の問題:
- `ComponentName` が definition と一致することを確認します
- custom property が正しく定義されていることを確認します
- property kind が適切であることを確認します
- external component を使う場合は component library reference を検証します

### 5. パフォーマンス上の考慮事項:
- YAML 内で deeply nested な formula を避けます
- 効率的な data source query を使います
- 大規模 data set には delegable formula を検討します
- 頻繁に更新される property の複雑な計算を最小化します

## 高度なトピック

### 1. Component Library Integration:
```yaml
ComponentDefinitions:
  MyLibraryComponent:
    DefinitionType: CanvasComponent
    AllowCustomization: true
    ComponentLibraryUniqueName: "pub_MyComponentLibrary"
    # Component definition details
```

### 2. Responsive Design の考慮事項:
- responsive sizing には `Parent.Width` と `Parent.Height` を使います
- 複雑な UI では container ベースの layout を検討します
- 動的な配置と sizing に formula を使います

### 3. Gallery Template:
```yaml
MyGallery:
  Control: Gallery
  Properties:
    Items: =DataSource
    TemplateSize: =100
  Children:
    - GalleryTemplate:  # Template for each gallery item
        Children:
          - TitleLabel:
              Control: Label
              Properties:
                Text: =ThisItem.Title
                Width: =Parent.TemplateWidth - 20
```

### 4. Form Control と Data Card:
```yaml
MyForm:
  Control: Form
  Properties:
    DataSource: =DataSource
    DefaultMode: =FormMode.New
  Children:
    - DataCard1:
        Control: DataCard
        Properties:
          DataField: ="Title"
        Children:
          - DataCardValue1:
              Control: TextInput
              Properties:
                Default: =Parent.Default
```

### 5. Formula 内の Error Handling:
```yaml
Properties:
  Text: =IfError(LookUp(DataSource, ID = 123).Name, "Not Found")
  Visible: =!IsError(DataSource)
  OnSelect: =IfError(
    Navigate(DetailScreen, ScreenTransition.Cover),
    Notify("Navigation failed", NotificationType.Error)
  )
```

## Power Apps Source Code 管理

### Source Code File へのアクセス:
Power Apps YAML file はいくつかの方法で取得できます。

1. **Power Platform CLI**:
   ```powershell
   # List canvas apps in environment
   pac canvas list

   # Download and extract YAML files
   pac canvas download --name "MyApp" --extract-to-directory "C:\path\to\destination"
   ```

2. **.msapp からの手動展開**:
   ```powershell
   # Extract .msapp file using PowerShell
   Expand-Archive -Path "C:\path\to\yourFile.msapp" -DestinationPath "C:\path\to\destination"
   ```

3. **Dataverse Git Integration**: .msapp file を介さず source file に直接アクセス

### .msapp 内の File 構造:
- `\src\App.pa.yaml` - main App configuration を表す
- `\src\[ScreenName].pa.yaml` - 各 screen ごとに 1 file
- `\src\Component\[ComponentName].pa.yaml` - component definition

**重要な注意点**:
- source control 対象として意図されているのは `\src` folder 内の file のみです
- .pa.yaml file は **read-only** で、レビュー専用です
- 外部編集、merge、conflict resolution はサポートされません
- .msapp 内の JSON file は source control に対して安定していません

### Schema Version の変遷:
1. **Experimental Format** (*.fx.yaml): もはや開発されていません
2. **Early Preview**: 一時的な format で、現在は未使用です
3. **Source Code** (*.pa.yaml): version control をサポートする現在の active format

## Power Fx Formula Reference

### Formula カテゴリ:

#### **Function**: parameter を受け取り、処理を行い、value を返す
```yaml
Properties:
  Text: =Concatenate("Hello ", User().FullName)
  X: =Sum(10, 20, 30)
  Items: =Filter(DataSource, Status = "Active")
```

#### **Signal**: 環境情報を返す (parameter なし)
```yaml
Properties:
  Text: =Location.Latitude & ", " & Location.Longitude
  Visible: =Connection.Connected
  Color: =If(Acceleration.X > 5, Color.Red, Color.Blue)
```

#### **Enumeration**: 定義済みの定数値
```yaml
Properties:
  Fill: =Color.Blue
  Transition: =ScreenTransition.Fade
  Align: =Align.Center
```

#### **Named Operator**: container 情報へアクセスする
```yaml
Properties:
  Text: =ThisItem.Title        # In galleries
  Width: =Parent.Width - 20    # In containers
  Height: =Self.Height / 2     # Self-reference
```

### YAML で重要な Power Fx Function:

#### **Navigation と App Control**:
```yaml
OnSelect: =Navigate(NextScreen, ScreenTransition.Cover)
OnSelect: =Back()
OnSelect: =Exit()
OnSelect: =Launch("https://example.com")
```

#### **Data Operation**:
```yaml
Items: =Filter(DataSource, Category = "Active")
Text: =LookUp(Users, ID = 123).Name
OnSelect: =Patch(DataSource, ThisItem, {Status: "Complete"})
OnSelect: =Collect(LocalCollection, {Name: TextInput1.Text})
```

#### **Conditional Logic**:
```yaml
Visible: =If(Toggle1.Value, true, false)
Text: =Switch(Status, "New", "🆕", "Complete", "✅", "❓")
Fill: =If(Value < 0, Color.Red, Color.Green)
```

#### **Text Manipulation**:
```yaml
Text: =Concatenate("Hello ", User().FullName)
Text: =Upper(TextInput1.Text)
Text: =Substitute(Label1.Text, "old", "new")
Text: =Left(Title, 10) & "..."
```

#### **Mathematical Operation**:
```yaml
Text: =Sum(Sales[Amount])
Text: =Average(Ratings[Score])
Text: =Round(Calculation, 2)
Text: =Max(Values[Number])
```

#### **Date と Time Function**:
```yaml
Text: =Text(Now(), "mm/dd/yyyy")
Text: =DateDiff(StartDate, EndDate, Days)
Text: =Text(Today(), "dddd, mmmm dd, yyyy")
Visible: =IsToday(DueDate)
```

### Formula 構文ガイドライン:

#### **基本構文ルール**:
- すべての formula は `=` で始まります
- Excel と違って、前置の `+` や追加の `=` は不要です
- text string には double quote を使います: `="Hello World"`
- property reference: `ControlName.PropertyName`
- YAML context では comment はサポートされません

#### **Formula 要素**:
```yaml
# Literal values
Text: ="Static Text"
X: =42
Visible: =true

# Control property references
Text: =TextInput1.Text
Visible: =Checkbox1.Value

# Function calls
Text: =Upper(TextInput1.Text)
Items: =Sort(DataSource, Title)

# Complex expressions
Text: =If(IsBlank(TextInput1.Text), "Enter text", Upper(TextInput1.Text))
```

#### **Behavior Formula と Property Formula**:
```yaml
# Property formulas (calculate values)
Properties:
  Text: =Concatenate("Hello ", User().FullName)
  Visible: =Toggle1.Value

# Behavior formulas (perform actions - use semicolon for multiple actions)
Properties:
  OnSelect: =Set(MyVar, true); Navigate(NextScreen); Notify("Done!")
```

### 高度な Formula パターン:

#### **Collection の利用**:
```yaml
Properties:
  Items: =Filter(MyCollection, Status = "Active")
  OnSelect: =ClearCollect(MyCollection, DataSource)
  OnSelect: =Collect(MyCollection, {Name: "New Item", Status: "Active"})
```

#### **Error Handling**:
```yaml
Properties:
  Text: =IfError(Value(TextInput1.Text), 0)
  OnSelect: =IfError(
    Patch(DataSource, ThisItem, {Field: Value}),
    Notify("Error updating record", NotificationType.Error)
  )
```

#### **動的 Property 設定**:
```yaml
Properties:
  Fill: =ColorValue("#" & HexInput.Text)
  Height: =Parent.Height * (Slider1.Value / 100)
  X: =If(Alignment = "Center", (Parent.Width - Self.Width) / 2, 0)
```

## Formula を扱う際のベスト プラクティス

### Formula の整理:
- 複雑な formula は小さく読みやすい部分に分けます
- 中間計算の保存には variable を使います
- 複雑な logic は説明的な control 名で読みやすくします
- 関連する計算をまとめます

### パフォーマンス最適化:
- 大規模 data set では delegation friendly な function を使います
- 頻繁に更新される property で nested function call を避けます
- 複雑な data transformation には collection を使います
- external data source 呼び出しを最小化します

## Power Fx Data Type と Operation

### Data Type カテゴリ:

#### **Primitive Type**:
- **Boolean**: `=true`, `=false`
- **Number**: `=123`, `=45.67`
- **Text**: `="Hello World"`
- **Date**: `=Date(2024, 12, 25)`
- **Time**: `=Time(14, 30, 0)`
- **DateTime**: `=Now()`

#### **Complex Type**:
- **Color**: `=Color.Red`, `=RGBA(255, 128, 0, 1)`
- **Record**: `={Name: "John", Age: 30}`
- **Table**: `=Table({Name: "John"}, {Name: "Jane"})`
- **GUID**: `=GUID()`

#### **型変換**:
```yaml
Properties:
  Text: =Text(123.45, "#,##0.00")        # Number to text
  Text: =Value("123.45")                 # Text to number
  Text: =DateValue("12/25/2024")         # Text to date
  Visible: =Boolean("true")              # Text to boolean
```

#### **型チェック**:
```yaml
Properties:
  Visible: =Not(IsBlank(OptionalField))
  Visible: =Not(IsError(Value(TextInput1.Text)))
  Visible: =IsNumeric(TextInput1.Text)
```

### Table Operation:

#### **Table の作成**:
```yaml
Properties:
  Items: =Table(
    {Name: "Product A", Price: 10.99},
    {Name: "Product B", Price: 15.99}
  )
  Items: =["Option 1", "Option 2", "Option 3"]  # Single-column table
```

#### **Filter と Sort**:
```yaml
Properties:
  Items: =Filter(Products, Price > 10)
  Items: =Sort(Products, Name, Ascending)
  Items: =SortByColumns(Products, "Price", Descending, "Name", Ascending)
```

#### **Data Transformation**:
```yaml
Properties:
  Items: =AddColumns(Products, "Total", Price * Quantity)
  Items: =RenameColumns(Products, "Price", "Cost")
  Items: =ShowColumns(Products, "Name", "Price")
  Items: =DropColumns(Products, "InternalID")
```

#### **集計**:
```yaml
Properties:
  Text: =Sum(Products, Price)
  Text: =Average(Products, Rating)
  Text: =Max(Products, Price)
  Text: =CountRows(Products)
```

### Variable と State 管理:

#### **Global Variable**:
```yaml
Properties:
  OnSelect: =Set(MyGlobalVar, "Hello World")
  Text: =MyGlobalVar
```

#### **Context Variable**:
```yaml
Properties:
  OnSelect: =UpdateContext({LocalVar: "Screen Specific"})
  OnSelect: =Navigate(NextScreen, None, {PassedValue: 42})
```

#### **Collection**:
```yaml
Properties:
  OnSelect: =ClearCollect(MyCollection, DataSource)
  OnSelect: =Collect(MyCollection, {Name: "New Item"})
  Items: =MyCollection
```

## Power Fx Enhanced Connector と External Data

### Connector Integration:
```yaml
DataSources:
  SharePointList:
    Type: Table
    Parameters:
      TableLogicalName: "Custom List"

  Office365Users:
    Type: Actions
    ConnectorId: shared_office365users
```

### External Data の利用:
```yaml
Properties:
  Items: =Filter(SharePointList, Status = "Active")
  OnSelect: =Office365Users.SearchUser({searchTerm: SearchInput.Text})
```

### Delegation の考慮事項:
```yaml
Properties:
  # Delegable operations (executed server-side)
  Items: =Filter(LargeTable, Status = "Active")    # Efficient

  # Non-delegable operations (may download all records)
  Items: =Filter(LargeTable, Len(Description) > 100)  # Warning issued
```

## トラブルシューティングと共通パターン

### よくある Error パターン:
```yaml
# Handle blank values
Properties:
  Text: =If(IsBlank(OptionalText), "Default", OptionalText)

# Handle errors gracefully
Properties:
  Text: =IfError(RiskyOperation(), "Fallback Value")

# Validate input
Properties:
  Visible: =And(
    Not(IsBlank(NameInput.Text)),
    IsNumeric(AgeInput.Text),
    IsMatch(EmailInput.Text, Email)
  )
```

### パフォーマンス最適化:
```yaml
# Efficient data loading
Properties:
  Items: =Filter(LargeDataSource, Status = "Active")    # Server-side filtering

# Use delegation-friendly operations
Properties:
  Items: =Sort(Filter(DataSource, Active), Name)        # Delegable
  # Avoid: Sort(DataSource, If(Active, Name, ""))       # Not delegable
```

### メモリ管理:
```yaml
# Clear unused collections
Properties:
  OnSelect: =Clear(TempCollection)

# Limit data retrieval
Properties:
  Items: =FirstN(Filter(DataSource, Status = "Active"), 50)
```

このガイドは、Power Apps Canvas Apps YAML structure と Power Fx formula を包括的に扱います。常に公式 schema で YAML を検証し、Power Apps Studio 環境で formula を test してください。
