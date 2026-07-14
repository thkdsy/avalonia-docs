---
id: sorting
title: 排序
description: 介绍如何在 Avalonia TreeDataGrid 控件中启用、禁用和自定义列排序。
doc-type: reference
tags:
  - avalonia pro
  - avalonia enterprise
---

:::info
该控件可作为 [Avalonia Pro](https://avaloniaui.net/pricing) 或更高版本的一部分使用。
:::

`TreeDataGrid` 控件支持通过点击列表头来对行进行排序。你可以按列启用或禁用排序、提供自定义比较逻辑，并以编程方式触发排序。本页将介绍这些场景。

## 列排序

### 启用排序

默认情况下，所有列都启用了排序。用户可以点击列表头进行排序。

若要禁用整个网格的排序：

```xml
<TreeDataGrid Source="{Binding Source}"
              CanUserSortColumns="False" />
```

### 将特定列设为不可排序

在 XAML 中，可在列上设置 `CanUserSortColumn` 属性：

```xml
<TreeDataGridTextColumn Header="Name" Binding="{Binding Name}" CanUserSortColumn="False" />
```

在代码后置中，可使用选项 lambda：

```csharp
source.WithTextColumn("Name", x => x.Name, o =>
{
    o.CanUserSortColumn = false;
})
```

### 以编程方式排序

你可以在数据源上使用 `SortBy` 和 `ClearSort` 方法，以编程方式对列进行排序：

```csharp
// 按指定列排序
Source.SortBy(Source.Columns[0], ListSortDirection.Ascending);
Source.SortBy(Source.Columns[1], ListSortDirection.Descending);

// 清除指定列上的排序
Source.ClearSort(Source.Columns[0]);
```

## 自定义排序

你可以在代码后置中使用比较委托提供自定义排序逻辑。这些委托接收 `object?` 参数，因此需要将其转换为你的模型类型：

```csharp
source.WithTextColumn("Name", x => x.Name, o =>
{
    o.CompareAscending = (a, b) =>
        string.Compare(((Person?)a)?.Name, ((Person?)b)?.Name, StringComparison.OrdinalIgnoreCase);
    o.CompareDescending = (a, b) =>
        string.Compare(((Person?)b)?.Name, ((Person?)a)?.Name, StringComparison.OrdinalIgnoreCase);
})
```

如果你希望在一次点击某列时按多个字段排序，这种做法会很有用。

你需要分别提供两个比较函数：一个用于升序，一个用于降序。

:::note
`TreeDataGrid` 只支持单列排序。当用户点击某个列表头时（或你以编程方式调用 `SortBy` 时），其他列上已有的排序都会被清除。如果你需要同时按多个字段排序，可以在某一列上使用自定义比较器，先比较主字段，再将次字段作为并列时的决胜条件。
:::

## 另请参阅

- [TreeDataGrid](/controls/data-display/structured-data/treedatagrid/)
- [展开与折叠](/controls/data-display/structured-data/treedatagrid/expand-and-collapse)
- [筛选](/controls/data-display/structured-data/treedatagrid/filtering)
- [选择模式](/controls/data-display/structured-data/treedatagrid/selection-modes)
- [列类型](/controls/data-display/structured-data/treedatagrid/column-types)
