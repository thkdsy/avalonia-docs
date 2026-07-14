---
id: area-chart
title: 面积图
description: 在折线图的基础上填充线条下方的区域，使用颜色或渐变突出随时间变化的幅度。
doc-type: reference
tags:
  - avalonia pro
---

import chartsCartesianArea from '/img/controls/charts/charts-cartesian-area.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

面积图基于折线图。坐标轴与线条之间的区域填充颜色或渐变，突出随时间变化的幅度。

<Image light={chartsCartesianArea} maxWidth={400} position="center" cornerRadius="true" alt="面积图显示网站流量数据，线条与水平轴之间带有渐变填充。" />

## 何时使用
- **累积总量**：可视化不同组成部分随时间对整体的贡献。
- **体积**：突出数据点的总体积或幅度。
- **视觉对比**：提供比简单折线图更鲜明的视觉表现。

## 代码示例

### XAML
```xml
<CartesianChart xmlns="https://github.com/avaloniaui" Name="AreaChart" Title="面积图" Height="250">
                        <CartesianChart.HorizontalAxis>
                            <CategoryAxis />
                        </CartesianChart.HorizontalAxis>
                        <CartesianChart.VerticalAxis>
                            <NumericalAxis />
                        </CartesianChart.VerticalAxis>
                        <CartesianChart.Series>
                            <AreaSeries Title="收入" ItemsSource="{Binding AreaSeriesData}" Fill="#7E4CAF50" Stroke="Green" StrokeThickness="2" />
                        </CartesianChart.Series>
                    </CartesianChart>
```

### 数据模型 (C#)
```csharp
public ObservableCollection<int> AreaSeriesData { get; } = new()
{
    120, 150, 135, 180, 165, 200, 185, 220
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Title` | 系列名称。 | `null` |
| `ItemsSource` | 数据项集合。 | `null` |
| `Stroke` | 顶部线条的颜色。 | 取决于主题 |
| `Fill` | 用于填充线条下方区域的画刷。 | 取决于主题 |
| `FillOpacity` | 填充的透明度（0.0 到 1.0）。 | `0.5` |
| `StrokeThickness`| 线条的宽度。 | `2` |
