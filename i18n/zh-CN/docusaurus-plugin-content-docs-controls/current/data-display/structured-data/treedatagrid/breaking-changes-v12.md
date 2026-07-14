---
id: breaking-changes-v12
title: TreeDataGrid v12 破坏性变更
tags:
  - avalonia pro
  - avalonia enterprise
---

本文档介绍 TreeDataGrid 11.x 与 12.x 之间的破坏性变更，并提供迁移指导。

TreeDataGrid 12.0 对 TreeDataGrid API 进行了重大重构，目的是让 API 更具前瞻性，并提供 TreeDataGrid 11.x 中尚不具备的 API 稳定性承诺。

TreeDataGrid 12.x 需要 Avalonia 12。有关 Avalonia 12 的破坏性变更，请参阅 [Avalonia 12 breaking changes](https://docs.avaloniaui.net/docs/avalonia12-breaking-changes) 文档。

## 列中移除了泛型类型参数

所有列类都移除了泛型类型参数，并统一重命名为带有 `TreeDataGrid` 前缀的名称。列不再针对模型类型使用泛型。

| 12.x | 替代 |
|---|---|
| `TreeDataGridTextColumn` | `TextColumn<TModel, TValue>` |
| `TreeDataGridCheckBoxColumn` | `CheckBoxColumn<TModel>` |
| `TreeDataGridTemplateColumn` | `TemplateColumn<TModel>` |
| `TreeDataGridHierarchicalExpanderColumn` | `HierarchicalExpanderColumn<TModel>` |
| `TreeDataGridRowHeaderColumn` | `RowHeaderColumn<TModel>` |
| `TreeDataGridColumns` | `ColumnList<TModel>` |

## XAML 支持

TreeDataGrid 现在支持直接在 XAML 中定义列，而不需要 code-behind 数据源。设置 `ItemsSource` 属性后，即可将列直接定义为内容：

```xml
<TreeDataGrid ItemsSource="{Binding Countries}" SelectionMode="Row,Multiple">
    <TreeDataGridTextColumn Header="Country" Binding="{Binding Name}" Width="6*" />
    <TreeDataGridTextColumn Header="Region" Binding="{Binding Region}" Width="2*" />
    <TreeDataGridCheckBoxColumn Binding="{Binding IsChecked}" CanUserResize="False" />
</TreeDataGrid>
```

对于分层数据，请使用 `TreeDataGridHierarchicalExpanderColumn`：

```xml
<TreeDataGrid ItemsSource="{Binding Files}">
    <TreeDataGridHierarchicalExpanderColumn Header="Name" Width="*"
                                            ChildrenBinding="{Binding Children}"
                                            HasChildrenBinding="{Binding HasChildren}"
                                            IsExpandedBinding="{Binding IsExpanded}">
        <TreeDataGridTextColumn Binding="{Binding Name}"/>
    </TreeDataGridHierarchicalExpanderColumn>
</TreeDataGrid>
```

基于 code-behind 的 `FlatTreeDataGridSource` / `HierarchicalTreeDataGridSource` 方式仍然可以像以前一样工作，但需要适配本文档所述的 API 变更。

## Fluent 列 API

现在可以通过 fluent API 在代码中创建列。

不再使用分离的 getter/setter lambda。当表达式可写时，getter 表达式会自动用于双向绑定。如果你想创建只读列，请在选项回调中设置 `IsReadOnly`。

当标题与 lambda 选中的属性名一致时，可以省略标题。

### `WithTextColumn`

```diff
-new TextColumn<Country, string>(
-    "Country",
-    x => x.Name,
-    (r, v) => r.Name = v,
-    new GridLength(6, GridUnitType.Star))
+source.WithTextColumn("Country", x => x.Name, o => o.Width = new GridLength(6, GridUnitType.Star))
```

示例如下：

```csharp
source.WithTextColumn("Name", x => x.Name)
source.WithTextColumn(x => x.Name)
source.WithTextColumn(x => x.Name, o => o.IsReadOnly = true)
source.WithTextColumn(x => x.Name, o => o.Width = GridLength.Star)
```

### `WithCheckBoxColumn`/`WithThreeStateCheckBoxColumn`

可以使用 `WithCheckBoxColumn` 或 `WithThreeStateCheckBoxColumn` 添加复选框列：

```diff
-new CheckBoxColumn<FileTreeNodeModel>(
-    null,
-    x => x.IsChecked,
-    (o, v) => o.IsChecked = v)
+source.WithCheckBoxColumn(null, x => x.IsChecked)
```

### `WithTemplateColumn`

模板列可以通过 `IDataTemplate` 实例或资源键来添加：

```diff
-new TemplateColumn<Country>(
-    "Region",
-    "RegionCell",
-    "RegionEditCell")
+source.WithTemplateColumnFromResourceKeys("Region", "RegionCell", "RegionEditCell")
```

### `WithHierarchicalExpanderColumn`/`WithHierarchicalExpanderTextColumn`

对于在展开列内部使用文本列的分层数据：

```diff
-new HierarchicalExpanderColumn<FileTreeNodeModel>(
-    new TextColumn<FileTreeNodeModel, string>(
-        "Name",
-        x => x.Name),
-    x => x.Children,
-    x => x.HasChildren,
-    x => x.IsExpanded)
+source.WithHierarchicalExpanderTextColumn(x => x.Name, x => x.Children, o =>
+{
+    o.HasChildren = x => x.HasChildren;
+    o.IsExpanded = x => x.IsExpanded;
+})
```

现在标题和宽度已从内部列移动到展开列本身。

对于带有自定义内部列（例如模板列）的展开列，请使用 `WithHierarchicalExpanderColumn`：

```csharp
source.WithHierarchicalExpanderColumn(
    "Name",
    new TreeDataGridTemplateColumn(null, "FileNameCell", "FileNameEditCell"),
    x => x.Children,
    o =>
    {
        o.Width = new GridLength(1, GridUnitType.Star);
        o.HasChildren = x => x.HasChildren;
        o.IsExpanded = x => x.IsExpanded;
    })
```

### 完整链式调用示例

```csharp
var source = new FlatTreeDataGridSource<Country>(data)
    .WithRowHeaderColumn()
    .WithTextColumn("Country", x => x.Name, o =>
    {
        o.Width = new GridLength(6, GridUnitType.Star);
        o.IsTextSearchEnabled = true;
    })
    .WithTemplateColumnFromResourceKeys("Region", "RegionCell", "RegionEditCell")
    .WithTextColumn(x => x.Population, o => o.Width = new GridLength(3, GridUnitType.Star))
    .WithTextColumn(x => x.Area, o => o.Width = new GridLength(3, GridUnitType.Star));
```

## 列选项

列选项类已被顶层的 `*CreateOptions` 类替代，并通过 fluent 方法上的 lambda 回调进行配置：

| 12.x | 替代 |
|---|---|
| `ColumnCreateOptions` | `ColumnOptions<TModel>` |
| `TextColumnCreateOptions` | `TextColumnOptions<TModel>` |
| `TemplateColumnCreateOptions` | `TemplateColumnOptions<TModel>` |
| `CheckBoxColumnCreateOptions` | `CheckBoxColumnOptions<TModel>` |

```diff
-new TextColumn<Country, string>(
-    "Country",
-    x => x.Name,
-    options: new TextColumnOptions<Country>
-    {
-        CanUserResizeColumn = false,
-        IsTextSearchEnabled = true,
-    })
+source.WithTextColumn("Country", x => x.Name, o =>
+{
+    o.CanUserResize = false;
+    o.IsTextSearchEnabled = true;
+})
```

`CanUserResizeColumn` 属性已重命名为 `CanUserResize`。

`TextColumnCreateOptions` 上的 `IsTextSearchEnabled` 现在默认值为 `true`。在 11.x 中，该值原本为 `false`。如果你不希望文本列启用文本搜索，现在必须显式禁用它：

```csharp
source.WithTextColumn("Name", x => x.Name, o => o.IsTextSearchEnabled = false)
```

## 接口被抽象类取代

TreeDataGrid API 中的主要接口已被抽象类替代：

| 12.x | 替代 |
|---|---|
| `TreeDataGridSource` | `ITreeDataGridSource` |
| `TreeDataGridSource<TModel>` | `ITreeDataGridSource<TModel>` |
| `TreeDataGridColumn` | `IColumn` |
| `TreeDataGridColumns` | `IColumns` |
| `TreeDataGridRows` | `IRows` |
| `TreeDataGridSelectionModel` | `ITreeDataGridSelection` |
| `TreeDataGridRowSelectionModel<T>` | `ITreeDataGridRowSelectionModel<T>` |
| `TreeDataGridCellSelectionModel<T>` | `ITreeDataGridCellSelectionModel<T>` |

## Renamed types

The following types have been renamed:

| 12.x | Replaces |
|---|---|
| `ITreeDataGridCellModel` | `ICell` |
| `ITreeDataGridRowModel` | `IRow` |

## Removed types

The following types were removed:

- `NotifyingBase`, `ReadOnlyListBase<T>`, `SortableRowsBase<TModel, TRow>`
- `AnonymousSortableRows<TModel>`, `HierarchicalRows<TModel>`, `HierarchicalRow<TModel>`
- `TextCell`, `CheckBoxCell`, `ExpanderCell`, `TemplateCell`, `RowHeaderCell`
- `IExpander`, `IExpanderCell`, `IExpanderRow`, `IExpanderRow<TModel>`,
  `IExpanderRowController<TModel>`
- `IIndentedRow`, `IRow<TModel>`
- `ICellOptions`, `ITextCellOptions`, `ITemplateCellOptions`
- `IUpdateColumnLayout`
- `NotifyingListBase<T>`
- `DragInfo`

If you depended on any of these types, please open an issue and we will discuss a replacement.

## Sources are now sealed

`FlatTreeDataGridSource<TModel>` and `HierarchicalTreeDataGridSource<TModel>` are now
`sealed` classes. They can no longer be subclassed.

## Custom sort comparisons use `object?`

Custom sort comparison delegates on `CompareAscending` and `CompareDescending` have
changed from `Comparison<TModel?>?` to `Comparison<object?>?`:

```diff
-options: new ColumnOptions<Country>
-{
-    CompareAscending = (a, b) => string.Compare(a?.Name, b?.Name),
-}
+source.WithTextColumn("Country", x => x.Name, o =>
+{
+    o.CompareAscending = (a, b) => string.Compare(((Country?)a)?.Name, ((Country?)b)?.Name);
+})
```

## Unified `SelectionChangedEventArgs`

Selection changed events now use `TreeDataGridSelectionChangedEventArgs` (or the generic
`TreeDataGridSelectionChangedEventArgs<TModel>`), replacing the previous
`TreeSelectionModelSelectionChangedEventArgs<T>`. The new event args provide:

- `SelectedIndexes` / `DeselectedIndexes`
- `SelectedItems` / `DeselectedItems`
- `SelectedCellIndexes` / `DeselectedCellIndexes`

A `SelectionChanged` event has also been added to the `TreeDataGrid` control itself.

## Row events use `TreeDataGridRowModelEventArgs`

The `RowExpanding`, `RowExpanded`, `RowCollapsing`, and `RowCollapsed` events now use
`TreeDataGridRowModelEventArgs` instead of `RowEventArgs<HierarchicalRow<TModel>>`.

## Text search uses bindings

Text search configuration has changed from lambda-based value selectors to Avalonia
bindings.

For template columns, the `TextSearchValueSelector` lambda on `TemplateColumnOptions` has
been replaced with a `TextSearchBinding` property on `TemplateColumnCreateOptions`:

```diff
-new TemplateColumn<FileTreeNodeModel>(
-    "Name",
-    "FileNameCell",
-    "FileNameEditCell",
-    options: new TemplateColumnOptions<FileTreeNodeModel>
-    {
-        TextSearchValueSelector = x => x.Name,
-    })
+source.WithTemplateColumnFromResourceKeys("Name", "FileNameCell", "FileNameEditCell", o =>
+{
+    o.TextSearchBinding = CompiledBinding.Create<FileTreeNodeModel, string>(x => x.Name);
+})
```

For text columns, text search is enabled by default via the `IsTextSearchEnabled` property
on `TextColumnCreateOptions` and uses the column's own binding automatically.

## Namespace changes

The `Avalonia.Controls.Models.TreeDataGrid` namespace (which contained the old column types
such as `TextColumn<TModel, TValue>`, `TemplateColumn<TModel>`, `HierarchicalExpanderColumn<TModel>`,
`TextColumnOptions<TModel>`, etc.) has been removed. The new column types and options classes
are in the `Avalonia.Controls` namespace. Remove `using Avalonia.Controls.Models.TreeDataGrid`
from your code.

## Experimental bindings removed

The `Avalonia.Experimental.Data` namespace and all its types have been removed. This
includes `TypedBinding<TIn, TOut>`, `TypedBindingExpression<TIn, TOut>`,
`LightweightObservableBase<T>`, `SingleSubscriberObservableBase<T>`, and related types.

TreeDataGrid 12.x uses standard Avalonia bindings instead.
