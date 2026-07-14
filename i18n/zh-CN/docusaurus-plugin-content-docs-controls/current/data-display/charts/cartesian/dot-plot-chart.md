---
id: dot-plot-chart
title: 散点图（点图）
description: 将数据点显示为沿坐标轴分布的简单圆点，适用于比较各品类的值或展示频率分布。
doc-type: reference
tags:
  - avalonia pro
---

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

点图将单个数据点显示为简单的圆点，因此可以在没有柱子视觉负担的情况下比较各品类的值。

## 何时使用
- **值比较**：在多个类别之间比较单个值，避免柱状图的杂乱。
- **频率分布**：显示数据集中特定值出现的频率。
- **小数据集**：当每个数据点的精确定位比整体形状更重要时。

## 代码示例

### XAML
```xml
<CartesianChart xmlns="https://github.com/avaloniaui" Title="员工评分" Height="250">
    <CartesianChart.HorizontalAxis>
        <CategoryAxis />
    </CartesianChart.HorizontalAxis>
    <CartesianChart.VerticalAxis>
        <NumericalAxis />
    </CartesianChart.VerticalAxis>
    <CartesianChart.Series>
        <DotPlotSeries Title="评分"
                                ItemsSource="{Binding RatingData}"
                                CategoryPath="Department"
                                ValuePath="Score"
                                DotSize="12"
                                ShowConnectorLines="True" />
    </CartesianChart.Series>
</CartesianChart>
```

### 数据模型 (C#)
```csharp
public record RatingItem(string Department, double Score);

public ObservableCollection<RatingItem> RatingData { get; } = new()
{
    new("工程部", 4.5),
    new("市场部", 3.8),
    new("销售部", 4.1),
    new("支持部", 3.5),
    new("设计部", 4.3)
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
| `DotSize` | 每个圆点的大小（像素）。 | `10` |
| `ShowConnectorLines` | 是否显示连接各点的连线。 | `false` |
| `ConnectorThickness` | 连接线的粗细。 | `1` |

## 另请参阅

- [散点图](/controls/data-display/charts/cartesian/scatter-chart)
- [棒棒糖图](/controls/data-display/charts/cartesian/lollipop-chart)
- [柱状图](/controls/data-display/charts/cartesian/bar-chart)
