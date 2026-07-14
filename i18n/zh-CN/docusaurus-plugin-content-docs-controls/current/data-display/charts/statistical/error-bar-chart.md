---
id: error-bar-chart
title: 误差线图
description: 通过向数据点添加误差指示器来表示数据变异性，显示标准差、置信区间或测量不确定度。
doc-type: reference
tags:
  - avalonia pro
---

import chartsStatisticalErrorbar from '/img/controls/charts/charts-statistical-error.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

误差线图表示数据的变异性，用于在图表中指示报告测量值的误差或不确定度。

<Image light={chartsStatisticalErrorbar} maxWidth={400} position="center" cornerRadius="true" alt="带有数据点和垂直误差指示器的图表，显示每个样本的标准差范围。" />

## 使用时机
- **科学研究**：可视化标准差或置信区间。
- **质量控制**：显示制造中的公差范围。
- **调查数据**：指示统计民意调查中的误差范围。

## 代码示例

### XAML
```xml
<CartesianChart xmlns="https://github.com/avaloniaui" Name="ErrorBarChartSample" Title="测量不确定度" Height="250">
    <CartesianChart.HorizontalAxis><CategoryAxis /></CartesianChart.HorizontalAxis>
    <CartesianChart.VerticalAxis><NumericalAxis /></CartesianChart.VerticalAxis>
    <CartesianChart.Series>
        <ErrorBarSeries Title="测量值"
                                 CategoryPath="Sample"
                                 ValuePath="Value"
                                 ErrorPath="Error"
                                 CapWidth="10"
                                 ShowMarkers="True"
                                 MarkerSize="8"
                                 ItemsSource="{Binding ErrorBarData}" />
    </CartesianChart.Series>
</CartesianChart>
```

### 数据模型 (C#)
```csharp
public record ErrorBarItem(string Sample, double Value, double Error);

public ObservableCollection<ErrorBarItem> ErrorBarData { get; } = new()
{
    new("A", 45.0, 5.0),
    new("B", 62.0, 8.0),
    new("C", 38.0, 4.0),
    new("D", 75.0, 10.0),
    new("E", 55.0, 6.0)
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 测量数据的集合。 | `null` |
| `ValuePath` | 中心值（均值/中位数）。 | `null` |
| `ErrorPath` | 应用于值上方和下方的对称误差量。 | `null` |
| `LowErrorPath` | 非对称误差线的低端误差量。 | `null` |
| `HighErrorPath` | 非对称误差线的高端误差量。 | `null` |
| `ErrorMode` | 是否在值上方、下方或两侧绘制误差线。 | `Both` |
| `CapWidth` | 误差线上水平端盖的宽度。 | `8` |
| `ShowMarkers` | 是否在每个中心值处绘制标记。 | `false` |
| `MarkerSize` | 中心标记的大小。 | `8` |
| `Stroke` | 误差指示线的颜色。 | 取决于主题 |
