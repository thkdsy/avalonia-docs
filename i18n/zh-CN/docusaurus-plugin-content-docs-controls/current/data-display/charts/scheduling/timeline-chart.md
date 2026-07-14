---
id: timeline-chart
title: 事件时间线图
description: 按时间顺序沿时间轴可视化一系列事件，清晰展示历史或计划中事件的发生。
doc-type: reference
tags:
  - avalonia pro
---

import chartsTimelineHorizontal from '/img/controls/charts/charts-timeline-event.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

事件时间线图按时间顺序可视化一系列事件。它们沿固定时间轴清晰展示历史或计划中的事件。

<Image light={chartsTimelineHorizontal} maxWidth={400} position="center" cornerRadius="true" alt="事件时间线图沿水平时间轴以带标签的标记形式显示按时间顺序排列的里程碑。" />

## 使用时机

- **历史记录**：可视化里程碑、产品发布或生活事件。
- **审计跟踪**：按顺序显示系统日志或用户活动。
- **垂直时间线**：适合移动友好或基于列的布局。

## 代码示例

### XAML

```xml
<EventTimelineChart xmlns="https://github.com/avaloniaui" Name="EventTimelineSample"
                                                 Title="Product Launches"
                                                 Height="300"
                                                 ItemsSource="{Binding TimelineEvents}"
                                                 DatePath="Date"
                                                 LabelPath="Event" />
```

### 数据模型 (C#)

```csharp
using System;

public record TimelineEvent(DateTime Date, string Event);

public ObservableCollection<TimelineEvent> TimelineEvents { get; } = new()
{
    new(new DateTime(2024, 1, 15), "v1.0 发布"),
    new(new DateTime(2024, 4, 10), "v2.0 Beta"),
    new(new DateTime(2024, 7, 20), "v2.0 发布"),
    new(new DateTime(2024, 10, 5), "v3.0 预览"),
    new(new DateTime(2024, 12, 1), "v3.0 发布")
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 事件的集合。 | `null` |
| `DatePath` | `DateTime` 属性的路径。 | `null` |
| `LabelPath` | 事件描述的路径。 | `null` |
| `DescriptionPath` | 可选的长文本支持信息路径。 | `null` |
| `BrushPath` | 可选，每个项目的画刷或颜色值的路径。 | `null` |
| `MarkerSize` | 事件标记的大小。 | `12.0` |
| `StrokeThickness` | 主时间线的粗细。 | `2.0` |
| `Orientation` | 图表的方向，`Horizontal` 或 `Vertical`。 | `Horizontal` |
