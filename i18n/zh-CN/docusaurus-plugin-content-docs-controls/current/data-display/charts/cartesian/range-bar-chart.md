---
id: range-bar-chart
title: 范围柱状图
description: 显示跨越每个品类低到高范围的浮动柱条，非常适合可视化温度范围、价格带或任务持续时间。
doc-type: reference
tags:
  - avalonia pro
---

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

范围柱状图显示浮动的矩形柱条，每个品类从低值跨越到高值。它们适用于展示数据范围、区间或带，而不是单个值。

## 何时使用
- **温度范围**：显示跨天或月的每日最低/最高温度带。
- **价格带**：可视化金融数据的价格范围或置信区间。
- **任务持续时间**：表示调度或时间线数据的起始到结束间隔。

## 代码示例

### XAML
```xml
<CartesianChart xmlns="https://github.com/avaloniaui" Title="周温度范围" Height="250">
    <CartesianChart.HorizontalAxis>
        <CategoryAxis />
    </CartesianChart.HorizontalAxis>
    <CartesianChart.VerticalAxis>
        <NumericalAxis />
    </CartesianChart.VerticalAxis>
    <CartesianChart.Series>
        <RangeBarSeries Title="温度"
                                 ItemsSource="{Binding TemperatureData}"
                                 CategoryPath="Day"
                                 LowPath="Min"
                                 HighPath="Max"
                                 BarWidth="0.6"
                                 BarCornerRadius="4" />
    </CartesianChart.Series>
</CartesianChart>
```

### 数据模型 (C#)
```csharp
public record TemperatureRange(string Day, double Min, double Max);

public ObservableCollection<TemperatureRange> TemperatureData { get; } = new()
{
    new("周一", 12, 22),
    new("周二", 14, 25),
    new("周三", 10, 18),
    new("周四", 16, 28),
    new("周五", 13, 21)
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Title` | 图例中显示的系列名称。 | `null` |
| `ItemsSource` | 要显示的数据项集合。 | `null` |
| `CategoryPath` | 用于 X 轴的属性路径。 | `null` |
| `LowPath` | 低（最小值）值的属性路径。 | `null` |
| `HighPath` | 高（最大值）值的属性路径。 | `null` |
| `Fill` | 用于填充柱子的颜色/画刷。 | 取决于主题 |
| `Stroke` | 柱子的轮廓颜色。 | `Transparent` |
| `BarWidth` | 每个柱子的宽度，以分类带宽度的比例表示（0.0 到 1.0）。 | `0.7` |
| `BarCornerRadius` | 柱子圆角半径。 | `2` |

## 另请参阅

- [柱状图](/controls/data-display/charts/cartesian/bar-chart)
- [差异图](/controls/data-display/charts/cartesian/variance-chart)
- [组合图](/controls/data-display/charts/cartesian/combo-chart)
