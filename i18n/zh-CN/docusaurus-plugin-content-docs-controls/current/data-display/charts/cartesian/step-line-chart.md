---
id: step-line-chart
title: 阶梯折线图
description: 使用水平线和垂直线以阶梯模式连接数据点，表示在间隔之间保持恒定的值。
doc-type: reference
tags:
  - avalonia pro
---

import chartsCartesianStepline from '/img/controls/charts/charts-cartesian-stepline.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

阶梯折线图使用水平线和垂直线连接数据点，创建类似阶梯的模式。它们适用于显示在离散间隔发生的变化。

<Image light={chartsCartesianStepline} maxWidth={400} position="center" cornerRadius="true" alt="阶梯折线图，使用水平和垂直段连接数据点，显示值在数值间隔处的变化。" />

## 何时使用
- **价格变化**：可视化利率、价格层级或库存水平。
- **离散转换**：当值在数据点之间保持不变时。
- **数字信号**：表示二进制或基于状态的数据。

## 代码示例

### XAML
```xml
<CartesianChart xmlns="https://github.com/avaloniaui" Name="StepLineChart" Title="阶梯折线图" Height="250">
                        <CartesianChart.HorizontalAxis>
                            <CategoryAxis />
                        </CartesianChart.HorizontalAxis>
                        <CartesianChart.VerticalAxis>
                            <NumericalAxis />
                        </CartesianChart.VerticalAxis>
                        <CartesianChart.Series>
                            <StepLineSeries Title="价格" ItemsSource="{Binding StepLineSeriesData}" Stroke="Teal" StrokeThickness="2" MarkerSize="6"  ShowMarkers="True"/>
                        </CartesianChart.Series>
                    </CartesianChart>
```

### 数据模型 (C#)
```csharp
public ObservableCollection<int> StepLineSeriesData { get; } =
    new() { 100, 100, 120, 120, 120, 140, 140, 160 };
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Title` | 系列名称。 | `null` |
| `ItemsSource` | 数据项集合。 | `null` |
| `CategoryPath` | 用于 X 轴值的属性路径。 | `null` |
| `ValuePath` | 用于 Y 轴值的属性路径。 | `null` |
| `Stroke` | 阶梯线的颜色。 | 取决于主题 |
| `StrokeThickness` | 线条的宽度。 | `2` |
| `ShowMarkers` | 是否在每个数据点显示标记。标记样式仅在此为 `true` 时可见。 | `false` |
| `MarkerSize` | 标记的大小（像素）。 | `6` |
| `MarkerShape` | 标记的形状，例如 `Circle` 或 `Square`。 | `Circle` |
| `MarkerFill` | 用于填充标记的画刷。 | `null` |
| `MarkerStroke` | 用于标记轮廓的画刷。 | `null` |
| `MarkerStrokeThickness` | 标记轮廓的粗细。当为 `NaN` 时，使用系列的 `StrokeThickness`。 | `NaN` |
| `StepMode` | `HorizontalFirst`（水平优先）、`VerticalFirst`（垂直优先）或 `Center`（居中）。 | `HorizontalFirst` |
