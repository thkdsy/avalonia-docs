---
id: selection-modes
title: 选择模式
tags:
  - avalonia pro
  - avalonia enterprise
---

支持两种选择模式：

- **行选择** 允许用户选择整行
- **单元格选择** 允许用户选择单个单元格

这两种选择类型都支持单选和多选。默认选择类型为单行选择。


:::info
此控件作为 [Avalonia Pro](https://avaloniaui.net/pricing) 或更高版本的一部分提供。
:::

## 在 XAML 中设置选择模式

直接在 `TreeDataGrid` 控件上设置 `SelectionMode` 属性即可。这种方式同时适用于 `ItemsSource` 和 `Source`：

```xml
<!-- 单行选择（默认） -->
<TreeDataGrid ItemsSource="{Binding People}" SelectionMode="Row" />

<!-- 多行选择 -->
<TreeDataGrid ItemsSource="{Binding People}" SelectionMode="Row,Multiple" />

<!-- 单个单元格选择 -->
<TreeDataGrid ItemsSource="{Binding People}" SelectionMode="Cell" />

<!-- 多个单元格选择 -->
<TreeDataGrid ItemsSource="{Binding People}" SelectionMode="Cell,Multiple" />
```

## SelectionChanged 事件

`TreeDataGrid` 控件具有一个 `SelectionChanged` 事件，会在选择发生变化时触发：

```csharp
treeDataGrid.SelectionChanged += (sender, e) =>
{
    // e is TreeDataGridSelectionChangedEventArgs
    foreach (var item in e.SelectedItems)
    {
        Debug.WriteLine($"Selected: {item}");
    }

    foreach (var item in e.DeselectedItems)
    {
        Debug.WriteLine($"Deselected: {item}");
    }
};
```

该事件同时适用于 XAML（`ItemsSource`）方式和 code-behind（`Source`）方式。（详见[主参考页](/controls/data-display/structured-data/treedatagrid#two-approaches)。）

## 索引路径

由于 `TreeDataGrid` 支持分层数据，因此仅使用简单索引不足以标识数据源中的某一行。为此，这里使用 `IndexPath` 结构来表示索引。

`IndexPath` 是一个索引数组，其中每个元素都表示数据层级结构中更深一层的索引位置。

请看下面这个数据源：

```text
|- A
|  |- B
|  |- C
|     |- D
|- E
```

- `A` 的索引路径是 `0`，因为它是层级根部的第一个项目
- `B` 的索引路径是 `0,0`，因为它是第一个项目的第一个子项
- `C` 的索引路径是 `0,1`，因为它是第一个项目的第二个子项
- `D` 的索引路径是 `0,1,0`，因为它是 `C` 的第一个子项
- `E` 的索引路径是 `1`，因为它是根部的第二个项目

`IndexPath` 是一个不可变结构体，使用整数数组构造，例如：`new IndexPath(0, 1, 0)`。在处理扁平数据源时，也支持从 `int` 的隐式转换。

## 行选择（code-behind）

当使用 code-behind 的 `Source` 方式时，行选择通过数据源上的 `RowSelection` 属性暴露出来。

行选择保存在 `TreeDataGridRowSelectionModel<TModel>` 类的实例中。

默认是单选。若要启用多选，请将 `SingleSelect` 属性设置为 `false`：

```csharp
Source = new FlatTreeDataGridSource<Person>(_people)
    .WithTextColumn("First Name", x => x.FirstName)
    .WithTextColumn("Last Name", x => x.LastName)
    .WithTextColumn(x => x.Age);

Source.RowSelection!.SingleSelect = false;
```

### 获取已选项

通过选择模型访问已选项：

```csharp
// Get single selected item
if (Source.RowSelection?.SelectedItem is Person selectedPerson)
{
    Debug.WriteLine($"Selected: {selectedPerson.Name}");
}

// Get multiple selected items
var selectedItems = Source.RowSelection?.SelectedItems;
if (selectedItems != null)
{
    foreach (var item in selectedItems.OfType<Person>())
    {
        Debug.WriteLine($"Selected: {item.Name}");
    }
}
```

### 以编程方式选择行

你可以使用选择模型通过代码选择行：

```csharp
var selection = Source.RowSelection;

// Select by index
selection.SelectedIndex = 2;
selection.SelectedIndex = new IndexPath(2);
selection.SelectedIndex = new IndexPath(2, 1);

// Clear selection
selection.Clear();

// Select multiple items
selection.Select(2);
selection.Select(new IndexPath(2, 1));

// Deselect multiple items
selection.Deselect(2);
selection.Deselect(new IndexPath(2, 1));

// Batch selection changes
selection.BeginBatchUpdate();
selection.Select(0);
selection.Select(1);
selection.Deselect(2);
selection.EndBatchUpdate();
```

### 选择变化事件

通过选择模型上的 `SelectionChanged` 事件来处理选择变化：

```csharp
Source.RowSelection.SelectionChanged += (sender, e) =>
{
    // e is TreeDataGridSelectionChangedEventArgs<Person>
    Debug.WriteLine($"Selection changed");
    Debug.WriteLine($"Added: {e.SelectedItems.Count}");
    Debug.WriteLine($"Removed: {e.DeselectedItems.Count}");
};
```

## 单元格选择（code-behind）

若要在 code-behind 方式下启用单元格选择，请将一个 `TreeDataGridCellSelectionModel<TModel>` 实例赋值给数据源的 `Selection` 属性：

```csharp
Source = new FlatTreeDataGridSource<Person>(_people)
    .WithTextColumn("First Name", x => x.FirstName)
    .WithTextColumn("Last Name", x => x.LastName)
    .WithTextColumn(x => x.Age);

Source.Selection = new TreeDataGridCellSelectionModel<Person>(Source);
```

启用多单元格选择后，可以选择一个矩形范围内的单元格：

```csharp
Source.Selection = new TreeDataGridCellSelectionModel<Person>(Source)
{
    SingleSelect = false
};
```
单元格选择通过数据源上的 `CellSelection` 属性暴露出来。

`CellIndex` 结构通过整型列索引和 `IndexPath` 行索引的组合来标识单个单元格：

```csharp
// Access selected cell
if (Source.CellSelection?.SelectedIndex is { } selectedCell)
{
    Debug.WriteLine($"Selected cell - Row: {selectedCell.RowIndex}, Column: {selectedCell.ColumnIndex}");
}
```

### 获取已选项

通过选择模型访问已选项：

```csharp
// Get single selected cell
var selection = Source.CellSelection!;

if (selection.SelectedIndex.ColumnIndex != -1 &&
    selection.SelectedIndex.RowIndex.Count == 1)
{
    var column = Source.Columns[selection.SelectedIndex.ColumnIndex];
    var model = _data[selection.SelectedIndex.RowIndex[0]];

    Debug.WriteLine("Selected column: " + column.Header);
    Debug.WriteLine("Selected item: " + model);
}

// Get multiple selected cells
foreach (var selected in selection.SelectedIndexes)
{
    if (selected.ColumnIndex != -1 && selected.RowIndex.Count == 1)
    {
        var column = Source.Columns[selected.ColumnIndex];
        var model = _data[selected.RowIndex[0]];

        Debug.WriteLine("Selected column: " + column.Header);
        Debug.WriteLine("Selected item: " + model);
    }
}
```

### Programmatically selecting cells

You can select cells programmatically using the selection model:

```csharp
var selection = Source.CellSelection;

// Select by index
selection.SelectedIndex = new CellIndex(2, 1);
selection.SelectedIndex = new CellIndex(3, new IndexPath(2));

// Select a range
selection.SetSelectedRange(new CellIndex(1, 1), columnCount: 2, rowCount: 2);
```

### Selection changed event

Handle selection changes with the `SelectionChanged` event:

```csharp
Source.CellSelection!.SelectionChanged += (s, e) =>
{
    Debug.WriteLine($"Selection changed");
};
```

## See also

- [TreeDataGrid](/controls/data-display/structured-data/treedatagrid/)
