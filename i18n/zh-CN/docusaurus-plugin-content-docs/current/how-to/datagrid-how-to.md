---
id: datagrid-how-to
title: "如何：使用 DataGrid"
description: 了解 DataGrid 的排序、筛选、分组、模板列、选择、验证和编辑功能。
doc-type: how-to
---

本指南介绍常见的 DataGrid 使用场景：排序、筛选、分组、模板列、选择、验证以及编辑。

## 安装与配置

安装 NuGet 包，并添加对应的样式引用：

```bash
dotnet add package Avalonia.Controls.DataGrid
```

```xml title='App.axaml'
<Application.Styles>
    <FluentTheme />
    <StyleInclude Source="avares://Avalonia.Controls.DataGrid/Themes/Fluent.xaml"/>
</Application.Styles>
```

## 基础绑定式 DataGrid

```xml
<DataGrid ItemsSource="{Binding Products}" AutoGenerateColumns="False"
          IsReadOnly="True" GridLinesVisibility="All"
          BorderThickness="1" BorderBrush="Gray">
    <DataGrid.Columns>
        <DataGridTextColumn Header="Name" Binding="{Binding Name}" Width="2*" />
        <DataGridTextColumn Header="Price" Binding="{Binding Price, StringFormat='{}{0:C}'}" Width="*" />
        <DataGridCheckBoxColumn Header="In Stock" Binding="{Binding InStock}" Width="Auto" />
    </DataGrid.Columns>
</DataGrid>
```

```csharp
public partial class MainViewModel : ObservableObject
{
    public ObservableCollection<Product> Products { get; } = new()
    {
        new Product("Widget", 9.99m, true),
        new Product("Gadget", 24.99m, false),
        new Product("Gizmo", 14.50m, true),
    };
}

public class Product
{
    public string Name { get; set; }
    public decimal Price { get; set; }
    public bool InStock { get; set; }

    public Product(string name, decimal price, bool inStock)
    {
        Name = name;
        Price = price;
        InStock = inStock;
    }
}
```

## 排序

排序默认是启用的（`CanUserSortColumns="True"`）。点击列头可按升序排序，再点一次则切换为降序。

如果你使用的是 `DataGridTemplateColumn`，并且需要自定义排序行为，请设置 `SortMemberPath`：

```xml
<DataGridTemplateColumn Header="Age" SortMemberPath="AgeInYears">
    <DataGridTemplateColumn.CellTemplate>
        <DataTemplate>
            <TextBlock Text="{Binding AgeInYears, StringFormat='{}{0} years'}" />
        </DataTemplate>
    </DataGridTemplateColumn.CellTemplate>
</DataGridTemplateColumn>
```

### 通过代码排序

```csharp
var column = myDataGrid.Columns[0];
myDataGrid.Columns.Clear();
myDataGrid.Columns.Add(column);
// 或者使用 CollectionView 排序（见下方筛选部分）
```

## 筛选

你可以在视图模型中维护一个筛选后的集合，并把 DataGrid 绑定到它：

```csharp
public partial class MainViewModel : ObservableObject
{
    private readonly List<Product> _allProducts;

    [ObservableProperty]
    private string _filterText = "";

    [ObservableProperty]
    private ObservableCollection<Product> _filteredProducts;

    public MainViewModel()
    {
        _allProducts = LoadProducts();
        _filteredProducts = new ObservableCollection<Product>(_allProducts);
    }

    partial void OnFilterTextChanged(string value)
    {
        var filtered = string.IsNullOrWhiteSpace(value)
            ? _allProducts
            : _allProducts.Where(p =>
                p.Name.Contains(value, StringComparison.OrdinalIgnoreCase));

        FilteredProducts = new ObservableCollection<Product>(filtered);
    }
}
```

```xml
<StackPanel Spacing="8">
    <TextBox Text="{Binding FilterText}" PlaceholderText="Search products..." />
    <DataGrid ItemsSource="{Binding FilteredProducts}" AutoGenerateColumns="True"
              IsReadOnly="True" />
</StackPanel>
```

## 分组

如果你想对行进行分组，可以先用 `DataGridCollectionView` 包装集合，再添加分组描述。这样 DataGrid 会自动为每个分组渲染一个可折叠的 `DataGridRowGroupHeader`。

### 基础分组

```csharp
using Avalonia.Collections;

public partial class MainViewModel : ObservableObject
{
    public DataGridCollectionView GroupedProducts { get; }

    public MainViewModel()
    {
        var products = new List<Product>
        {
            new("Widget", "Hardware", 9.99m),
            new("Gadget", "Hardware", 24.99m),
            new("App", "Software", 4.99m),
            new("Plugin", "Software", 14.50m),
        };

        GroupedProducts = new DataGridCollectionView(products);
        GroupedProducts.GroupDescriptions.Add(
            new DataGridPathGroupDescription("Category"));
    }
}
```

```xml
<DataGrid ItemsSource="{Binding GroupedProducts}" AutoGenerateColumns="False"
          IsReadOnly="True">
    <DataGrid.Columns>
        <DataGridTextColumn Header="Name" Binding="{Binding Name}" Width="*" />
        <DataGridTextColumn Header="Price" Binding="{Binding Price, StringFormat='{}{0:C}'}" Width="*" />
    </DataGrid.Columns>
</DataGrid>
```

### 多级分组

你可以添加多个 `DataGridPathGroupDescription` 来实现嵌套分组：

```csharp
GroupedProducts.GroupDescriptions.Add(new DataGridPathGroupDescription("Category"));
GroupedProducts.GroupDescriptions.Add(new DataGridPathGroupDescription("SubCategory"));
```

### 自定义分组标题

你可以处理 `LoadingRowGroup` 事件，以修改分组标题文本或追加汇总信息：

```csharp
private void OnLoadingRowGroup(object? sender, DataGridRowGroupHeaderEventArgs e)
{
    var group = e.RowGroupHeader.DataContext as DataGridCollectionViewGroup;
    if (group is null)
        return;

    e.RowGroupHeader.PropertyName = "Category";
    e.RowGroupHeader.PropertyValue = $"{group.Key} ({group.ItemCount} products)";
}
```

```xml
<DataGrid ItemsSource="{Binding GroupedProducts}"
          LoadingRowGroup="OnLoadingRowGroup" />
```

### 通过代码展开或折叠分组

可以直接调用 DataGrid 的 `ExpandRowGroup` 和 `CollapseRowGroup` 方法：

```csharp
if (viewModel.GroupedProducts.Groups is { } groups)
{
    foreach (var group in groups.OfType<DataGridCollectionViewGroup>())
    {
        myDataGrid.CollapseRowGroup(group, collapseAllSubgroups: true);
    }
}
```

## 列类型

| 列类型 | 用途 |
|---|---|
| `DataGridTextColumn` | 用于文本显示和编辑。 |
| `DataGridCheckBoxColumn` | 用于布尔值，支持 `bool?` 的三态显示。 |
| `DataGridTemplateColumn` | 用于借助任意控件实现自定义显示和编辑。 |

## 模板列

使用 `DataGridTemplateColumn` 可以实现自定义单元格渲染：

```xml
<DataGridTemplateColumn Header="Status">
    <DataGridTemplateColumn.CellTemplate>
        <DataTemplate>
            <Border Background="{Binding StatusColor}" CornerRadius="4"
                    Padding="8,2" HorizontalAlignment="Center">
                <TextBlock Text="{Binding Status}" Foreground="White"
                           FontSize="11" />
            </Border>
        </DataTemplate>
    </DataGridTemplateColumn.CellTemplate>
</DataGridTemplateColumn>
```

### Editable template column

Provide both `CellTemplate` (display) and `CellEditingTemplate` (editing):

```xml
<DataGridTemplateColumn Header="Rating">
    <DataGridTemplateColumn.CellTemplate>
        <DataTemplate>
            <TextBlock Text="{Binding Rating}" HorizontalAlignment="Center" />
        </DataTemplate>
    </DataGridTemplateColumn.CellTemplate>
    <DataGridTemplateColumn.CellEditingTemplate>
        <DataTemplate>
            <NumericUpDown Value="{Binding Rating}" Minimum="0" Maximum="5"
                           FormatString="N0" />
        </DataTemplate>
    </DataGridTemplateColumn.CellEditingTemplate>
</DataGridTemplateColumn>
```

## 选择

### 单选

```xml
<DataGrid ItemsSource="{Binding Products}"
          SelectedItem="{Binding SelectedProduct}"
          SelectionMode="Single" />
```

```csharp
[ObservableProperty]
private Product? _selectedProduct;

partial void OnSelectedProductChanged(Product? value)
{
    // 响应选择变化
}
```

### 多选

```xml
<DataGrid ItemsSource="{Binding Products}"
          SelectionMode="Extended" />
```

在 code-behind 中访问已选项：

```csharp
var selectedItems = myDataGrid.SelectedItems;
```

## 编辑

默认情况下，只要 `IsReadOnly` 为 `false`，DataGrid 就允许编辑。你可以通过双击单元格或按下 F2 进入编辑模式，按 Enter 提交，按 Escape 取消。

```xml
<DataGrid ItemsSource="{Binding Products}" IsReadOnly="False"
          CellEditEnding="OnCellEditEnding" />
```

### 处理编辑事件

```csharp
private void OnCellEditEnding(object? sender, DataGridCellEditEndingEventArgs e)
{
    if (e.EditAction == DataGridEditAction.Commit)
    {
        // 在这里验证或处理编辑结果
    }
}
```

## 列宽模式

| 宽度值 | 行为 |
|---|---|
| `Auto` | 根据内容自动调整宽度。 |
| `*` | 平分剩余空间。 |
| `2*` | 获得 `*` 列两倍的剩余空间份额。 |
| `200` | 固定为 200 像素宽。 |
| `SizeToCells` | 根据单元格内容调整宽度。 |
| `SizeToHeader` | 根据列头内容调整宽度。 |

```xml
<DataGrid.Columns>
    <DataGridTextColumn Header="Name" Width="2*" Binding="{Binding Name}" />
    <DataGridTextColumn Header="Code" Width="Auto" Binding="{Binding Code}" />
    <DataGridTextColumn Header="Price" Width="*" Binding="{Binding Price}" />
</DataGrid.Columns>
```

## 行详情

当某一行被选中时显示额外内容：

```xml
<DataGrid ItemsSource="{Binding Products}" IsReadOnly="True"
          RowDetailsVisibilityMode="VisibleWhenSelected">
    <DataGrid.RowDetailsTemplate>
        <DataTemplate>
            <Border Background="#F5F5F5" Padding="16" Margin="4">
                <StackPanel Spacing="4">
                    <TextBlock Text="{Binding Description}" TextWrapping="Wrap" />
                    <TextBlock Text="{Binding LastUpdated, StringFormat='Updated: {0:d}'}"
                               Foreground="Gray" FontSize="11" />
                </StackPanel>
            </Border>
        </DataTemplate>
    </DataGrid.RowDetailsTemplate>
    <DataGrid.Columns>
        <DataGridTextColumn Header="Name" Binding="{Binding Name}" Width="*" />
    </DataGrid.Columns>
</DataGrid>
```

## 网格线与交替行背景

```xml
<DataGrid ItemsSource="{Binding Products}"
          GridLinesVisibility="Horizontal"
          AlternatingRowBackground="#F8F8F8" />
```

| GridLinesVisibility | 说明 |
|---|---|
| `None` | 不显示网格线（默认）。 |
| `Horizontal` | 只显示水平线。 |
| `Vertical` | 只显示垂直线。 |
| `All` | 同时显示水平和垂直线。 |

## 冻结列

在水平滚动时保持部分列始终可见：

```xml
<DataGrid ItemsSource="{Binding Products}" FrozenColumnCount="1">
    <DataGrid.Columns>
        <DataGridTextColumn Header="ID" Binding="{Binding Id}" Width="60" />
        <!-- 这一列在滚动时始终保持可见 -->
        <DataGridTextColumn Header="Name" Binding="{Binding Name}" Width="200" />
        <DataGridTextColumn Header="Category" Binding="{Binding Category}" Width="200" />
        <!-- 其余列可以参与水平滚动 -->
    </DataGrid.Columns>
</DataGrid>
```

## 按条件设置行样式

你可以使用 `DataGridRowTheme`，或者在 code-behind 中处理 `LoadingRow`：

```csharp
private void OnLoadingRow(object? sender, DataGridRowEventArgs e)
{
    if (e.Row.DataContext is Product product && !product.InStock)
    {
        e.Row.Background = Brushes.MistyRose;
    }
    else
    {
        e.Row.Background = null;
    }
}
```

```xml
<DataGrid ItemsSource="{Binding Products}" LoadingRow="OnLoadingRow" />
```

## 另请参阅

- [DataGrid Control Reference](/controls/data-display/structured-data/datagrid)：控件安装与属性表说明。
- [TreeDataGrid](/controls/data-display/structured-data/treedatagrid)：适用于层级数据展示的控件。
- [Binding to Collections](/docs/data-binding/how-to-bind-to-a-collection)：`ObservableCollection` 使用模式。
- [Performance Optimization](/docs/app-development/performance#collection-performance)：大集合场景下的批量更新与虚拟化优化。
