---
id: column-types
title: 列类型
tags:
  - avalonia pro
  - avalonia enterprise
---

:::info
该控件可作为 [Avalonia Pro](https://avaloniaui.net/pricing) 或更高版本的一部分使用。
:::

## TreeDataGridTextColumn

`TreeDataGridTextColumn` 以文本形式显示属性值。显示时，值会通过 `ToString()` 转换为字符串。对于可编辑列，文本输入会通过 `Convert.ChangeType()` 转换回对应的属性类型。

### XAML 用法

```xml
<!-- 只读列 -->
<TreeDataGridTextColumn Header="First Name" Binding="{Binding FirstName}" />

<!-- 列宽 -->
<TreeDataGridTextColumn Header="First Name" Binding="{Binding FirstName}" Width="200" />

<!-- 格式化 -->
<TreeDataGridTextColumn Header="Birth Date" Binding="{Binding BirthDate, StringFormat='{}{0:yyyy-MM-dd}'}" />

<!-- 对齐与裁剪 -->
<TreeDataGridTextColumn Header="GDP" Binding="{Binding GDP}"
                        TextAlignment="Right"
                        MaxWidth="150" />
```

绑定到可写属性的列默认可编辑。若要将列设为只读，请设置 `IsReadOnly="True"`。

### 代码后置用法

使用 `WithTextColumn` Fluent 方法。当表达式可写时，getter 表达式会自动用于双向绑定。如果表头与属性名一致，也可以省略表头：

```csharp
// 从属性名推断表头
source.WithTextColumn(x => x.FirstName)

// 显式指定表头
source.WithTextColumn("First Name", x => x.FirstName)

// 带选项配置
source.WithTextColumn("First Name", x => x.FirstName, o =>
{
    o.Width = new GridLength(200);
    o.IsReadOnly = true;
})
```

### 选项

这些选项既可以作为 XAML 属性设置，也可以在代码后置中通过 `TextColumnCreateOptions` lambda 进行配置：

| 选项 | XAML 属性 | 默认值 | 说明 |
|---|---|---|---|
| `Width` | `Width` | `Auto` | 列宽 |
| `IsReadOnly` | `IsReadOnly` | `false` | 该列是否为只读 |
| `StringFormat` | 使用绑定的 `StringFormat` | N/A | 用于显示的格式字符串（例如 `"{0:C}"` 表示货币） |
| `Culture` | N/A | `CurrentCulture` | 用于格式化的区域性 |
| `TextAlignment` | `TextAlignment` | `Left` | 文本的水平对齐方式 |
| `TextTrimming` | `TextTrimming` | N/A | 文本过长时的裁剪方式 |
| `TextWrapping` | `TextWrapping` | `NoWrap` | 单元格内的文本换行方式 |
| `IsTextSearchEnabled` | `IsTextSearchEnabled` | `true` | 该列是否参与文本搜索 |
| `CanUserResize` | `CanUserResize` | `true` | 用户是否可以调整列宽 |
| `CanUserSortColumn` | `CanUserSortColumn` | `true` | 用户是否可以通过点击表头排序 |
| `AllowTriStateSorting` | `AllowTriStateSorting` | `false` | 是否允许升序/降序/未排序三种状态 |
| `MinWidth` / `MaxWidth` | `MinWidth` / `MaxWidth` | N/A | 最小和最大列宽 |
| `CompareAscending` / `CompareDescending` | N/A | N/A | 用于排序的自定义比较函数 |
| `BeginEditGestures` | `BeginEditGestures` | N/A | 触发编辑模式的手势（`None`、`F2`、`Tap`、`DoubleTap`、`WhenSelected`） |

:::note
对于文本列，`IsTextSearchEnabled` 默认为 `true`。如果你不希望某一列参与文本搜索，需要显式将其设为 `false`。
:::

## TreeDataGridCheckBoxColumn

`TreeDataGridCheckBoxColumn` 以复选框形式显示布尔值。

### XAML 用法

```xml
<!-- 基本复选框 -->
<TreeDataGridCheckBoxColumn Header="Active" Binding="{Binding IsActive}" />

<!-- 只读且不可调整大小 -->
<TreeDataGridCheckBoxColumn Binding="{Binding IsChecked}"
                            CanUserResize="False"
                            IsReadOnly="True" />
```

### 代码后置用法

对于 `bool` 属性，使用 `WithCheckBoxColumn`；对于 `bool?`（可空）属性，使用 `WithThreeStateCheckBoxColumn`：

```csharp
// 基本复选框
source.WithCheckBoxColumn(x => x.IsActive)

// 显式指定表头
source.WithCheckBoxColumn("Active", x => x.IsActive)

// 用于可空 bool 的三态复选框
source.WithThreeStateCheckBoxColumn(x => x.IsChecked)

// 带选项配置
source.WithCheckBoxColumn(x => x.IsActive, o =>
{
    o.CanUserResize = false;
    o.Width = new GridLength(80);
})
```

### 选项

| 选项 | XAML 属性 | 默认值 | 说明 |
|---|---|---|---|
| `Width` | `Width` | `Auto` | 列宽 |
| `IsReadOnly` | `IsReadOnly` | `false` | 该列是否为只读 |
| `CanUserResize` | `CanUserResize` | `true` | 用户是否可以调整列宽 |
| `CanUserSortColumn` | `CanUserSortColumn` | `true` | 用户是否可以通过点击表头排序 |
| `AllowTriStateSorting` | `AllowTriStateSorting` | `false` | 是否允许升序/降序/未排序三种状态 |
| `MinWidth` / `MaxWidth` | `MinWidth` / `MaxWidth` | N/A | 最小和最大列宽 |
| `CompareAscending` / `CompareDescending` | N/A | N/A | 用于排序的自定义比较函数 |
| `BeginEditGestures` | `BeginEditGestures` | N/A | 触发编辑模式的手势 |

## TreeDataGridHierarchicalExpanderColumn

`TreeDataGridHierarchicalExpanderColumn` 使用展开器控件来显示分层树数据，并可显示或隐藏子项。这种列类型只能用于分层数据。

展开器列会包裹一个内部列（通常是 `TreeDataGridTextColumn` 或 `TreeDataGridTemplateColumn`），该内部列决定在展开器图标旁边显示什么内容。

### XAML 用法

在 XAML 中定义展开器列时，需要为子项提供绑定，并将内部列作为内容嵌套进去：

```xml
<TreeDataGridHierarchicalExpanderColumn Header="Name" Width="*"
                                        ChildrenBinding="{Binding Children}"
                                        HasChildrenBinding="{Binding HasChildren}"
                                        IsExpandedBinding="{Binding IsExpanded}">
  <TreeDataGridTextColumn Binding="{Binding Name}" />
</TreeDataGridHierarchicalExpanderColumn>
```

| 属性 | 说明 |
|---|---|
| `ChildrenBinding` | 绑定到每一行的子项集合（必需） |
| `HasChildrenBinding` | 用于在不加载子项的情况下判断某行是否有子项（适合延迟加载） |
| `IsExpandedBinding` | 将展开状态持久化绑定到模型属性 |

### 代码后置用法

如果展开器内部是文本列，请使用 `WithHierarchicalExpanderTextColumn`：

```csharp
source.WithHierarchicalExpanderTextColumn(x => x.Name, x => x.Children)

// 带选项配置
source.WithHierarchicalExpanderTextColumn(x => x.Name, x => x.Children, o =>
{
    o.Width = GridLength.Star;
    o.HasChildren = x => x.HasChildren;
    o.IsExpanded = x => x.IsExpanded;
})
```

如果需要自定义内部列（例如模板列），请使用 `WithHierarchicalExpanderColumn`：

```csharp
source.WithHierarchicalExpanderColumn(
    "Name",
    new TreeDataGridTemplateColumn(null, "FileNameCell", "FileNameEditCell"),
    x => x.Children,
    o =>
    {
        o.Width = GridLength.Star;
        o.HasChildren = x => x.HasChildren;
        o.IsExpanded = x => x.IsExpanded;
    })
```

## TreeDataGridTemplateColumn

`TreeDataGridTemplateColumn` 使用数据模板来渲染单元格内容，因此可以完全控制外观和行为。

### XAML 用法

你可以直接内联定义单元格模板（以及可选的编辑模板）：

```xml
<TreeDataGridTemplateColumn Header="Region">
  <TreeDataGridTemplateColumn.CellTemplate>
    <DataTemplate DataType="m:Country">
      <TextBlock Text="{Binding Region}" />
    </DataTemplate>
  </TreeDataGridTemplateColumn.CellTemplate>
  <TreeDataGridTemplateColumn.CellEditingTemplate>
    <DataTemplate DataType="m:Country">
      <ComboBox ItemsSource="{x:Static m:Countries.Regions}"
                SelectedItem="{Binding Region}" />
    </DataTemplate>
  </TreeDataGridTemplateColumn.CellEditingTemplate>
</TreeDataGridTemplateColumn>
```

### 代码后置用法

#### 使用 `IDataTemplate` 实例：

```csharp
source.WithTemplateColumn(
    "Selected",
    new FuncDataTemplate<Person>((_, _) => new CheckBox
    {
        [!CheckBox.IsCheckedProperty] = new Binding("IsSelected"),
    }))
```

#### 使用 XAML 资源键：

先在 `TreeDataGrid.Resources` 中定义模板：

```xml
<TreeDataGrid Source="{Binding Source}">
  <TreeDataGrid.Resources>
    <DataTemplate x:Key="CheckBoxCell">
      <CheckBox IsChecked="{Binding IsSelected}" />
    </DataTemplate>
  </TreeDataGrid.Resources>
</TreeDataGrid>
```

然后通过键名引用它：

```csharp
source.WithTemplateColumnFromResourceKeys("Selected", "CheckBoxCell")

// 使用单独的编辑模板
source.WithTemplateColumnFromResourceKeys("Selected", "CheckBoxCell", "CheckBoxCellEdit")
```

### 选项

| 选项 | XAML 属性 | 默认值 | 说明 |
|---|---|---|---|
| `Width` | `Width` | `Auto` | 列宽 |
| `TextSearchBinding` | N/A | N/A | 从模型中提取可搜索文本的绑定 |
| `CanUserResize` | `CanUserResize` | `true` | 用户是否可以调整列宽 |
| `CanUserSortColumn` | `CanUserSortColumn` | `true` | 用户是否可以通过点击表头排序 |
| `AllowTriStateSorting` | `AllowTriStateSorting` | `false` | 是否允许升序/降序/未排序三种状态 |
| `MinWidth` / `MaxWidth` | `MinWidth` / `MaxWidth` | N/A | 最小和最大列宽 |
| `CompareAscending` / `CompareDescending` | N/A | N/A | 用于排序的自定义比较函数 |
| `BeginEditGestures` | `BeginEditGestures` | N/A | 触发编辑模式的手势 |

若要在模板列上启用文本搜索，请使用 `CompiledBinding.Create` 设置 `TextSearchBinding`：

```csharp
source.WithTemplateColumnFromResourceKeys("Name", "FileNameCell", "FileNameEditCell", o =>
{
    o.TextSearchBinding = CompiledBinding.Create<FileTreeNodeModel, string>(x => x.Name);
})
```

## TreeDataGridRowHeaderColumn

`TreeDataGridRowHeaderColumn` 在最左侧列中显示行头（通常是行号）。

### XAML 用法

```xml
<TreeDataGrid ItemsSource="{Binding Data}">
  <TreeDataGridRowHeaderColumn />
  <TreeDataGridTextColumn Header="Name" Binding="{Binding Name}" />
</TreeDataGrid>
```

### 代码后置用法

```csharp
source.WithRowHeaderColumn()
```

## 另请参阅

- [TreeDataGrid](/controls/data-display/structured-data/treedatagrid/)
