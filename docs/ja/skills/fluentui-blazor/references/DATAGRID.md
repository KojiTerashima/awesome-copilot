# FluentDataGrid

`FluentDataGrid<TGridItem>` は、表形式データを表示するための、厳密に型付けされたジェネリックコンポーネントです。

## 基本的な使い方

```razor
<FluentDataGrid Items="@people" TGridItem="Person">
    <PropertyColumn Property="@(p => p.Name)" Sortable="true" />
    <PropertyColumn Property="@(p => p.Email)" />
    <PropertyColumn Property="@(p => p.BirthDate)" Format="yyyy-MM-dd" />
    <TemplateColumn Title="Actions">
        <FluentButton OnClick="@(() => Edit(context))">Edit</FluentButton>
    </TemplateColumn>
</FluentDataGrid>
```

**重要**: 列はプロパティではなく子コンポーネントです。グリッド内では `PropertyColumn`、`TemplateColumn`、`SelectColumn` を使用してください。

## 列の種類

### PropertyColumn

プロパティ式にバインドします。タイトルはプロパティ名または `[Display]` 属性から自動的に導出されます。

```razor
<PropertyColumn Property="@(p => p.Name)" Sortable="true" />
<PropertyColumn Property="@(p => p.Price)" Format="C2" Title="Unit Price" />
<PropertyColumn Property="@(p => p.Category)" Comparer="@StringComparer.OrdinalIgnoreCase" />
```

パラメーター: `Property`（必須）、`Format`、`Title`、`Sortable`、`SortBy`、`Comparer`、`IsDefaultSortColumn`、`InitialSortDirection`、`Class`、`Tooltip`。

### TemplateColumn

レンダーフラグメントによる完全なカスタム描画。`context` は `TGridItem` です。

```razor
<TemplateColumn Title="Status" SortBy="@statusSort">
    <FluentBadge Appearance="Appearance.Accent"
                 BackgroundColor="@(context.IsActive ? "green" : "red")">
        @(context.IsActive ? "Active" : "Inactive")
    </FluentBadge>
</TemplateColumn>
```

### SelectColumn

チェックボックスによる選択列です。

```razor
<SelectColumn TGridItem="Person"
              SelectMode="DataGridSelectMode.Multiple"
              @bind-SelectedItems="@selectedPeople" />
```

モード: `DataGridSelectMode.Single`、`DataGridSelectMode.Multiple`。

## データソース

相互排他的な 2 つのアプローチがあります。

### インメモリ（IQueryable）

```razor
<FluentDataGrid Items="@people.AsQueryable()" TGridItem="Person">
    ...
</FluentDataGrid>
```

### サーバーサイド / カスタム（ItemsProvider）

```razor
<FluentDataGrid ItemsProvider="@peopleProvider" TGridItem="Person">
    ...
</FluentDataGrid>

@code {
    private GridItemsProvider<Person> peopleProvider = async request =>
    {
        var result = await PeopleService.GetPeopleAsync(
            request.StartIndex,
            request.Count ?? 50,
            request.GetSortByProperties().FirstOrDefault());

        return GridItemsProviderResult.From(result.Items, result.TotalCount);
    };
}
```

### EF Core Adapter

```csharp
// Program.cs
builder.Services.AddDataGridEntityFrameworkAdapter();
```

```razor
<FluentDataGrid Items="@dbContext.People" TGridItem="Person">
    ...
</FluentDataGrid>
```

## ページネーション

```razor
<FluentDataGrid Items="@people" Pagination="@pagination" TGridItem="Person">
    ...
</FluentDataGrid>

<FluentPaginator State="@pagination" />

@code {
    private PaginationState pagination = new() { ItemsPerPage = 10 };
}
```

## 仮想化

大規模データセットでは、仮想化を有効にします。

```razor
<FluentDataGrid Items="@people" Virtualize="true" ItemSize="46" TGridItem="Person">
    ...
</FluentDataGrid>
```

`ItemSize` は行の高さ（ピクセル）の推定値です（既定値は状況により異なります）。スクロール位置の計算に重要です。

## 主要パラメーター

| Parameter | Type | Description |
|---|---|---|
| `Items` | `IQueryable<TGridItem>?` | インメモリデータソース |
| `ItemsProvider` | `GridItemsProvider<TGridItem>?` | 非同期データプロバイダー |
| `Pagination` | `PaginationState?` | ページネーション状態 |
| `Virtualize` | `bool` | 仮想化を有効化 |
| `ItemSize` | `float` | 行の高さの推定値（px） |
| `ItemKey` | `Func<TGridItem, object>?` | `@key` 用の安定キー |
| `ResizableColumns` | `bool` | 列リサイズを有効化 |
| `HeaderCellAsButtonWithMenu` | `bool` | ソート可能ヘッダー UI |
| `GridTemplateColumns` | `string?` | CSS grid-template-columns |
| `Loading` | `bool` | ローディングインジケーターを表示 |
| `ShowHover` | `bool` | ホバー時に行を強調表示 |
| `OnRowClick` | `EventCallback<FluentDataGridRow<TGridItem>>` | 行クリックハンドラー |
| `OnRowDoubleClick` | `EventCallback<FluentDataGridRow<TGridItem>>` | 行ダブルクリックハンドラー |
| `OnRowFocus` | `EventCallback<FluentDataGridRow<TGridItem>>` | 行フォーカスハンドラー |

## ソート

```razor
<PropertyColumn Property="@(p => p.Name)" Sortable="true" IsDefaultSortColumn="true"
                InitialSortDirection="SortDirection.Ascending" />
```

またはカスタムソートを使用:

```razor
<TemplateColumn Title="Full Name" SortBy="@(GridSort<Person>.ByAscending(p => p.LastName).ThenAscending(p => p.FirstName))">
    @context.LastName, @context.FirstName
</TemplateColumn>
```
