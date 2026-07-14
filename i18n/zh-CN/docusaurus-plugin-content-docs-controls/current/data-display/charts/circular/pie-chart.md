---
id: pie-chart
title: 饼图
description: 一种圆形图表，划分为扇区以说明数值比例，在显示部分与整体关系且品类较少时最为有效。
doc-type: reference
tags:
  - avalonia pro
---

import chartsPie from '/img/controls/charts/charts-pie.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

饼图是划分为扇区以说明数值比例的圆形图表。它们在显示部分与整体关系时最为有效。

<Image light={chartsPie} maxWidth={400} position="center" cornerRadius="true" alt="饼图划分为彩色扇区，显示少数品类的比例市场份额。" />

## 何时使用
- **比例**：可视化各品类相对于总量的相对大小。
- **有限品类**：最好在 2-6 个品类时使用，以保持可读性。
- **构成**：显示总额如何在不同的段之间分配。

## 代码示例

### XAML
```xml
<PieChart xmlns="https://github.com/avaloniaui" Name="PieChartSample" IsTooltipEnabled="True" Title="市场份额" Height="300">
                        <PieChart.Series>
                            <PieSeries ItemsSource="{Binding PieChartData}" LabelPath="Value" />
                        </PieChart.Series>
                    </PieChart>
```

### 数据模型 (C#)
```csharp
public ObservableCollection<double> PieChartData { get; } = new()
{
    35, 25, 20, 15, 5
};
```

## 常用属性

### PieChart

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `InnerRadiusFactor` | 当 `PieSeries` 未设置自己的内半径时应用的中间孔大小。设置大于 `0.0` 的值可创建[环形图](/controls/data-display/charts/circular/donut-chart)。 | `0.0` |
| `Palette` | 用于段的自定义画刷集合。 | 自动生成 |

### PieSeries

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 数据项集合。 | `null` |
| `ValuePath` | 用于段值的属性路径。 | `null` |
| `LabelPath` | 用于段标签的属性路径。 | `null` |
| `RadiusFactor` | 系列的外半径因子，从 `0.0` 到 `1.0`。 | `0.9` |
| `InnerRadiusFactor` | 系列的可选内半径因子。当为 `null` 时，使用图表级别的值。 | `null` |
| `StartAngle` | 第一个扇区的起始角度（度）。 | `-90.0` |
| `ShowLabels` | 是否在扇区上显示标签。 | `true` |
| `LabelPosition` | 扇区标签的位置，`Inside`（内部）或 `Outside`（外部）。 | `Inside` |
| `SliceLabelFormat` | 扇区标签使用的格式。 | `Percentage` |
| `LabelFontSize` | 扇区标签使用的字号。 | `11.0` |
| `LabelForeground` | 扇区标签使用的画刷。 | `null` |
