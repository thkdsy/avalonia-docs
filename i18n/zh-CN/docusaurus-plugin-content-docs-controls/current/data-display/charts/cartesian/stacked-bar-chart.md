---
id: stacked-bar-chart
title: 堆叠柱状图
description: 在单个柱条中堆叠多个数据系列，以比较各品类的总值和内部组件分布。
doc-type: reference
tags:
  - avalonia pro
---

import chartsCartesianStackedbar from '/img/controls/charts/charts-cartesian-stackedbar.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

堆叠柱状图将多个系列的数据库叠在一起，允许同时比较总值和各个组成部分。

<Image light={chartsCartesianStackedbar} maxWidth={400} position="center" cornerRadius="true" alt="堆叠柱状图，每个柱条中包含彩色分段，显示每季度各地区的销售贡献。" />

## 何时使用
- **部分与整体**：可视化多个小部分如何组成更大的总品类。
- **品类比较**：比较不同组别的总数，同时查看内部分布。
- **空间优化**：显示多个数据系列，无需为每个系列单独设置柱条。

## 代码示例

### XAML
```xml
<CartesianChart xmlns="https://github.com/avaloniaui" Name="StackedBarChart" Title="堆叠柱状图" Height="250" ShowLegend="True">
                        <CartesianChart.HorizontalAxis>
                            <CategoryAxis />
                        </CartesianChart.HorizontalAxis>
                        <CartesianChart.VerticalAxis>
                            <NumericalAxis />
                        </CartesianChart.VerticalAxis>
                        <CartesianChart.Series>
                            <StackedBarSeries Title="产品 A" ItemsSource="{Binding StackedBarProductA}" Fill="DodgerBlue" />
                            <StackedBarSeries Title="产品 B" ItemsSource="{Binding StackedBarProductB}" Fill="Orange" />
                            <StackedBarSeries Title="产品 C" ItemsSource="{Binding StackedBarProductC}" Fill="Green" />
                        </CartesianChart.Series>
                    </CartesianChart>
```

### 数据模型 (C#)
```csharp
public ObservableCollection<int> StackedBarProductA { get; } = new()
{
    40, 55, 50, 60, 45, 70
};

public ObservableCollection<int> StackedBarProductB { get; } = new()
{
    30, 35, 40, 45, 35, 50
};

public ObservableCollection<int> StackedBarProductC { get; } = new()
{
    20, 25, 30, 25, 30, 35
};
```

## 常用属性（StackedBarSeries）

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Title` | 图例中显示的名称。 | `null` |
| `ItemsSource` | 此堆叠部分的数据集合。 | `null` |
| `CategoryPath` | 每个柱条的共享品类值路径。 | `null` |
| `ValuePath` | 此系列的数值路径。 | `null` |
| `Fill` | 此系列段的背景颜色。 | 自动生成 |
| `StackGroup` | 用于将相关系列堆叠在一起的标识符。 | `"default"` |
| `BarWidth` | 每个柱条的宽度，以可用槽位比例表示。 | `0.7` |
| `BarCornerRadius` | 柱条段的圆角半径。 | `0` |
