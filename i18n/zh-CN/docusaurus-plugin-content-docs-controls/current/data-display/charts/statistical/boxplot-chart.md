---
id: boxplot-chart
title: 箱线图
description: 使用盒须布局对数据分布进行图形化摘要，显示中位数、四分位数和异常值，适用于统计比较。
doc-type: reference
tags:
  - avalonia pro
---

import chartsCartesianBoxplot from '/img/controls/charts/charts-cartesian-boxplot.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

箱线图（或盒须图）提供数据分布、集中趋势和变异性的图形化摘要，包括四分位数和异常值。

<Image light={chartsCartesianBoxplot} maxWidth={400} position="center" cornerRadius="true" alt="箱线图，每个分类显示盒须符号，包括中位数、四分位数和异常数据点。" />

## 使用时机
- **统计分析**：比较多个数据集的分布（例如不同班级的测试分数）。
- **异常值检测**：识别落在"须"之外的极端数据点。
- **范围可视化**：一目了然地显示最小值、最大值、中位数和四分位距。

## 代码示例

### XAML
```xml
<CartesianChart xmlns="https://github.com/avaloniaui" Name="BoxPlotSample" Title="箱线图（盒须图）" Height="250">
    <CartesianChart.HorizontalAxis>
        <CategoryAxis />
    </CartesianChart.HorizontalAxis>
    <CartesianChart.VerticalAxis>
        <NumericalAxis />
    </CartesianChart.VerticalAxis>
    <CartesianChart.Series>
        <BoxPlotSeries Title="分布"
                                ItemsSource="{Binding BoxPlotData}"
                                CategoryPath="Category"
                                MinPath="Min" Q1Path="Q1" MedianPath="Median" Q3Path="Q3" MaxPath="Max"
                                Fill="#7E9C27B0" />
    </CartesianChart.Series>
</CartesianChart>
```

### 数据模型 (C#)
```csharp
public record BoxPlotPoint(string Category, double Min, double Q1, double Median, double Q3, double Max);

public ObservableCollection<BoxPlotPoint> BoxPlotData { get; } = new()
{
    new("Q1", 10, 25, 35, 45, 60),
    new("Q2", 15, 30, 42, 55, 70),
    new("Q3", 20, 35, 48, 60, 75),
    new("Q4", 25, 40, 52, 65, 80)
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 统计数据点的集合。 | `null` |
| `MinPath` | 最小值的路径。 | `null` |
| `MaxPath` | 最大值的路径。 | `null` |
| `MedianPath` | 中位数的路径。 | `null` |
| `Q1Path` | 第一四分位数（第 25 百分位）的路径。 | `null` |
| `Q3Path` | 第三四分位数（第 75 百分位）的路径。 | `null` |
| `BoxWidth` | 箱体宽度，占分类槽位的比例。 | `0.6` |
| `MedianStroke` | 箱体内中位线使用的画刷。 | `null` |
| `WhiskerThickness` | 须线的粗细。 | `1.0` |
| `Fill` | 用于"箱体"的画刷。 | 取决于主题 |
| `Stroke` | 须线和箱体轮廓的颜色。 | 取决于主题 |
