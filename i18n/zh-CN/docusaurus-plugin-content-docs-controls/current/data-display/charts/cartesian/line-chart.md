---
id: line-chart
title: 折线图
description: 在 X 轴和 Y 轴上用直线段连接数据点，非常适合显示随时间或跨品类变化的趋势。
doc-type: reference
tags:
  - avalonia pro
---

import chartsCartesianLines from '/img/controls/charts/charts-cartesian-lines.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

折线图使用 X 轴和 Y 轴来可视化由直线段连接的数据点。它们非常适合显示随时间或品类变化的趋势。

<Image light={chartsCartesianLines} maxWidth={400} position="center" cornerRadius="true" alt="折线图用直线段连接数据点，显示月度销售趋势。" />

## 何时使用
- **时间序列**：可视化数据在连续时间间隔内的变化。
- **趋势分析**：识别上升、下降或波动模式。
- **多系列**：使用多条折线比较不同品类的趋势。

## 代码示例

### XAML
```xml
<CartesianChart xmlns="https://github.com/avaloniaui" Name="LineChart" Title="折线图" Height="250" ShowLegend="True">
                        <CartesianChart.HorizontalAxis>
                            <CategoryAxis />
                        </CartesianChart.HorizontalAxis>
                        <CartesianChart.VerticalAxis>
                            <NumericalAxis />
                        </CartesianChart.VerticalAxis>
                        <CartesianChart.Series>
                            <LineSeries Title="2023年" ItemsSource="{Binding LineSeries2023}" Stroke="DodgerBlue" StrokeThickness="2" MarkerSize="6" MarkerFill="DodgerBlue"  ShowMarkers="True"/>
                            <LineSeries Title="2024年" ItemsSource="{Binding LineSeries2024}" Stroke="Orange" StrokeThickness="2" MarkerSize="6" MarkerFill="Orange"  ShowMarkers="True"/>
                        </CartesianChart.Series>
                    </CartesianChart>
```

### 数据模型 (C#)
```csharp
public ObservableCollection<int> LineSeries2023 { get; } = new()
{
    45, 52, 48, 60, 55, 70, 65, 75, 68, 80, 72, 85
};

public ObservableCollection<int> LineSeries2024 { get; } = new()
{
    50, 58, 55, 68, 62, 78, 72, 82, 75, 88, 80, 92
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Title` | 图例中显示的系列名称。 | `null` |
| `ItemsSource` | 要显示的数据项集合。 | `null` |
| `CategoryPath` | 用于 X 轴（类别）的属性路径。 | `null` |
| `ValuePath` | 用于 Y 轴（值）的属性路径。 | `null` |
| `Stroke` | 线条的颜色。 | 取决于主题 |
| `StrokeThickness` | 线条的粗细。 | `2` |
| `ShowMarkers` | 是否显示各个数据点的标记。标记样式仅在此为 `true` 时可见。 | `false` |
| `MarkerSize` | 标记的大小（像素）。 | `8` |
| `MarkerShape` | 标记的形状，例如 `Circle` 或 `Square`。 | `Circle` |
| `MarkerFill` | 用于填充标记的画刷。当为 `null` 时，使用系列描边颜色。 | `null` |
| `MarkerStroke` | 用于标记轮廓的画刷。 | `null` |
| `MarkerStrokeThickness` | 标记轮廓的粗细。当为 `NaN` 时，使用系列的 `StrokeThickness`。 | `NaN` |
