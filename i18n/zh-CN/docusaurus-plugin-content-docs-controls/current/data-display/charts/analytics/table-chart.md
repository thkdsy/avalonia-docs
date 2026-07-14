---
id: table-chart
title: 表格图
description: 将表格数据与嵌入式视觉提示（如色阶）相结合，适用于需要精确数值的密集型报表。
doc-type: reference
tags:
  - avalonia pro
---

import chartsAnalyticsTable from '/img/controls/charts/charts-analytics-table.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

表格图将传统表格数据与嵌入式视觉提示相结合。它们适用于需要精确数值和快速视觉比较的密集型报表。

<Image light={chartsAnalyticsTable} maxWidth={400} position="center" cornerRadius="true" alt="表格图，显示带有颜色编码指示符的数据行，用于视觉比较。" />

## 使用场景
- **产品对比**：以网格形式展示多个产品的特性。
- **财务状况**：以颜色编码的"健康"指示器显示账户信息。
- **多指标报表**：当用户需要在紧凑布局中比较多个指标时使用。

## 代码示例

### XAML
```xml
<TableChart xmlns="https://github.com/avaloniaui" Title="Product Comparison" Height="400"
                                         ItemsSource="{Binding TableData}"
                                         Columns="{Binding TableColumns}"
                                         RowLabelPath="Product" />
```

### 数据模型 (C#)
```csharp
using Avalonia.Controls.Charts;
using Avalonia.Media;

public record TableItem(
    string Product,
    double Sales,
    double Price,
    double Sweetness,
    double Juiciness,
    double Acidity);

public ObservableCollection<TableItem> TableData { get; } = new()
{
    new("Apple", 200, 1.2, 6.8, 7.0, 4.5),
    new("Banana", 180, 0.5, 7.0, 8.5, 3.0),
    new("Orange", 150, 1.0, 5.5, 9.0, 6.0),
    new("Grape", 140, 2.5, 6.5, 6.0, 4.0),
    new("Pineapple", 130, 1.8, 6.0, 7.5, 5.0),
    new("Blueberry", 120, 3.0, 5.0, 6.5, 4.0),
    new("Mango", 110, 1.5, 8.5, 8.0, 2.5),
    new("Strawberry", 100, 2.0, 8.0, 5.0, 3.5)
};

public ObservableCollection<TableChartColumn> TableColumns { get; } = new()
{
    new()
    {
        Header = "Avg Sales\n(units/mo)",
        ValuePath = "Sales",
        UseColorScale = true,
        MinValue = 100,
        MaxValue = 200,
        LowBrush = new SolidColorBrush(Color.FromRgb(220, 235, 255)),
        HighBrush = new SolidColorBrush(Color.FromRgb(33, 150, 243))
    },
    new()
    {
        Header = "Avg Price\n($)",
        ValuePath = "Price",
        Format = "C1",
        UseColorScale = true,
        MinValue = 0.5,
        MaxValue = 3.0,
        LowBrush = new SolidColorBrush(Color.FromRgb(255, 235, 220)),
        HighBrush = new SolidColorBrush(Color.FromRgb(255, 152, 0))
    },
    new() { Header = "Sweetness\n(0-10)", ValuePath = "Sweetness" },
    new() { Header = "Juiciness\n(0-10)", ValuePath = "Juiciness" },
    new() { Header = "Acidity\n(0-10)", ValuePath = "Acidity" }
};
```

`Columns` 接受 `TableChartColumn` 对象。每个列可以定义 `Header`（标题）、`ValuePath`（值路径）、`Format`（格式）和可选的色阶设置。

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 行数据源。 | `null` |
| `RowLabelPath` | 左侧行标题列中显示的文本路径。 | `null` |
| `Columns` | 网格列的配置。 | `null` |
| `RowHeight` | 每行数据的高度。 | `40.0` |
| `ColumnWidth` | 每列指标的最小宽度。 | `80.0` |
| `RowLabelWidth` | 行标签列的宽度。 | `100.0` |
| `HeaderHeight` | 标题行的高度。 | `50.0` |
| `ShowGridLines`| 是否显示行和列之间的网格线。 | `true` |
| `CellPadding` | 每个表格单元格的内边距。 | `5.0` |
| `LabelFontSize` | 标题和单元格值的字体大小。 | `12.0` |
