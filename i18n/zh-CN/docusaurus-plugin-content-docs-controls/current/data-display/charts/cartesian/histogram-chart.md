---
id: histogram-chart
title: 直方图
description: 将连续数据分组到"桶"中，显示每个桶内数据点的频率，以揭示单个变量的分布情况。
doc-type: reference
tags:
  - avalonia pro
---

import chartsStatisticalHistogram from '/img/controls/charts/charts-statistical-histogram.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

直方图将连续数据分组到"桶"中，并显示每个桶内数据点的频率。它们对于理解单个变量的分布至关重要。

<Image light={chartsStatisticalHistogram} maxWidth={400} position="center" cornerRadius="true" alt="直方图将连续数据分组到桶中，展示值的频率分布。" />

## 何时使用
- **年龄分布**：可视化有多少用户属于特定年龄段。
- **性能日志**：分析系统中响应时间的频率。
- **质量保证**：评估产品尺寸或重量的分布范围。

## 代码示例

### XAML
```xml
<CartesianChart xmlns="https://github.com/avaloniaui" Name="HistogramSample" Title="分数分布" Height="250">
                        <CartesianChart.HorizontalAxis><CategoryAxis /></CartesianChart.HorizontalAxis>
                        <CartesianChart.VerticalAxis><NumericalAxis /></CartesianChart.VerticalAxis>
                        <CartesianChart.Series>
                            <HistogramSeries ItemsSource="{Binding HistogramData}" ValuePath="Score" BinCount="10" />
                        </CartesianChart.Series>
                    </CartesianChart>
```

### 数据模型 (C#)
```csharp
using System;
using System.Linq;

public record HistogramItem(double Score);

public ObservableCollection<HistogramItem> HistogramData { get; } = CreateScores();

private static ObservableCollection<HistogramItem> CreateScores()
{
    var random = new Random(42);

    return new ObservableCollection<HistogramItem>(
        Enumerable.Range(0, 50)
            .Select(_ => new HistogramItem(50 + random.NextDouble() * 50)));
}
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 原始数据点集合。 | `null` |
| `ValuePath` | 要分组的数值属性路径。 | `null` |
| `BinCount` | 要创建的柱条（范围）数量。 | `10` |
| `BinWidth` | （可选）每个范围的显式宽度；设置后覆盖 BinCount。 | `null` |
| `Fill` | 用于频率柱条的画刷。 | 取决于主题 |
| `BarWidth` | 每个直方图柱条的宽度，以桶宽度的比例表示。 | `0.9` |
| `BarCornerRadius` | 直方图柱条的圆角半径。 | `CornerRadius(2,2,0,0)` |
