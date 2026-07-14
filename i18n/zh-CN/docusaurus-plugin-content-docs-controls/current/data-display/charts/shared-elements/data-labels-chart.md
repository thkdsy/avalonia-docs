---
id: data-labels-chart
title: 数据标签
description: 直接在图表系列元素上显示实际数据值，无需用户根据轴位置估算数值。
doc-type: reference
tags:
  - avalonia pro
---

import chartsFeaturesLabels from '/img/controls/charts/charts-datalabel.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

数据标签将实际值直接放置在图表系列上。这让读者无需根据轴位置估算即可比较数值。

<Image light={chartsFeaturesLabels} maxWidth={400} position="center" cornerRadius="true" alt="柱状图在每个季度收入柱上方显示数据标签。" />

## 使用时机
- **演示图形**：当需要清晰、直接的数值时。
- **小多图**：当省略轴以节省空间时。
- **关键里程碑**：突出显示需要注意的特定值。

## 代码示例

### XAML
```xml
<CartesianChart xmlns="https://github.com/avaloniaui" Name="BasicLabelsChart" Height="250">
                        <CartesianChart.Series>
                            <BarSeries Title="Sales" ItemsSource="{Binding SalesData}"
                                                CategoryPath="Category" ValuePath="Value"
                                                ShowLabels="True" LabelOffset="5"/>
                        </CartesianChart.Series>
                        <CartesianChart.HorizontalAxis>
                            <CategoryAxis Title="Quarter" />
                        </CartesianChart.HorizontalAxis>
                        <CartesianChart.VerticalAxis>
                            <NumericalAxis Title="Revenue" />
                        </CartesianChart.VerticalAxis>
                    </CartesianChart>
```

### 数据模型 (C#)
```csharp
public record SalesPoint(string Category, double Value);

public ObservableCollection<SalesPoint> SalesData { get; } = new()
{
    new("Q1", 120),
    new("Q2", 150),
    new("Q3", 180),
    new("Q4", 220)
};
```

## 常用属性（在 Series 上）

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ShowLabels` | 切换系列点上数值的显示。 | `false` |
| `LabelFormat` | 字符串格式。`{0}` 是值，`{1}` 是分类。 | `"{0:N0}"` |
| `LabelFontSize` | 文本大小（像素）。 | `12.0` |
| `LabelForeground` | 文本颜色使用的画刷。 | `null` |
| `LabelBackground` | 标签文本背景使用的画刷。 | `null` |
| `LabelCornerRadius` | 标签背景的圆角半径。 | `4` |
| `LabelPadding` | 标签背景内的内边距。 | `4,2` |
| `LabelOffset` | 从数据点到标签的距离。 | `10.0` |
