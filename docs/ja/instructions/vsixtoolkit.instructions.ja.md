---
description: 'Community.VisualStudio.Toolkit を使った Visual Studio extension (VSIX) 開発のためのガイドライン'
applyTo: '**/*.cs, **/*.vsct, **/*.xaml, **/source.extension.vsixmanifest'
---

# Community.VisualStudio.Toolkit を使う Visual Studio Extension 開発

## 適用範囲

**これらの指示は、`Community.VisualStudio.Toolkit` を使う Visual Studio extension にのみ適用する。**

project が toolkit を使っているかは次を確認する:
- `Community.VisualStudio.Toolkit.*` NuGet package reference
- `ToolkitPackage` base class (`AsyncPackage` を直接使わない)
- command に `BaseCommand<T>` pattern を使っている

**project が raw VSSDK (`AsyncPackage` を直接使用) または新しい `VisualStudio.Extensibility` model を使っている場合は、この instruction を適用しない。**

## 目標

- async-first で thread-safe な extension code を生成する
- toolkit の抽象化 (`VS.*` helper、`BaseCommand<T>`、`BaseOptionModel<T>`) を使う
- すべての UI が Visual Studio theme を尊重する
- VSSDK と VSTHRD analyzer rule に従う
- テストしやすく保守しやすい extension code を作る
- repository に存在する場合は **`.editorconfig` 設定に従う**

## Code Style (.editorconfig)

**repository に `.editorconfig` file がある場合、生成・修正するすべての code はその rule に従わなければならない。**

これには次を含むが、これに限らない:
- indent style (tab か space か) と size
- line ending と final newline 要件
- 命名規則 (field、property、method など)
- code style preference (`var` の使い方、expression body、brace など)
- analyzer の severity level と suppression

code を生成する前に repository root の `.editorconfig` を確認し、その設定を適用する。迷った場合は、編集中 file の周辺 style に合わせる。

## .NET Framework と C# Language 制約

**Visual Studio extension は .NET Framework 4.8 を対象にする** が、.NET Framework runtime の制約内であれば最新の C# 構文 (C# 14 まで) を使える。

### ✅ 利用可能な最新 C# 機能
- primary constructor
- file-scoped namespace
- global using
- pattern matching (すべての形式)
- record (一部制限あり)
- `init` accessor
- target-typed `new`
- nullable reference type (annotation のみ)
- raw string literal
- collection expression

### ❌ 非対応 (.NET Framework の制限)
- `Span<T>`、`ReadOnlySpan<T>`、`Memory<T>` (runtime support なし)
- `IAsyncEnumerable<T>` (polyfill package なしでは不可)
- default interface implementation
- `Index` と `Range` 型 (`^` と `..` 演算子の runtime support なし)
- struct 上の `init`-only setter (runtime 制限)
- 一部の `System.Text.Json` 機能

### ベストプラクティス
code を書くときは、.NET Framework 4.8 で利用できる API を優先する。最新 API が必要な場合は、polyfill NuGet package が存在するか確認する (例: `IAsyncEnumerable<T>` 用の `Microsoft.Bcl.AsyncInterfaces`)。

## Prompt 動作例

### ✅ 良い提案
- "Create a command that opens the current file's containing folder using `BaseCommand<T>`"
- "Add an options page with a boolean setting using `BaseOptionModel<T>`"
- "Write a tagger provider for C# files that highlights TODO comments"
- "Show a status bar progress indicator while processing files"

### ❌ 避けること
- `ToolkitPackage` ではなく raw `AsyncPackage` を提案すること
- `BaseCommand<T>` ではなく `OleMenuCommandService` を直接使うこと
- UI thread へ切り替えずに WPF 要素を作成すること
- UI 作業に `.Result`、`.Wait()`、`Task.Run` を使うこと
- VS theme color を使わず色をハードコードすること

## Project 構成

```
src/
├── Commands/           # Command handler (menu item、toolbar button)
├── Options/            # Setting / options page
├── Services/           # Business logic と service
├── Tagging/            # ITagger 実装 (syntax highlighting、outlining)
├── Adornments/         # Editor adornment (IntraTextAdornment、margin)
├── QuickInfo/          # QuickInfo / tooltip provider
├── SuggestedActions/   # Light bulb action
├── Handlers/           # Event handler (format document、paste など)
├── Resources/          # 画像、icon、license file
├── source.extension.vsixmanifest  # Extension manifest
├── VSCommandTable.vsct            # Command 定義 (menu、button)
├── VSCommandTable.cs              # 自動生成 command ID
└── *Package.cs                    # Main package class
```

## Community.VisualStudio.Toolkit Pattern

### Global Usings

toolkit を使う extension では、Package file に次の global using を持つべきである:

```csharp
global using System;
global using Community.VisualStudio.Toolkit;
global using Microsoft.VisualStudio.Shell;
global using Task = System.Threading.Tasks.Task;
```

### Package Class

```csharp
[PackageRegistration(UseManagedResourcesOnly = true, AllowsBackgroundLoading = true)]
[InstalledProductRegistration(Vsix.Name, Vsix.Description, Vsix.Version)]
[ProvideMenuResource("Menus.ctmenu", 1)]
[Guid(PackageGuids.YourExtensionString)]
[ProvideOptionPage(typeof(OptionsProvider.GeneralOptions), Vsix.Name, "General", 0, 0, true, SupportsProfiles = true)]
public sealed class YourPackage : ToolkitPackage
{
    protected override async Task InitializeAsync(CancellationToken cancellationToken, IProgress<ServiceProgressData> progress)
    {
        await this.RegisterCommandsAsync();
    }
}
```

### Command

command は `[Command]` attribute を使い、`BaseCommand<T>` を継承する:

```csharp
[Command(PackageIds.YourCommandId)]
internal sealed class YourCommand : BaseCommand<YourCommand>
{
    protected override async Task ExecuteAsync(OleMenuCmdEventArgs e)
    {
        // Command 実装
    }

    // 任意: command state (enabled、checked、visible) を制御する
    protected override void BeforeQueryStatus(EventArgs e)
    {
        Command.Checked = someCondition;
        Command.Enabled = anotherCondition;
    }
}
```

### Options Page

```csharp
internal partial class OptionsProvider
{
    [ComVisible(true)]
    public class GeneralOptions : BaseOptionPage<General> { }
}

public class General : BaseOptionModel<General>
{
    [Category("Category Name")]
    [DisplayName("Setting Name")]
    [Description("設定の説明。")]
    [DefaultValue(true)]
    public bool MySetting { get; set; } = true;
}
```

## MEF Component

### Tagger Provider

`[Export]` と適切な `[ContentType]` attribute を使う:

```csharp
[Export(typeof(IViewTaggerProvider))]
[ContentType("CSharp")]
[ContentType("Basic")]
[TagType(typeof(IntraTextAdornmentTag))]
[TextViewRole(PredefinedTextViewRoles.Document)]
internal sealed class YourTaggerProvider : IViewTaggerProvider
{
    [Import]
    internal IOutliningManagerService OutliningManagerService { get; set; }

    public ITagger<T> CreateTagger<T>(ITextView textView, ITextBuffer buffer) where T : ITag
    {
        if (textView == null || !(textView is IWpfTextView wpfTextView))
            return null;

        if (textView.TextBuffer != buffer)
            return null;

        return wpfTextView.Properties.GetOrCreateSingletonProperty(
            () => new YourTagger(wpfTextView)) as ITagger<T>;
    }
}
```

### QuickInfo Source

```csharp
[Export(typeof(IAsyncQuickInfoSourceProvider))]
[Name("YourQuickInfo")]
[ContentType("code")]
[Order(Before = "Default Quick Info Presenter")]
internal sealed class YourQuickInfoSourceProvider : IAsyncQuickInfoSourceProvider
{
    public IAsyncQuickInfoSource TryCreateQuickInfoSource(ITextBuffer textBuffer)
    {
        return textBuffer.Properties.GetOrCreateSingletonProperty(
            () => new YourQuickInfoSource(textBuffer));
    }
}
```

### Suggested Action (Light Bulb)

```csharp
[Export(typeof(ISuggestedActionsSourceProvider))]
[Name("Your Suggested Actions")]
[ContentType("text")]
internal sealed class YourSuggestedActionsSourceProvider : ISuggestedActionsSourceProvider
{
    public ISuggestedActionsSource CreateSuggestedActionsSource(ITextView textView, ITextBuffer textBuffer)
    {
        return new YourSuggestedActionsSource(textView, textBuffer);
    }
}
```

## Threading Guideline

### WPF 操作では常に UI thread に切り替える

```csharp
await ThreadHelper.JoinableTaskFactory.SwitchToMainThreadAsync(cancellationToken);
// ここから WPF 要素の作成 / 変更が安全になる
```

### Background 作業

```csharp
ThreadHelper.JoinableTaskFactory.RunAsync(async () =>
{
    await ThreadHelper.JoinableTaskFactory.SwitchToMainThreadAsync();
    await VS.Commands.ExecuteAsync("View.TaskList");
});
```

## VSSDK と Threading Analyzer Rule

extension は次の analyzer rule を強制するべきである。.editorconfig に追加する:

```ini
dotnet_diagnostic.VSSDK*.severity = error
dotnet_diagnostic.VSTHRD*.severity = error
```

### Performance Rule
| ID | Rule | Fix |
|----|------|-----|
| **VSSDK001** | `AsyncPackage` を継承する | `ToolkitPackage` を使う (AsyncPackage を継承している) |
| **VSSDK002** | `AllowsBackgroundLoading = true` | `[PackageRegistration]` に追加する |

### Threading Rule (VSTHRD)
| ID | Rule | Fix |
|----|------|-----|
| **VSTHRD001** | `.Wait()` を避ける | `await` を使う |
| **VSTHRD002** | `JoinableTaskFactory.Run` を避ける | `RunAsync` または `await` を使う |
| **VSTHRD010** | COM call は UI thread が必要 | `await ThreadHelper.JoinableTaskFactory.SwitchToMainThreadAsync()` |
| **VSTHRD100** | `async void` を使わない | `async Task` を使う |
| **VSTHRD110** | async result を観測する | `await task;` または pragma で suppress |

## Visual Studio Theming

**すべての UI は VS theme (Light、Dark、Blue、High Contrast) を尊重しなければならない**

### Environment Color を使う WPF Theming

```xml
<!-- MyControl.xaml -->
<UserControl x:Class="MyExt.MyControl"
             xmlns:vsui="clr-namespace:Microsoft.VisualStudio.PlatformUI;assembly=Microsoft.VisualStudio.Shell.15.0">
    <Grid Background="{DynamicResource {x:Static vsui:EnvironmentColors.ToolWindowBackgroundBrushKey}}">
        <TextBlock Foreground="{DynamicResource {x:Static vsui:EnvironmentColors.ToolWindowTextBrushKey}}"
                   Text="Hello, themed world!" />
    </Grid>
</UserControl>
```

### Toolkit による自動 Theming (推奨)

toolkit は WPF UserControl に自動 theming を提供する:

```xml
<UserControl x:Class="MyExt.MyUserControl"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:toolkit="clr-namespace:Community.VisualStudio.Toolkit;assembly=Community.VisualStudio.Toolkit"
             toolkit:Themes.UseVsTheme="True">
    <!-- Control は自動的に VS styling を取得する -->
</UserControl>
```

dialog window には `DialogWindow` を使う:

```xml
<platform:DialogWindow
    x:Class="MyExt.MyDialog"
    xmlns:platform="clr-namespace:Microsoft.VisualStudio.PlatformUI;assembly=Microsoft.VisualStudio.Shell.15.0"
    xmlns:toolkit="clr-namespace:Community.VisualStudio.Toolkit;assembly=Community.VisualStudio.Toolkit"
    toolkit:Themes.UseVsTheme="True">
</platform:DialogWindow>
```

### よく使う Theme Color Token

| Category | Token | Usage |
|----------|-------|-------|
| **Background** | `EnvironmentColors.ToolWindowBackgroundBrushKey` | Window / panel background |
| **Foreground** | `EnvironmentColors.ToolWindowTextBrushKey` | Text |
| **Command Bar** | `EnvironmentColors.CommandBarTextActiveBrushKey` | Menu item |
| **Links** | `EnvironmentColors.ControlLinkTextBrushKey` | Hyperlink |

### Theme-Aware Icon

VS Image Catalog の `KnownMonikers` を使って theme-aware icon を使う:

```csharp
public ImageMoniker IconMoniker => KnownMonikers.Settings;
```

VSCT では:
```xml
<Icon guid="ImageCatalogGuid" id="Settings"/>
<CommandFlag>IconIsMoniker</CommandFlag>
```

## よく使う VS SDK API

### VS Helper Method (Community.VisualStudio.Toolkit)

```csharp
// Status bar
await VS.StatusBar.ShowMessageAsync("Message");
await VS.StatusBar.ShowProgressAsync("Working...", currentStep, totalSteps);

// Solution / Project
Solution solution = await VS.Solutions.GetCurrentSolutionAsync();
IEnumerable<SolutionItem> items = await VS.Solutions.GetActiveItemsAsync();
bool isOpen = await VS.Solutions.IsOpenAsync();

// Document
DocumentView docView = await VS.Documents.GetActiveDocumentViewAsync();
string text = docView?.TextBuffer?.CurrentSnapshot.GetText();
await VS.Documents.OpenAsync(fileName);
await VS.Documents.OpenInPreviewTabAsync(fileName);

// Command
await VS.Commands.ExecuteAsync("View.TaskList");

// Setting
await VS.Settings.OpenAsync<OptionsProvider.GeneralOptions>();

// Message
await VS.MessageBox.ShowAsync("Title", "Message");
await VS.MessageBox.ShowErrorAsync("Extension Name", ex.ToString());

// Event
VS.Events.SolutionEvents.OnAfterOpenProject += OnAfterOpenProject;
VS.Events.DocumentEvents.Saved += OnDocumentSaved;
```

### Setting の扱い

```csharp
// 同期的に setting を読む
var value = General.Instance.MyOption;

// 非同期に setting を読む
var general = await General.GetLiveInstanceAsync();
var value = general.MyOption;

// setting を書く
General.Instance.MyOption = newValue;
General.Instance.Save();

// または async
general.MyOption = newValue;
await general.SaveAsync();

// setting change を listen する
General.Saved += OnSettingsSaved;
```

### Text Buffer 操作

```csharp
// snapshot を取得
ITextSnapshot snapshot = textBuffer.CurrentSnapshot;

// line を取得
ITextSnapshotLine line = snapshot.GetLineFromLineNumber(lineNumber);
string lineText = line.GetText();

// tracking span を作成
ITrackingSpan trackingSpan = snapshot.CreateTrackingSpan(span, SpanTrackingMode.EdgeInclusive);

// buffer を編集
using (ITextEdit edit = textBuffer.CreateEdit())
{
    edit.Replace(span, newText);
    edit.Apply();
}

// caret 位置に挿入
DocumentView docView = await VS.Documents.GetActiveDocumentViewAsync();
if (docView?.TextView != null)
{
    SnapshotPoint position = docView.TextView.Caret.Position.BufferPosition;
    docView.TextBuffer?.Insert(position, "text to insert");
}
```

## VSCT Command Table

### Menu / Command 構造

```xml
<Commands package="YourPackage">
  <Menus>
    <Menu guid="YourPackage" id="SubMenu" type="Menu">
      <Parent guid="YourPackage" id="MenuGroup"/>
      <Strings>
        <ButtonText>Menu Name</ButtonText>
        <CommandName>Menu Name</CommandName>
        <CanonicalName>.YourExtension.MenuName</CanonicalName>
      </Strings>
    </Menu>
  </Menus>

  <Groups>
    <Group guid="YourPackage" id="MenuGroup" priority="0x0600">
      <Parent guid="guidSHLMainMenu" id="IDM_VS_CTXT_CODEWIN"/>
    </Group>
  </Groups>

  <Buttons>
    <Button guid="YourPackage" id="CommandId" type="Button">
      <Parent guid="YourPackage" id="MenuGroup"/>
      <Icon guid="ImageCatalogGuid" id="Settings"/>
      <CommandFlag>IconIsMoniker</CommandFlag>
      <CommandFlag>DynamicVisibility</CommandFlag>
      <Strings>
        <ButtonText>Command Name</ButtonText>
        <CanonicalName>.YourExtension.CommandName</CanonicalName>
      </Strings>
    </Button>
  </Buttons>
</Commands>

<Symbols>
  <GuidSymbol name="YourPackage" value="{guid-here}">
    <IDSymbol name="MenuGroup" value="0x0001"/>
    <IDSymbol name="CommandId" value="0x0100"/>
  </GuidSymbol>
</Symbols>
```

## ベストプラクティス

### 1. パフォーマンス

- 大きな document を処理する前に file / buffer size を確認する
- 効率的な span 操作には `NormalizedSnapshotSpanCollection` を使う
- 可能なら parse 結果を cache する
- library code では `ConfigureAwait(false)` を使う

```csharp
// 大きな file はスキップ
if (buffer.CurrentSnapshot.Length > 150000)
    return null;
```

### 2. Error Handling

- 外部操作は try-catch で囲む
- error は適切に log する
- 例外で VS を落としてはならない

```csharp
try
{
    // Operation
}
catch (Exception ex)
{
    await ex.LogAsync();
}
```

### 3. Disposable Resource

- tagger や長寿命 object には `IDisposable` を実装する
- Dispose で event subscription を解除する

```csharp
public void Dispose()
{
    if (!_isDisposed)
    {
        _buffer.Changed -= OnBufferChanged;
        _isDisposed = true;
    }
}
```

### 4. Content Type

`[ContentType]` attribute によく使う content type:
- `"text"` - すべての text file
- `"code"` - すべての code file
- `"CSharp"` - C# file
- `"Basic"` - VB.NET file
- `"CSS"`、`"LESS"`、`"SCSS"` - style file
- `"TypeScript"`、`"JavaScript"` - script file
- `"HTML"`、`"HTMLX"` - HTML file
- `"XML"` - XML file
- `"JSON"` - JSON file

### 5. 画像と Icon

VS Image Catalog の `KnownMonikers` を使う:

```csharp
public ImageMoniker IconMoniker => KnownMonikers.Settings;
```

VSCT では:
```xml
<Icon guid="ImageCatalogGuid" id="Settings"/>
<CommandFlag>IconIsMoniker</CommandFlag>
```

## テスト

- VS context が必要な test には `[VsTestMethod]` を使う
- 可能な限り VS service を mock する
- business logic は VS integration から分離してテストする

## よくある落とし穴

| Pitfall | Solution |
|---------|----------|
| UI thread を block する | 常に `async` / `await` を使う |
| background thread で WPF を作成する | 先に `SwitchToMainThreadAsync()` を呼ぶ |
| cancellation token を無視する | async chain 全体に渡す |
| VSCommandTable.cs の不一致 | VSCT 変更後に再生成する |
| GUID をハードコードする | `PackageGuids` と `PackageIds` 定数を使う |
| exception を握りつぶす | `await ex.LogAsync()` で log する |
| DynamicVisibility がない | `BeforeQueryStatus` を動かすのに必要 |
| `.Result`、`.Wait()` を使う | deadlock の原因。常に `await` |
| 色をハードコードする | VS theme color (`EnvironmentColors`) を使う |
| `async void` method | `async Task` を使う |

## 検証

extension を build して確認する:

```bash
msbuild /t:rebuild
```

`.editorconfig` で analyzer が有効であることを確認する:

```ini
dotnet_diagnostic.VSSDK*.severity = error
dotnet_diagnostic.VSTHRD*.severity = error
```

release 前に VS Experimental Instance でテストする。

## NuGet Package

| Package | Purpose |
|---------|---------|
| `Community.VisualStudio.Toolkit.17` | VS extension 開発を簡素化する |
| `Microsoft.VisualStudio.SDK` | Core VS SDK |
| `Microsoft.VSSDK.BuildTools` | VSIX 向け build tool |
| `Microsoft.VisualStudio.Threading.Analyzers` | Threading analyzer |
| `Microsoft.VisualStudio.SDK.Analyzers` | VSSDK analyzer |

## リソース

- [Community.VisualStudio.Toolkit](https://github.com/VsixCommunity/Community.VisualStudio.Toolkit)
- [VS Extensibility Docs](https://learn.microsoft.com/en-us/visualstudio/extensibility/)
- [VSIX Community Samples](https://github.com/VsixCommunity/Samples)

## README と Marketplace 表示

良い README は GitHub と VS Marketplace の両方で機能する。Marketplace は README.md を extension の description page として使う。

### README 構成

```markdown
[marketplace]: https://marketplace.visualstudio.com/items?itemName=Publisher.ExtensionName
[repo]: https://github.com/user/repo

# Extension Name

[![Build](https://github.com/user/repo/actions/workflows/build.yaml/badge.svg)](...)
[![Visual Studio Marketplace Version](https://img.shields.io/visual-studio-marketplace/v/Publisher.ExtensionName)][marketplace]
[![Visual Studio Marketplace Downloads](https://img.shields.io/visual-studio-marketplace/d/Publisher.ExtensionName)][marketplace]

Download this extension from the [Visual Studio Marketplace][marketplace]
or get the [CI build](http://vsixgallery.com/extension/ExtensionId/).

--------------------------------------

**extension を 1 文で売り込む hook line。**

![Screenshot](art/screenshot.png)

## Features

### Feature 1
説明と screenshot...

## How to Use
...

## License
[Apache 2.0](LICENSE)
```

### README のベストプラクティス

| Element | Guideline |
|---------|-----------|
| **Title** | vsixmanifest の `DisplayName` と同じ名前を使う |
| **Hook line** | badge の直後に、太字 1 文で value proposition を置く |
| **Screenshot** | `/art` folder に置き、relative path (`art/image.png`) を使う |
| **Image size** | 明瞭さのため 1MB 未満、幅 800-1200px に保つ |
| **Badge** | version、download、rating、build status |
| **Feature section** | 主要 feature ごとに H3 (`###`) と screenshot を使う |
| **Keyboard shortcut** | **Ctrl+M, Ctrl+C** の形式で書く |
| **Table** | option 比較や feature 一覧に有効 |
| **Link** | markdown を見やすくするため、先頭で reference-style link を使う |

### VSIX Manifest (source.extension.vsixmanifest)

```xml
<Metadata>
  <Identity Id="ExtensionName.guid-here" Version="1.0.0" Language="en-US" Publisher="Your Name" />
  <DisplayName>Extension Name</DisplayName>
  <Description xml:space="preserve">200 文字未満の短く魅力的な説明。検索結果と extension tile に表示される。</Description>
  <MoreInfo>https://github.com/user/repo</MoreInfo>
  <License>Resources\LICENSE.txt</License>
  <Icon>Resources\Icon.png</Icon>
  <PreviewImage>Resources\Preview.png</PreviewImage>
  <Tags>keyword1, keyword2, keyword3</Tags>
</Metadata>
```

### Manifest のベストプラクティス

| Element | Guideline |
|---------|-----------|
| **DisplayName** | 3-5 語程度にし、"for Visual Studio" は付けない (暗黙で分かる) |
| **Description** | 200 文字未満。feature ではなく価値に焦点を当てる。search tile に表示される |
| **Tags** | 5-10 個の関連 keyword をカンマ区切りで入れ、discoverability を高める |
| **Icon** | 128x128 または 256x256 の PNG。小さく表示されても分かる単純な design |
| **PreviewImage** | 200x200 PNG。Icon と同じでも、feature screenshot でもよい |
| **MoreInfo** | documentation と issue のため GitHub repo へ link する |

### 書き方のコツ

1. **feature ではなく benefit から始める** - "Stop wrestling with XML comments" は "XML comment formatter" より強い
2. **言うより見せる** - screenshot は説明文より説得力がある
3. **用語を一貫させる** - README、manifest、UI の用語を揃える
4. **description は流し読みしやすく保つ** - 短い paragraph、bullet、table を使う
5. **keyboard shortcut を含める** - user は生産性向上の tip を好む
6. **"Why" section を追加する** - 解決策の前に問題を説明する
