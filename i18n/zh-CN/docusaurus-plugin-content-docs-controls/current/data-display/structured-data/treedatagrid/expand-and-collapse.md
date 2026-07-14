---
id: expand-and-collapse
title: 展开与折叠操作
description: 了解如何在分层 TreeDataGrid 中以编程方式展开和折叠行、响应展开/折叠事件，以及按需实现延迟加载。
doc-type: reference
tags:
  - avalonia pro
  - avalonia enterprise
---

:::info
该控件可作为 [Avalonia Pro](https://avaloniaui.net/pricing) 或更高版本的一部分使用。
:::

当你使用分层 `TreeDataGrid` 时，用户可以通过展开和折叠行来浏览父子关系。Avalonia 在 `HierarchicalTreeDataGridSource<T>` 上提供了相应方法，使你能够以编程方式控制这一行为，无论是展开单个节点、一次性展开全部节点，还是按条件展开符合筛选器的行都可以实现。你还可以订阅在每次展开或折叠操作前后触发的事件。

:::note
以编程方式执行展开/折叠，以及处理展开/折叠事件，都需要使用代码后置 `Source` 方式并配合 `HierarchicalTreeDataGridSource<T>`。
:::

## 基本展开与折叠操作

你可以在分层 `TreeDataGrid` 中以编程方式展开或折叠行：

```csharp
var Source = new HierarchicalTreeDataGridSource<Person>(_people)
    .WithHierarchicalExpanderTextColumn(x => x.Name, x => x.Children)
    .WithTextColumn(x => x.Age);

// 通过索引路径展开指定节点
Source.Expand(new IndexPath(0));  // 展开第一个根项目

// 折叠一个节点
Source.Collapse(new IndexPath(0));
```

:::info
有关 `IndexPath` 的更多信息，请参阅 [选择模式](/controls/data-display/structured-data/treedatagrid/selection-modes)
:::

### 全部展开与全部折叠

数据源提供了用于展开或折叠所有行的内置方法：

```csharp
// 展开树中的所有行
Source.ExpandAll();

// 折叠树中的所有行
Source.CollapseAll();
```

### 基于条件展开或折叠

你可以根据条件展开或折叠行：

```csharp
// 展开所有 Person.Age > 18 的行
Source.ExpandCollapseRecursive(person => person.Age > 18);

// 折叠所有行
Source.ExpandCollapseRecursive(_ => false);
```

## 响应展开与折叠事件

你可以处理展开与折叠事件，以按需加载数据或执行其他操作。这些事件使用 `TreeDataGridRowModelEventArgs`：

```csharp
Source.RowExpanding += (sender, e) =>
{
    var person = (Person)e.Row.Model!;
    var indexPath = e.Row.ModelIndexPath;
        Debug.WriteLine($"正在展开：{person.Name}，位置 {indexPath}");
};

Source.RowExpanded += (sender, e) =>
{
    var person = (Person)e.Row.Model!;
    var indexPath = e.Row.ModelIndexPath;
        Debug.WriteLine($"已展开：{person.Name}，位置 {indexPath}");
};

Source.RowCollapsing += (sender, e) =>
{
    var person = (Person)e.Row.Model!;
    Debug.WriteLine($"正在折叠：{person.Name}");
};

Source.RowCollapsed += (sender, e) =>
{
    var person = (Person)e.Row.Model!;
    Debug.WriteLine($"已折叠：{person.Name}");
};
```

## 在展开时延迟加载数据

一种常见模式是在用户展开某一行时按需加载子数据，而不是在一开始就加载整棵树。你可以使用 `RowExpanding` 事件，在行展开之前填充子项。这样可以保持较快的初始加载速度，尤其适合大型数据集。

```csharp
Source.RowExpanding += (sender, e) =>
{
    var person = (Person)e.Row.Model!;

    if (!person.ChildrenLoaded)
    {
        var children = MyDataService.LoadChildren(person.Id);
        person.Children.Clear();
        person.Children.AddRange(children);
        person.ChildrenLoaded = true;
    }
};
```

## 另请参阅

- [TreeDataGrid](/controls/data-display/structured-data/treedatagrid/)
- [排序](/controls/data-display/structured-data/treedatagrid/sorting)
- [筛选](/controls/data-display/structured-data/treedatagrid/filtering)
- [选择模式](/controls/data-display/structured-data/treedatagrid/selection-modes)
- [列类型](/controls/data-display/structured-data/treedatagrid/column-types)
