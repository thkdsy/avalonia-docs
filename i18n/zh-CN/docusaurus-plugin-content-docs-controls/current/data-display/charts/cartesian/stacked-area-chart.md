---
id: stacked-area-chart
title: 堆叠面积图
description: 将多个面积系列堆叠在一起，显示多个变量如何随时间贡献累积总数。
doc-type: reference
tags:
  - avalonia pro
---

import chartsCartesianStackedarea from '/img/controls/charts/charts-cartesian-stackedarea.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

堆叠面积图将多个面积系列堆叠在一起。它们非常适合显示多个变量如何随时间贡献总数。

<Image light={chartsCartesianStackedarea} maxWidth={400} position="center" cornerRadius="true" alt="堆叠面积图，多个彩色层代表流量来源，堆叠显示累积总数。" />

## 何时使用
- **累积**：可视化多个品类在一段时间内的总和。
- **时间构成**：显示总值的构成如何随时间变化。
- **趋势比较**：比较不同层的相对增长。

## 代码示例

### XAML
```xml
<CartesianChart xmlns="https://github.com/avaloniaui" Name="StackedAreaChart" Title="堆叠面积图" Height="250" ShowLegend="True">
    <CartesianChart.HorizontalAxis>
        <CategoryAxis />
    </CartesianChart.HorizontalAxis>
    <CartesianChart.VerticalAxis>
        <NumericalAxis />
    </CartesianChart.VerticalAxis>
    <CartesianChart.Series>
        <StackedAreaSeries Title="桌面端"
                                  ItemsSource="{Binding StackedAreaDesktop}"
                                  Fill="#7E2196F3"
                                  Stroke="DodgerBlue" />
        <StackedAreaSeries Title="移动端"
                                  ItemsSource="{Binding StackedAreaMobile}"
                                  Fill="#7E4CAF50"
                                  Stroke="Green" />
        <StackedAreaSeries Title="平板端"
                                  ItemsSource="{Binding StackedAreaTablet}"
                                  Fill="#7EFF9800"
                                  Stroke="Orange" />
    </CartesianChart.Series>
</CartesianChart>
```

### 数据模型 (C#)
```csharp
public ObservableCollection<int> StackedAreaDesktop { get; } =
    new() { 500, 480, 520, 510, 490, 530, 550 };

public ObservableCollection<int> StackedAreaMobile { get; } =
    new() { 300, 350, 380, 420, 450, 480, 520 };

public ObservableCollection<int> StackedAreaTablet { get; } =
    new() { 100, 120, 130, 140, 150, 160, 170 };
```

## 常用属性（StackedAreaSeries）

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Title` | 系列名称。 | `null` |
| `ItemsSource` | 此层的数据集合。 | `null` |
| `Fill` | 用于此特定面积层的画刷。 | 自动生成 |
| `FillOpacity` | 透明度级别（0.0 到 1.0）。 | `0.7` |
| `StackGroup` | 堆叠标识符。具有相同值的系列将堆叠在一起。 | `"default"` |
