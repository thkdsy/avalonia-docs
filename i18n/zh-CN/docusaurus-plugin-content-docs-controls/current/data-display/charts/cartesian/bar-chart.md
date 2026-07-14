---
id: bar-chart
title: 柱状图
description: 使用长度与数值成比例的矩形条表示数据，用于比较不同类别间的离散数量。
doc-type: reference
tags:
  - avalonia pro
---

import chartsCartesianBar from '/img/controls/charts/charts-cartesian-bar.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

柱状图使用长度与所表示数值成比例的矩形条来展示数据。

<Image light={chartsCartesianBar} maxWidth={400} position="center" cornerRadius="true" alt="柱状图，包含不同高度的垂直矩形条，比较各品类每季度的收入。" />

## 何时使用
- **比较**：比较不同类别之间的离散数量。
- **排名**：显示哪些类别具有最高或最低的值。
- **分类数据**：当数据分组为不同的、非连续的组别时。

## 代码示例

### XAML
```xml
<CartesianChart xmlns="https://github.com/avaloniaui" Name="BarChart" Title="柱状图" Height="250">
                        <CartesianChart.HorizontalAxis>
                            <CategoryAxis />
                        </CartesianChart.HorizontalAxis>
                        <CartesianChart.VerticalAxis>
                            <NumericalAxis />
                        </CartesianChart.VerticalAxis>
                        <CartesianChart.Series>
                            <BarSeries Title="销售" ItemsSource="{Binding BarSeriesData}" Fill="DodgerBlue" />
                        </CartesianChart.Series>
                    </CartesianChart>
```

### 数据模型 (C#)
```csharp
public ObservableCollection<int> BarSeriesData { get; } = new()
{
    150, 180, 165, 190, 175, 200
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Title` | 图例中显示的系列名称。 | `null` |
| `ItemsSource` | 要显示的数据项集合。 | `null` |
| `CategoryPath` | 用于 X 轴的属性路径。 | `null` |
| `ValuePath` | 用于 Y 轴的属性路径。 | `null` |
| `Fill` | 用于填充柱子的颜色/画刷。 | 取决于主题 |
| `Stroke` | 柱子的轮廓颜色。 | `Transparent` |
| `ColorByPoint` | 每个柱子是否使用自己的调色板颜色。 | `false` |
| `BarCornerRadius` | 柱子圆角半径。 | `0` |
| `BarWidth` | 每个柱子的宽度，以分类带宽度的比例表示（0.0 到 1.0）。 | `0.7` |
