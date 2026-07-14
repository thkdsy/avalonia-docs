---
id: gantt-chart
title: 甘特图
description: 用于项目管理的专业时间线图表，在水平时间轴上显示任务持续时间、开始和结束日期以及依赖关系。
doc-type: reference
tags:
  - avalonia pro
---

import chartsTimelineGantt from '/img/controls/charts/charts-timeline-gantt.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

甘特图是用于项目管理的专业时间线图表。它们通过显示任务持续时间、开始/结束日期和依赖关系来展示项目进度。

<Image light={chartsTimelineGantt} maxWidth={400} position="center" cornerRadius="true" alt="甘特图在时间轴上以水平条形式显示项目任务，包含开始日期、持续时间和依赖关系。" />

## 使用时机
- **项目规划**：识别关键路径和任务重叠。
- **资源管理**：跟踪团队成员分配到特定活动的时间。
- **发布跟踪**：可视化软件发布中的里程碑和截止日期。

## 代码示例

### XAML
```xml
<GanttChart xmlns="https://github.com/avaloniaui" Name="GanttChartSample" Title="Project Timeline" Height="300"
                     ItemsSource="{Binding GanttTasks}"
                     TaskNamePath="Name"
                     StartPath="Start"
                     EndPath="End"
                     ProgressPath="Progress"
                     BarHeight="0.72"
                     RowHeight="44"
                     BarBrush="#93C5FD"
                     ProgressBrush="#2563EB" />
```

### 数据模型 (C#)
```csharp
using System;

public record GanttTask(string Name, DateTime Start, DateTime End, double Progress);

public ObservableCollection<GanttTask> GanttTasks { get; } = new()
{
    new("规划", DateTime.Today, DateTime.Today.AddDays(5), 100),
    new("设计", DateTime.Today.AddDays(3), DateTime.Today.AddDays(10), 80),
    new("开发", DateTime.Today.AddDays(8), DateTime.Today.AddDays(20), 48),
    new("测试", DateTime.Today.AddDays(18), DateTime.Today.AddDays(25), 18),
    new("部署", DateTime.Today.AddDays(24), DateTime.Today.AddDays(28), 0)
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 项目任务的集合。 | `null` |
| `StartPath` | 任务开始时间的属性名。 | `null` |
| `EndPath` | 任务结束时间的属性名。 | `null` |
| `TaskNamePath` | 任务标签的属性名。 | `null` |
| `ProgressPath` | 任务进度值（`0` 到 `100`）的属性名。 | `null` |
| `BarHeight` | 每个任务条的高度，占行高的比例。 | `0.6` |
| `RowHeight` | 每个任务行的高度（像素）。 | `40.0` |
| `BarBrush` | 任务条使用的画刷。 | `#2196F3` |
| `ProgressBrush` | 进度覆盖层使用的画刷。 | `#1565C0` |
