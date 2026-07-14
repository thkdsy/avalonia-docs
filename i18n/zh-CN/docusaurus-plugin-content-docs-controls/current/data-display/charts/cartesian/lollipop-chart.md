---
id: lollipop-chart
title: 棒棒糖图
description: 将数据显示为带有细茎连接到坐标轴的圆点，结合了点图的精确性和柱状图的视觉锚定。
doc-type: reference
tags:
  - avalonia pro
---

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

棒棒糖图将数据点显示为带有细茎延伸到基线的圆点。它们是柱状图的轻量级替代方案，减少视觉杂乱，同时保持清晰的值传达。

## 何时使用
- **轻量比较**：当柱状图感觉过于厚重，希望获得更简洁的外观时。
- **多品类**：在并排比较多个品类时，降低墨水与数据比率。
- **演示**：创建视觉上吸引人的图表，强调数据点的值。

## 代码示例

### XAML
```xml
<CartesianChart xmlns="https://github.com/avaloniaui" Title="月度销售" Height="250">
    <CartesianChart.HorizontalAxis>
        <CategoryAxis />
    </CartesianChart.HorizontalAxis>
    <CartesianChart.VerticalAxis>
        <NumericalAxis />
    </CartesianChart.VerticalAxis>
    <CartesianChart.Series>
        <LollipopSeries Title="销售"
                                 ItemsSource="{Binding MonthlySales}"
                                 CategoryPath="Month"
                                 ValuePath="Amount"
                                 StemThickness="2"
                                 Orientation="Vertical" />
    </CartesianChart.Series>
</CartesianChart>
```

### 数据模型 (C#)
```csharp
public record SalesItem(string Month, double Amount);

public ObservableCollection<SalesItem> MonthlySales { get; } = new()
{
    new("一月", 320),
    new("二月", 450),
    new("三月", 280),
    new("四月", 510),
    new("五月", 390)
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Title` | 图例中显示的系列名称。 | `null` |
| `ItemsSource` | 要显示的数据项集合。 | `null` |
| `CategoryPath` | 用于 X 轴的属性路径。 | `null` |
| `ValuePath` | 用于 Y 轴的属性路径。 | `null` |
| `Fill` | 用于填充圆点的颜色/画刷。 | 取决于主题 |
| `Stroke` | 圆点的轮廓颜色。 | `Transparent` |
| `StemThickness` | 茎线的粗细。 | `2` |
| `StemBrush` | 茎线的画刷。 | `null`（未设置时默认为与 `Fill` 相同的颜色。） |
| `Orientation` | 茎的方向，`Vertical`（垂直）或 `Horizontal`（水平）。 | `Vertical` |

## 另请参阅

- [点图](/controls/data-display/charts/cartesian/dot-plot-chart)
- [柱状图](/controls/data-display/charts/cartesian/bar-chart)
- [散点图](/controls/data-display/charts/cartesian/scatter-chart)
