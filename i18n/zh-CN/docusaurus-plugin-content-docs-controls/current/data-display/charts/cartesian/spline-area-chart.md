---
id: spline-area-chart
title: 样条面积图
description: 在平滑样条曲线下方显示填充区域，结合样条曲线的视觉流畅性和面积图的体积强调效果。
doc-type: reference
tags:
  - avalonia pro
---

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

样条面积图在平滑插值曲线下方显示填充区域。它们结合了样条曲线的美观平滑性和面积填充的体积强调效果，使趋势在视觉上更加突出。

## 何时使用
- **平滑趋势**：显示连续数据，其中平滑插值能更好地代表潜在趋势。
- **体积强调**：通过曲线下方的填充区域突出值的幅度。
- **时间序列**：可视化随时间逐渐变化的数据，如收入或流量。

## 代码示例

### XAML
```xml
<CartesianChart xmlns="https://github.com/avaloniaui" Title="网站流量" Height="250">
    <CartesianChart.HorizontalAxis>
        <CategoryAxis />
    </CartesianChart.HorizontalAxis>
    <CartesianChart.VerticalAxis>
        <NumericalAxis />
    </CartesianChart.VerticalAxis>
    <CartesianChart.Series>
        <SplineAreaSeries Title="访客"
                                   ItemsSource="{Binding SplineAreaSeriesData}"
                                   FillOpacity="0.4"
                                   SplineTension="0.3" />
    </CartesianChart.Series>
</CartesianChart>
```

### 数据模型 (C#)
```csharp
public ObservableCollection<int> SplineAreaSeriesData { get; } = new()
{
    1200, 1400, 1100, 1600, 1800, 1500, 2000
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Title` | 图例中显示的系列名称。 | `null` |
| `ItemsSource` | 要显示的数据项集合。 | `null` |
| `CategoryPath` | 用于 X 轴的属性路径。 | `null` |
| `ValuePath` | 用于 Y 轴的属性路径。 | `null` |
| `Fill` | 用于填充区域的颜色/画刷。 | 取决于主题 |
| `Stroke` | 样条曲线轮廓的颜色。 | `Transparent` |
| `FillOpacity` | 填充区域的透明度（0.0 到 1.0）。 | `0.5` |
| `SplineTension` | 样条曲线的张力（0.0 到 1.0）。较低的值产生更尖锐的曲线，较高的值产生更平滑的曲线。 | `0.25` |

## 另请参阅

- [面积图](/controls/data-display/charts/cartesian/area-chart)
- [样条图](/controls/data-display/charts/cartesian/spline-chart)
- [折线图](/controls/data-display/charts/cartesian/line-chart)
