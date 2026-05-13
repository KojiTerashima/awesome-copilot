# レイアウトとナビゲーション

## レイアウトコンポーネント

### FluentLayout

ルートのレイアウトコンテナー。最も外側の構造コンポーネントとして使用します。

```razor
<FluentLayout Orientation="Orientation.Vertical">
    <FluentHeader>...</FluentHeader>
    <FluentBodyContent>...</FluentBodyContent>
    <FluentFooter>...</FluentFooter>
</FluentLayout>
```

### FluentHeader / FluentFooter

`FluentLayout` 内の固定表示されるヘッダー／フッターセクション。

```razor
<FluentHeader Height="50">
    <FluentStack Orientation="Orientation.Horizontal" HorizontalAlignment="HorizontalAlignment.SpaceBetween">
        <span>App Title</span>
        <FluentButton>Settings</FluentButton>
    </FluentStack>
</FluentHeader>
```

### FluentBodyContent

`FluentLayout` 内のメインのスクロール可能なコンテンツ領域。

### FluentStack

水平方向または垂直方向のレイアウト用 Flexbox コンテナー。

```razor
<FluentStack Orientation="Orientation.Horizontal"
             HorizontalGap="10"
             VerticalGap="10"
             HorizontalAlignment="HorizontalAlignment.Center"
             VerticalAlignment="VerticalAlignment.Center"
             Wrap="true"
             Width="100%">
    <FluentButton>One</FluentButton>
    <FluentButton>Two</FluentButton>
</FluentStack>
```

パラメーター: `Orientation`, `HorizontalGap`, `VerticalGap`, `HorizontalAlignment`, `VerticalAlignment`, `Wrap`, `Width`。

### FluentGrid / FluentGridItem

12 列のレスポンシブグリッドシステム。

```razor
<FluentGrid Spacing="3" Justify="JustifyContent.Center" AdaptiveRendering="true">
    <FluentGridItem xs="12" sm="6" md="4" lg="3">
        Card 1
    </FluentGridItem>
    <FluentGridItem xs="12" sm="6" md="4" lg="3">
        Card 2
    </FluentGridItem>
</FluentGrid>
```

サイズパラメーター（`xs`, `sm`, `md`, `lg`, `xl`, `xxl`）は、12 列中で何列分を占有するかを表します。収まらない項目を非表示にするには `AdaptiveRendering="true"` を使用します。

### FluentMainLayout（簡易版）

ヘッダー、ナビメニュー、ボディ領域をあらかじめ構成したレイアウト。

```razor
<FluentMainLayout Header="@header"
                  SubHeader="@subheader"
                  NavMenuContent="@navMenu"
                  Body="@body"
                  HeaderHeight="50"
                  NavMenuWidth="250"
                  NavMenuTitle="Navigation" />
```

## ナビゲーションコンポーネント

### FluentNavMenu

キーボード操作をサポートした折りたたみ可能なナビゲーションメニュー。

```razor
<FluentNavMenu Width="250"
               Collapsible="true"
               @bind-Expanded="@menuExpanded"
               Title="Main navigation"
               CollapsedChildNavigation="true"
               Margin="4px 0">
    <FluentNavLink Href="/" Icon="@(Icons.Regular.Size20.Home)" Match="NavLinkMatch.All">
        Home
    </FluentNavLink>
    <FluentNavLink Href="/counter" Icon="@(Icons.Regular.Size20.NumberSymbol)">
        Counter
    </FluentNavLink>
    <FluentNavGroup Title="Admin" Icon="@(Icons.Regular.Size20.Shield)" @bind-Expanded="@adminExpanded">
        <FluentNavLink Href="/admin/users">Users</FluentNavLink>
        <FluentNavLink Href="/admin/roles">Roles</FluentNavLink>
    </FluentNavGroup>
</FluentNavMenu>
```

主要なパラメーター:
- `Width` — ピクセル単位の幅（折りたたみ時は 40px）
- `Collapsible` — 展開／折りたたみトグルを有効化
- `Expanded` / `ExpandedChanged` — バインド可能な折りたたみ状態
- `CollapsedChildNavigation` — 折りたたみ時にグループのフライアウトメニューを表示
- `CustomToggle` — モバイルのハンバーガーボタンパターン用
- `Title` — アクセシビリティ用 aria-label

### FluentNavGroup

ナビメニュー内の展開可能なグループ。

```razor
<FluentNavGroup Title="Settings"
                Icon="@(Icons.Regular.Size20.Settings)"
                @bind-Expanded="@settingsExpanded"
                Gap="2">
    <FluentNavLink Href="/settings/general">General</FluentNavLink>
    <FluentNavLink Href="/settings/profile">Profile</FluentNavLink>
</FluentNavGroup>
```

パラメーター: `Title`, `Expanded`/`ExpandedChanged`, `Icon`, `IconColor`, `HideExpander`, `Gap`, `MaxHeight`, `TitleTemplate`。

### FluentNavLink

アクティブ状態の追跡に対応したナビゲーションリンク。

```razor
<FluentNavLink Href="/page"
               Icon="@(Icons.Regular.Size20.Document)"
               Match="NavLinkMatch.Prefix"
               Target="_blank"
               Disabled="false">
    Page Title
</FluentNavLink>
```

パラメーター: `Href`, `Target`, `Match`（既定は `NavLinkMatch.Prefix`、または `All`）、`ActiveClass`, `Icon`, `IconColor`, `Disabled`, `Tooltip`。

すべてのナビコンポーネントは `FluentNavBase` を継承し、`Icon`, `IconColor`, `CustomColor`, `Disabled`, `Tooltip` を提供します。

### FluentBreadcrumb / FluentBreadcrumbItem

```razor
<FluentBreadcrumb>
    <FluentBreadcrumbItem Href="/">Home</FluentBreadcrumbItem>
    <FluentBreadcrumbItem Href="/products">Products</FluentBreadcrumbItem>
    <FluentBreadcrumbItem>Current Page</FluentBreadcrumbItem>
</FluentBreadcrumb>
```

### FluentTab / FluentTabs

```razor
<FluentTabs @bind-ActiveTabId="@activeTab">
    <FluentTab Id="tab1" Label="Details">
        Details content
    </FluentTab>
    <FluentTab Id="tab2" Label="History">
        History content
    </FluentTab>
</FluentTabs>
```
