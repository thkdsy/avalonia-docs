---
id: donut-chart
title: 环形图
description: 饼图的一种变体，中心为空白，通常用于在中心显示总值或标签，以提高可读性。
doc-type: reference
tags:
  - avalonia pro
---

import chartsPieDonut from '/img/controls/charts/charts-pie-donut.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

环形图是[饼图](/controls/data-display/charts/circular/pie-chart)的一种变体，使用非零的 `InnerRadiusFactor`。这种设计让您可以将中心保留用于汇总值或标签。

<Image light={chartsPieDonut} maxWidth={400} position="center" cornerRadius="true" alt="环形图，中心为空白圆孔，显示按来源划分的收入分布比例段。" />

## 何时使用
- **比例比较**：与饼图类似，但中心有更多空间用于标签或总计。
- **汇总视图**：中心空间可用于显示总和或关键指标。
- **极简仪表板**：对于品类较少的简单部分与整体可视化非常有效。

## 代码示例

### XAML
```xml
<PieChart xmlns="https://github.com/avaloniaui" Name="DonutChartSample" IsTooltipEnabled="True" Title="收入分布" Height="300" InnerRadiusFactor="0.6">
                         <PieChart.Series>
                            <PieSeries ItemsSource="{Binding DonutChartData}" LabelPath="Value" />
                        </PieChart.Series>
                    </PieChart>
```

### 数据模型 (C#)
```csharp
public ObservableCollection<double> DonutChartData { get; } = new()
{
    40, 30, 20, 10
};
```

没有专门的 `DonutChart` 控件。使用 `PieChart` 并将 `InnerRadiusFactor` 设置为大于 `0.0` 的值。

## 常用属性

### PieChart

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `InnerRadiusFactor` | 中心孔的大小，从 `0.0` 到 `1.0`。 | `0.0` |
| `Title` | 环形图上方显示的图表标题。 | `null` |
| `Palette` | 用于扇区的自定义画刷集合。 | 取决于主题 |

### PieSeries

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 数据扇区的集合。 | `null` |
| `LabelPath` | 在扇区上或附近显示的文本路径。 | `null` |
| `ValuePath` | 用于扇区大小的数值路径。 | `null` |
| `RadiusFactor` | 系列的外半径因子，从 `0.0` 到 `1.0`。 | `0.9` |
| `InnerRadiusFactor` | 系列的可选内半径因子。当为 `null` 时，使用图表级别的值。 | `null` |
| `StartAngle` | 第一个扇区的起始角度（度）。 | `-90.0` |
| `ShowLabels` | 是否在扇区上显示标签。 | `true` |
| `LabelPosition` | 扇区标签的位置，`Inside`（内部）或 `Outside`（外部）。 | `Inside` |
| `SliceLabelFormat` | 扇区标签使用的格式。 | `Percentage` |
| `LabelFontSize` | 扇区标签使用的字号。 | `11.0` |
| `LabelForeground` | 扇区标签使用的画刷。 | `null` |
