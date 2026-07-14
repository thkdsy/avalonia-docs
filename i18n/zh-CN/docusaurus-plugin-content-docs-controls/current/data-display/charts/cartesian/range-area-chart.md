---
id: range-area-chart
title: 范围面积图
description: 显示连接每个品类高低值的填充区域，适用于可视化不确定性、价格带或温度范围。
doc-type: reference
tags:
  - avalonia pro
---

import chartsCartesianRangearea from '/img/controls/charts/charts-cartesian-rangearea.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

范围面积图显示连接每个品类两个值（高值和低值）的填充区域。它们可用于可视化不确定性、价格包络、温度变化和类似范围。

<Image light={chartsCartesianRangearea} maxWidth={400} position="center" cornerRadius="true" alt="范围面积图，在每日温度高低值之间显示填充带。" />

## 何时使用
- **误差范围**：显示平均值周围的置信区间或误差范围。
- **价格包络**：在单个系列中可视化每日高低价。
- **温度范围**：显示一段时间内的最低和最高温度。

## 代码示例

### XAML
```xml
<CartesianChart xmlns="https://github.com/avaloniaui" Name="RangeAreaChart" Title="范围面积图" Height="250">
    <CartesianChart.HorizontalAxis>
        <CategoryAxis />
    </CartesianChart.HorizontalAxis>
    <CartesianChart.VerticalAxis>
        <NumericalAxis />
    </CartesianChart.VerticalAxis>
    <CartesianChart.Series>
        <AreaRangeSeries Title="温度范围"
                                ItemsSource="{Binding RangeAreaData}"
                                LowPath="Low"
                                HighPath="High"
                                CategoryPath="Category"
                                Fill="#7EE91E63"
                                Stroke="DeepPink"
                                StrokeThickness="2" />
    </CartesianChart.Series>
</CartesianChart>
```

### 数据模型 (C#)
```csharp
public record CategoryRangePoint(string Category, double Low, double High);

public ObservableCollection<CategoryRangePoint> RangeAreaData { get; } = new()
{
    new("一月", 5, 15),
    new("二月", 8, 18),
    new("三月", 12, 25),
    new("四月", 18, 30),
    new("五月", 22, 35),
    new("六月", 20, 32),
    new("七月", 15, 28)
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Title` | 系列名称。 | `null` |
| `ItemsSource` | 范围数据点集合。 | `null` |
| `CategoryPath` | X 轴上的品类属性路径。 | `null` |
| `HighPath` | 最大值属性路径。 | `null` |
| `LowPath` | 最小值属性路径。 | `null` |
| `Fill` | 用于填充点之间区域的画刷。 | 取决于主题 |
| `Stroke` | 用于边界线的画刷。 | 取决于主题 |
| `ShowLines` | 是否渲染上边界线和下边界线。 | `true` |
| `FillOpacity` | 高低值之间填充带的透明度。 | `0.5` |
