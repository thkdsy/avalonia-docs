---
id: markers-chart
title: 标记
description: 在系列中每个数据点处绘制的符号，帮助定位精确坐标，支持多种形状以及可自定义的大小和颜色。
doc-type: reference
tags:
  - avalonia pro
---

import chartsFeaturesMarkers from '/img/controls/charts/charts-markers.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

标记是在系列中每个数据点处绘制的符号。它们帮助用户定位点的精确坐标，尤其是在路径可能密集的折线图或样条线图上。

对于笛卡尔折线系列（如 `LineSeries`、`SplineSeries` 和 `StepLineSeries`），标记是可选加入的。在标记样式属性（如 `MarkerSize`、`MarkerFill` 或 `MarkerStroke`）产生可见效果之前，需要设置 `ShowMarkers="True"`。

<Image light={chartsFeaturesMarkers} maxWidth={400} position="center" cornerRadius="true" alt="折线图在每个数据点处使用不同形状的标记来突出显示精确的坐标位置。" />

## 使用时机

- **离散数据**：强调线条代表单个测量点。
- **低密度图表**：使点更易于点击以获取工具提示或进行选择。
- **分类区分**：使用不同的形状（圆形、方形、菱形）来区分系列。

## 代码示例

### XAML

```xml
<CartesianChart xmlns="https://github.com/avaloniaui" Name="AllMarkersChart" Title="标记形状比较" Height="320" ShowLegend="True" LegendPosition="Bottom" HorizontalAlignment="Stretch">
                        <CartesianChart.Series>
                            <LineSeries Title="圆形" ItemsSource="{Binding CircleMarkerData}"
                                               MarkerShape="Circle" MarkerSize="10"
                                               Stroke="DodgerBlue" MarkerFill="DodgerBlue" StrokeThickness="2"  ShowMarkers="True"/>
                            <LineSeries Title="方形" ItemsSource="{Binding SquareMarkerData}"
                                               MarkerShape="Square" MarkerSize="10"
                                               Stroke="Green" MarkerFill="Green" StrokeThickness="2"  ShowMarkers="True"/>
                            <LineSeries Title="菱形" ItemsSource="{Binding DiamondMarkerData}"
                                               MarkerShape="Diamond" MarkerSize="12"
                                               Stroke="Orange" MarkerFill="Orange" StrokeThickness="2"  ShowMarkers="True"/>
                            <LineSeries Title="三角形" ItemsSource="{Binding TriangleMarkerData}"
                                               MarkerShape="Triangle" MarkerSize="10"
                                               Stroke="Purple" MarkerFill="Purple" StrokeThickness="2"  ShowMarkers="True"/>
                            <LineSeries Title="五边形" ItemsSource="{Binding PentagonMarkerData}"
                                               MarkerShape="Pentagon" MarkerSize="10"
                                               Stroke="Crimson" MarkerFill="Crimson" StrokeThickness="2"  ShowMarkers="True"/>
                        </CartesianChart.Series>
                        <CartesianChart.HorizontalAxis>
                            <CategoryAxis />
                        </CartesianChart.HorizontalAxis>
                        <CartesianChart.VerticalAxis>
                            <NumericalAxis />
                        </CartesianChart.VerticalAxis>
                    </CartesianChart>
```

### 数据模型 (C#)

```csharp
public ObservableCollection<int> CircleMarkerData { get; } = new() { 20, 35, 25, 40, 30 };
public ObservableCollection<int> SquareMarkerData { get; } = new() { 25, 40, 30, 45, 35 };
public ObservableCollection<int> DiamondMarkerData { get; } = new() { 30, 45, 35, 50, 40 };
public ObservableCollection<int> TriangleMarkerData { get; } = new() { 15, 30, 20, 35, 25 };
public ObservableCollection<int> PentagonMarkerData { get; } = new() { 35, 50, 40, 55, 45 };
```

## 常用属性（应用于 Series）

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ShowMarkers` | 数据点符号的全局切换。折线系列默认为 `false`；一些以点为中心的系列会覆盖为 `true`。 | 因系列而异 |
| `MarkerSize` | 标记的直径（像素）。 | 因系列而异 |
| `MarkerShape` | `Circle`、`Square`、`Rectangle`、`Diamond`、`Triangle`、`InvertedTriangle`、`Cross`、`Pentagon`、`VerticalLine` 或 `HorizontalLine`。 | `Circle` |
| `MarkerFill` | 填充标记内部的画刷。 | 系列颜色 |
| `MarkerStroke` | 标记轮廓使用的画刷。 | `null` |
| `MarkerStrokeThickness` | 标记轮廓的粗细。当为 `NaN` 时，使用系列的 `StrokeThickness`。 | `NaN` |
