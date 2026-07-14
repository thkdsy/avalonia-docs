---
id: slope-chart
title: 斜率图
description: 比较多个实体在恰好两个时间点或类别之间的值，突出显示哪些实体增加、减少或保持不变。
doc-type: reference
tags:
  - avalonia pro
---

import chartsAnalyticsSlope from '/img/controls/charts/charts-statistical-slope.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

斜率图比较多个实体在两个时间点（或两个类别）之间的变化。它们非常适合可视化两种状态之间排名或数值的变化。

<Image light={chartsAnalyticsSlope} maxWidth={400} position="center" cornerRadius="true" alt="斜率图，比较两个时间点之间的实体数值，使用带标签的线条显示哪些增加或减少了。" />

## 使用场景
- **前后对比分析**：展示某个政策或事件对不同群体的影响。
- **排名变化**：可视化产品在两个季度之间的热度变化。
- **两种状态对比**：突出显示哪些实体改善了，哪些恶化了。

## 代码示例

### XAML
```xml
<SlopeChart xmlns="https://github.com/avaloniaui" Name="SlopeChartSample"
                                         Title="Before vs After"
                                         Height="300"
                                         LabelPath="Label"
                                         StartValuePath="Before"
                                         EndValuePath="After"
                                         IsCurved="True"
                                         ShowGridLines="True"
                                         ShowXAxis="True"
                                         ItemsSource="{Binding SlopeData}" />
```

### 数据模型 (C#)
```csharp
public record SlopeItem(string Label, double Before, double After);

public ObservableCollection<SlopeItem> SlopeData { get; } = new()
{
    new("Sales", 100.0, 150.0),
    new("Cost", 80.0, 70.0),
    new("Profit", 20.0, 80.0)
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 要比较的项的集合。 | `null` |
| `LabelPath` | 实体名称的路径。 | `null` |
| `StartValuePath` | 左侧显示的第一个值的路径。 | `null` |
| `EndValuePath` | 右侧显示的第二个值的路径。 | `null` |
| `StartLabel` | 左侧显示的标签。 | `"Before"` |
| `EndLabel` | 右侧显示的标签。 | `"After"` |
| `StrokeThickness` | 连接线的宽度。 | `2.0` |
| `MarkerSize` | 每条线起点和终点标记的大小。 | `8.0` |
| `ShowLabels` | 切换起点和终点数值标签的显示。 | `true` |
| `IsCurved` | 使用曲线代替直线连接。 | `false` |
| `ShowGridLines` | 是否为起点和终点位置绘制引导线。 | `false` |
| `ShowXAxis` | 是否绘制底部轴并将侧边标签移动到图表下方。 | `false` |
| `IsHighlightEnabled` | 启用斜率线的悬停高亮。 | `false` |
