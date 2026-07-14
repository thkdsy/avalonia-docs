---
id: spiral-timeline-chart
title: 螺旋时间线
description: 将时间序列包裹成螺旋状，在同一可视化中同时揭示长期趋势和重复的周期模式。
doc-type: reference
tags:
  - avalonia pro
---

import chartsTimelineSpiral from '/img/controls/charts/charts-timeline-spiral.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

螺旋时间线将同时具有强序列分量和周期模式的数据可视化。通过将时间线包裹成螺旋状，长期趋势和短期重复变得可见。

<Image light={chartsTimelineSpiral} maxWidth={400} position="center" cornerRadius="true" alt="螺旋时间线图表将时间数据包裹成向外扩展的螺旋，同时显示长期趋势和周期模式。" />

## 使用时机

- **长期周期数据**：可视化跨越数十年的年度气候变化。
- **系统日志**：检测跨周或跨月的服务器活动模式。
- **生物节律**：显示随时间变化的睡眠模式或活动周期。

## 代码示例

### XAML

```xml
<SpiralTimeline xmlns="https://github.com/avaloniaui" Name="SpiralTimelineSample"
                                             Title="Annual Events"
                                             Height="400"
                                             ItemsSource="{Binding SpiralEvents}"
                                             ValuePath="Value"
                                             LabelPath="Event" />
```

### 数据模型 (C#)

```csharp
using System;

public record SpiralEvent(DateTime Date, string Event, double Value);

public ObservableCollection<SpiralEvent> SpiralEvents { get; } = new()
{
    new(new DateTime(2024, 1, 1), "新年", 1.0),
    new(new DateTime(2024, 3, 20), "春季", 2.0),
    new(new DateTime(2024, 6, 21), "夏季", 3.0),
    new(new DateTime(2024, 9, 22), "秋季", 2.0),
    new(new DateTime(2024, 12, 21), "冬季", 1.0)
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 螺旋上数据点的集合。 | `null` |
| `DatePath` | 时间属性的路径。 | `null` |
| `ValuePath` | 决定点大小/颜色的数值属性。 | `null` |
| `LabelPath` | 每个点文本标签的路径。 | `null` |
| `Turns` | 螺旋的圈数。 | `3.0` |
| `InnerRadius` | 螺旋的内半径。 | `30.0` |
| `MarkerSize` | 数据点标记的大小。 | `8.0` |
