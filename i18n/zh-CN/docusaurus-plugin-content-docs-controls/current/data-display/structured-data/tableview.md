---
id: tableview
title: TableView
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

`TableView` 可在可配置的列中显示项目集合。它是一个只读的表格控件：它展示数据，但不提供单元格内容的就地编辑功能。

`TableView` 派生自 [`ListBox`](/controls/data-display/collections/listbox)，因此它复用相同的 `ItemsSource` 和 `SelectionModel`。每一行是一个 `TableViewRow`，每一列是一个 `TableViewColumn`。

:::info
`TableView` 是核心 **Avalonia.Controls** 包的一部分。无需额外的 NuGet 包或样式引用。自 Avalonia 12.1 起可用。
:::

## 基本用法

将 `ItemsSource` 属性绑定到视图模型中的集合，然后在 `TableView.Columns` 中为每列声明一个 `TableViewColumn`。使用列的 `Binding` 属性来选择每个单元格中显示的值：

<Tabs
  defaultValue="xaml"
  values={[
      { label: 'XAML 视图', value: 'xaml', },
      { label: 'C# 视图模型', value: 'viewmodel', },
      { label: 'C# 项目类', value: 'item', },
  ]}
>

<TabItem value="xaml">

```xml
<TableView ItemsSource="{Binding Countries}">
  <TableView.Columns>
    <TableViewColumn Header="名称" Binding="{Binding Name}" />
    <TableViewColumn Header="地区" Binding="{Binding Region}" />
    <TableViewColumn Header="人口"
                     Binding="{Binding Population, StringFormat=N0}"
                     HorizontalContentAlignment="Right" />
  </TableView.Columns>
</TableView>
```

</TabItem>

<TabItem value="viewmodel">

```csharp
using System.Collections.ObjectModel;

public class MainWindowViewModel
{
    public ObservableCollection<Country> Countries { get; } =
    [
        new("阿富汗", "亚洲", 31056997),
        new("阿尔巴尼亚", "东欧", 3581655),
        new("阿尔及利亚", "北非", 32930091),
    ];
}
```

</TabItem>

<TabItem value="item">

```csharp
public record Country(string Name, string Region, int Population);
```

</TabItem>

</Tabs>

:::info
这些示例使用 MVVM 模式，通过数据绑定到 `ObservableCollection`。有关数据绑定概念的更多信息，请参阅[数据绑定简介](/docs/data-binding/introduction-to-data-binding)。
:::

## 常用属性

您可能会经常使用以下 `TableView` 属性：

| 属性                   | 描述                                                                                                                            |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| `ItemsSource`          | 绑定的集合，用作行的数据源。                                                                                                    |
| `Columns`              | `TableViewColumn` 对象的集合，用于定义每列的显示方式。请参阅[列](#columns)。                                                   |
| `CanUserResizeColumns` | 用户是否可以通过拖动列表头之间的分隔符来调整列宽。默认为 `true`。                                                              |

由于 `TableView` 派生自 `ListBox`，标准的选择成员也同样适用；例如 `SelectionMode`、`SelectedItem`、`SelectedItems` 和 `SelectedIndex`。详情请参阅 [ListBox](/controls/data-display/collections/listbox)。

## 列

`TableView` 由 `Columns` 集合组成。每个 `TableViewColumn` 描述该列表头和对应的数据单元格。

### 显示单元格值

有两种方式可以确定单元格显示的内容：

| 属性           | 描述                                                                                                                                               |
| -------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Binding`      | 通过绑定从行的数据项中获取单元格值。适用于简单的属性显示，例如 `Binding="{Binding Name}"`。                                                         |
| `CellTemplate` | 使用数据模板构建单元格内容。整行数据项作为模板的数据上下文传递，因此您可以绑定到任何属性。                                                           |

`CellTemplate` 优先于 `Binding`。对于纯文本值使用 `Binding`，当需要图像、按钮或组合多个属性等更丰富的内容时使用 `CellTemplate`：

```xml
<TableView ItemsSource="{Binding Countries}">
  <TableView.Columns>
    <TableViewColumn Header="名称" Binding="{Binding Name}" />
    <TableViewColumn Header="人口">
      <TableViewColumn.CellTemplate>
        <DataTemplate>
          <ProgressBar Minimum="0" Maximum="1500000000"
                       Value="{Binding Population}"
                       VerticalAlignment="Center" />
        </DataTemplate>
      </TableViewColumn.CellTemplate>
    </TableViewColumn>
  </TableView.Columns>
</TableView>
```

### 列属性

| 属性                         | 描述                                                                                                                       |
| ---------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| `Header`                     | 列标题中显示的内容。                                                                                                       |
| `HeaderTemplate`             | 用于显示标题内容的数据模板。                                                                                               |
| `HeaderTheme`                | 应用于标题的 `ControlTheme`。它必须以 `TableViewColumnHeader` 为目标。                                                     |
| `Binding`                    | 从行数据项中读取单元格值的绑定（[见上文](#displaying-cell-values)）。                                                      |
| `CellTemplate`               | 单元格内容的数据模板，接收行数据项作为其数据上下文（[见上文](#displaying-cell-values)）。                                  |
| `CellTheme`                  | 应用于单元格的 `ControlTheme`。它必须以 `TableViewCell` 为目标。                                                           |
| `Width`                      | 列宽，以 `GridLength` 表示（[见下文](#column-width)）。默认为 `1*`。                                                       |
| `CanUserResize`              | 此特定列是否可以调整大小。当保留为默认值（`null`）时，该值回退到 `TableView` 的 `CanUserResizeColumns` 属性（[见上文](#useful-properties)）。 |
| `HorizontalContentAlignment` | 列表头和列单元格内内容的水平对齐方式。默认为 `Left`。                                                                     |

### 列宽

`Width` 属性是一个 `GridLength`，因此列可以使用绝对单位或相对单位进行大小调整，就像 [Grid](/controls/layout/panels/grid) 列一样：

- **星号**（`*`）：列占用剩余空间的比例份额。这是默认值（`1*`）。
- **像素**：以设备无关像素为单位的绝对宽度。

```xml
<TableView.Columns>
  <TableViewColumn Header="名称" Width="3*" Binding="{Binding Name}" />
  <TableViewColumn Header="地区" Width="2*" Binding="{Binding Region}" />
  <TableViewColumn Header="代码" Width="80" Binding="{Binding Code}" />
</TableView.Columns>
```

### 调整列宽

默认情况下，用户可以通过拖动两个列表头之间的分隔符来调整列宽。要为整个 `TableView` 关闭此功能，请将 `CanUserResizeColumns` 设置为 `False`。

```xml
<TableView ItemsSource="{Binding Countries}"
           CanUserResizeColumns="False">
  <!-- ... -->
</TableView>
```

您还可以使用 `CanUserResize` 覆盖单个列的行为。当其保留为默认值 `null` 时，该列遵循整个 `TableView` 的 `CanUserResizeColumns` 设置。将其更改为 `True` 或 `False` 可以仅覆盖该列的设置：

```xml
<TableView.Columns>
  <!-- 固定宽度，不可调整大小，无论表格设置如何 -->
  <TableViewColumn Header="#" Width="40"
                   Binding="{Binding Index}"
                   CanUserResize="False" />
  <TableViewColumn Header="名称" Binding="{Binding Name}" />
</TableView.Columns>
```

:::info
拖动调整器会将列切换为像素宽度。
:::

## 虚拟化

与 `ListBox` 类似，行默认会被虚拟化和回收。单元格也会随其所属行一起被回收。

:::warning
`TableView` 会虚拟化其行，但**不会**虚拟化列。每一列始终会被实例化，因此请将列数保持在合理范围内。
:::

## 另请参阅

- [ListBox](/controls/data-display/collections/listbox)
- [TableView API 参考](https://api-docs.avaloniaui.net/docs/T_Avalonia_Controls_TableView)
