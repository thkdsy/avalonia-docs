---
id: scatter-chart
title: 散点图
description: 使用两个数值变量将数据点绘制为圆点，以揭示数据集中的相关性、分布和异常值。
doc-type: reference
tags:
  - avalonia pro
---

import chartsCartesianScatter from '/img/controls/charts/charts-cartesian-scatter.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

散点图使用圆点表示两个不同数值变量的值。它们对于显示和比较数值数据（如科学、统计和工程数据）至关重要。

<Image light={chartsCartesianScatter} maxWidth={400} position="center" cornerRadius="true" alt="散点图将单个数据点绘制为两个数值轴上的圆点，以揭示相关性。" />

## 何时使用
- **相关性**：识别两个变量之间的关系（例如身高与体重）。
- **分布**：可视化数据点的分布和聚类情况。
- **异常值检测**：轻松发现远离常态的数据点。

## 代码示例

### XAML
```xml
<CartesianChart xmlns="https://github.com/avaloniaui" Name="ScatterChart" Title="散点图" Height="250">
    <CartesianChart.HorizontalAxis>
        <NumericalAxis />
    </CartesianChart.HorizontalAxis>
    <CartesianChart.VerticalAxis>
        <NumericalAxis />
    </CartesianChart.VerticalAxis>
    <CartesianChart.Series>
        <ScatterSeries Title="数据点"
                              ItemsSource="{Binding ScatterSeriesData}"
                              Fill="Purple"
                              MarkerSize="10"
                              MarkerShape="Circle" />
    </CartesianChart.Series>
</CartesianChart>
```

### 数据模型 (C#)
```csharp
public ObservableCollection<int> ScatterSeriesData { get; } =
    new() { 25, 45, 35, 55, 40, 60, 50, 70, 55, 75 };
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Title` | 系列名称。 | `null` |
| `ItemsSource` | 数据点集合。 | `null` |
| `CategoryPath` | 品类路径的值（X 轴）。 | `null` |
| `ValuePath` | 值路径的值（Y 轴）。 | `null` |
| `ShowMarkers` | 是否显示数据点标记。 | `true` |
| `MarkerSize` | 圆点的大小（像素）。 | `8` |
| `MarkerShape` | 圆点的形状，例如 `Circle` 或 `Square`。 | `Circle` |
| `MarkerFill` | 用于填充标记的画刷。当为 `null` 时，使用 `Fill`。 | `null` |
| `MarkerStroke` | 用于标记轮廓的画刷。 | `null` |
| `MarkerStrokeThickness` | 标记轮廓的粗细。当为 `NaN` 时，使用系列的 `StrokeThickness`。 | `NaN` |
| `Fill` | 散点圆点的颜色。 | 取决于主题 |

## 散点折线变体

`ScatterLineSeries` 通过绘制数据点之间的连接线扩展散点图，结合了散点图的可视性和折线图的趋势展示。

### XAML
```xml
<CartesianChart xmlns="https://github.com/avaloniaui" Name="ScatterChart" Title="散点图" Height="250">
                        <CartesianChart.HorizontalAxis>
                            <NumericalAxis />
                        </CartesianChart.HorizontalAxis>
                        <CartesianChart.VerticalAxis>
                            <NumericalAxis />
                        </CartesianChart.VerticalAxis>
                        <CartesianChart.Series>
                            <ScatterLineSeries Title="数据点" ItemsSource="{Binding ScatterSeriesData}" Fill="Purple" MarkerSize="10" MarkerShape="Circle" />
                        </CartesianChart.Series>
                    </CartesianChart>
```

### ScatterLineSeries 属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ShowLines` | 是否显示散点之间的连接线。 | `true` |
| `StrokeDashStyle` | 连接线的虚线样式。当为 `null` 时，线条为实线。 | `null` |
| `ShowMarkers` | 是否显示数据点标记。 | `true` |
| `MarkerSize` | 标记的大小（像素）。 | `8` |
| `MarkerShape` | 标记的形状，例如 `Circle` 或 `Square`。 | `Circle` |
| `MarkerFill` | 用于填充标记的画刷。当为 `null` 时，使用 `Fill`。 | `null` |
| `MarkerStroke` | 用于标记轮廓的画刷。 | `null` |
| `MarkerStrokeThickness` | 标记轮廓的粗细。当为 `NaN` 时，使用系列的 `StrokeThickness`。 | `NaN` |

## 另请参阅

- [折线图](/controls/data-display/charts/cartesian/line-chart)
- [点图](/controls/data-display/charts/cartesian/dot-plot-chart)
- [组合图](/controls/data-display/charts/cartesian/combo-chart)
