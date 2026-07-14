---
id: radial-bar-chart
title: 径向柱状图
description: 在极坐标系中绘制的柱状图，提供空间高效的圆形布局，用于比较类别或跟踪多个进度目标。
doc-type: reference
tags:
  - avalonia pro
---

import chartsRadialBar from '/img/controls/charts/charts-radial-bar.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

径向柱状图使用极坐标系。它们本质上是绘制在圆形网格上的柱状图，提供独特且空间高效的方式来比较类别。

<Image light={chartsRadialBar} maxWidth={400} position="center" cornerRadius="true" alt="径向柱状图，具有同心圆形柱，弧长不同，在极坐标网格上比较类别进度。" />

## 何时使用
- **循环比较**：显示具有周期性特征的数据（如一天中的小时数）。
- **仪表板信息图**：为排名类别创建紧凑的可视化摘要。
- **进度跟踪**：在合并的径向形式中可视化多个目标轨道。

## 代码示例

### XAML
```xml
<RadialBarChart xmlns="https://github.com/avaloniaui" Name="RadialBarChartSample" Title="性能指标" Height="350"
                                             ItemsSource="{Binding RadialBarData}"
                                             CategoryPath="Label" ValuePath="Value" />
```

### 数据模型 (C#)
```csharp
public record RadialPoint(string Label, double Value);

public ObservableCollection<RadialPoint> RadialBarData { get; } = new()
{
    new("速度", 85),
    new("力量", 70),
    new("敏捷", 60),
    new("防御", 75),
    new("耐力", 90)
};
```

## 通用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 类别项目的集合。 | `null` |
| `ValuePath` | 柱长度的数值属性。 | `null` |
| `CategoryPath` | 类别标签的属性名称。 | `null` |
| `InnerRadiusFactor` | 中心孔的相对大小，从 `0.0` 到 `1.0`。 | `0.2` |
| `StartAngle` | 起始角度（度）。 | `-90.0` |
| `GapAngle` | 柱之间的角度间隔。 | `2.0` |
| `ShowLabels` | 标签是否可见。 | `true` |
| `ShowValues` | 数值是否可见。 | `true` |
| `LabelFontSize` | 类别标签使用的字号。数值标签渲染小一个像素。 | `10.0` |
| `LabelForeground` | 用于类别标签的画笔。为 `null` 时，图表使用有效的标签前景色。 | `null` |
| `IsHighlightEnabled` | 启用径向柱的悬停高亮。 | `false` |
