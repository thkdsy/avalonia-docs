---
id: spline-chart
title: 样条图
description: 与折线图类似，但使用平滑的多项式曲线连接数据点，使趋势呈现更自然的视觉效果。
doc-type: reference
tags:
  - avalonia pro
---

import chartsCartesianSpline from '/img/controls/charts/charts-cartesian-spline.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

样条图与折线图类似，但使用平滑的多项式曲线连接数据点，为数据趋势提供更"有机"的感觉。

<Image light={chartsCartesianSpline} maxWidth={400} position="center" cornerRadius="true" alt="样条图，使用平滑曲线连接各时间间隔的温度数据点。" />

## 何时使用
- **平滑数据**：可视化连续且平滑变化的数据时（例如温度）。
- **美观性**：当需要专业、圆润的外观而非尖锐的角度时。
- **趋势平滑**：帮助可视化总体趋势，避免直线段的生硬感。

## 代码示例

### XAML
```xml
<CartesianChart xmlns="https://github.com/avaloniaui" Name="SplineChart" Title="样条图" Height="250">
                        <CartesianChart.HorizontalAxis>
                            <CategoryAxis />
                        </CartesianChart.HorizontalAxis>
                        <CartesianChart.VerticalAxis>
                            <NumericalAxis />
                        </CartesianChart.VerticalAxis>
                        <CartesianChart.Series>
                            <SplineSeries Title="温度" ItemsSource="{Binding SplineSeriesData}" Stroke="Crimson" StrokeThickness="3" MarkerSize="6" MarkerFill="White"  ShowMarkers="True"/>
                        </CartesianChart.Series>
                    </CartesianChart>
```

### 数据模型 (C#)
```csharp
public ObservableCollection<int> SplineSeriesData { get; } = new()
{
    15, 18, 22, 28, 32, 35, 33, 28, 22, 17, 12, 10
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Title` | 系列名称。 | `null` |
| `ItemsSource` | 数据项集合。 | `null` |
| `Stroke` | 样条曲线的颜色。 | 取决于主题 |
| `StrokeThickness` | 曲线的宽度。 | `2` |
| `ShowMarkers` | 是否在数据点显示标记。标记样式仅在此为 `true` 时可见。 | `false` |
| `MarkerSize` | 标记的大小（像素）。 | `6` |
| `MarkerShape` | 标记的形状，例如 `Circle` 或 `Square`。 | `Circle` |
| `MarkerFill` | 用于填充标记的画刷。 | `null` |
| `MarkerStroke` | 用于标记轮廓的画刷。 | `null` |
| `MarkerStrokeThickness` | 标记轮廓的粗细。当为 `NaN` 时，使用系列的 `StrokeThickness`。 | `NaN` |
| `SplineTension` | 控制曲线的平滑度。 | `0.25` |
