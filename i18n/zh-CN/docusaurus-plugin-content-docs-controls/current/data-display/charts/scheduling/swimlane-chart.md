---
id: swimlane-chart
title: 泳道图
description: 将任务或流程组织到不同的水平泳道中，跨部门、角色或团队成员显示所有权和时间顺序。
doc-type: reference
tags:
  - avalonia pro
---

import chartsTimelineSwimlane from '/img/controls/charts/charts-timeline-swimlane.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

泳道图将任务或流程组织到不同的水平或垂直"泳道"中。它们有助于可视化跨不同部门或角色的工作流。

<Image light={chartsTimelineSwimlane} maxWidth={400} position="center" cornerRadius="true" alt="泳道图将任务按部门或角色组织到水平泳道中，显示所有权和时间重叠。" />

## 使用时机

- **流程映射**：展示请求如何在销售、工程和支持部门之间流转。
- **项目排期**：可视化团队成员之间的任务所有权。
- **跨职能流程**：明确复杂业务流程中的职责。

## 代码示例

### XAML

```xml
<SwimlaneChart xmlns="https://github.com/avaloniaui" Name="SwimlaneSample"
                                            Title="Process Flow"
                                            Height="350"
                                            ItemsSource="{Binding SwimlaneTasks}"
                                            LanePath="Lane"
                                            TaskNamePath="Task"
                                            StartPath="Start"
                                            EndPath="End"
                                            BrushPath="Brush"
                                            LaneHeight="90"
                                            TaskHeight="34"
                                            TaskSpacing="8"
                                            TaskCornerRadius="8"
                                            LaneSeparatorBrush="{DynamicResource DemoSwimlaneLaneSeparatorBrush}"
                                            LaneBackgroundBrush="{DynamicResource DemoSwimlaneLaneBackgroundBrush}" />
```

### 数据模型 (C#)

```csharp
using Avalonia.Media;

public record SwimlaneTask(string Lane, string Task, double Start, double End, IBrush? Brush = null);

public ObservableCollection<SwimlaneTask> SwimlaneTasks { get; } = new()
{
    new("设计", "线框图", 0, 3, Brushes.DodgerBlue),
    new("设计", "高保真图", 2, 5, Brushes.SteelBlue),
    new("开发", "前端", 4, 10, Brushes.SeaGreen),
    new("开发", "后端", 3, 9, Brushes.MediumSeaGreen),
    new("测试", "单元测试", 6, 10, Brushes.Orange),
    new("测试", "集成测试", 9, 12, Brushes.DarkOrange),
    new("部署", "预发布", 11, 13, Brushes.MediumPurple),
    new("部署", "生产环境", 13, 14, Brushes.Purple)
};
```

`StartPath` 和 `EndPath` 期望数值。使用您自己的比例，例如天、小时、冲刺点数或序列位置。同一泳道中重叠的任务会使用 `TaskSpacing` 堆叠成行。

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 任务/项目的集合。 | `null` |
| `LanePath` | 决定任务属于哪个泳道。 | `null` |
| `TaskNamePath` | 单个任务块的标签。 | `null` |
| `StartPath` | 任务跨度数值开始值的属性名。 | `null` |
| `EndPath` | 任务跨度数值结束值的属性名。 | `null` |
| `BrushPath` | 用于为每个任务绑定 `IBrush` 或颜色字符串的属性名。 | `null` |
| `LaneHeight` | 每个泳道的高度。 | `80.0` |
| `TaskHeight` | 每个任务条的高度。 | `30.0` |
| `TaskSpacing` | 同一泳道中堆叠任务时的间距。 | `5.0` |
| `LaneSeparatorBrush` | 泳道分隔线使用的画刷。 | `null` |
| `LaneBackgroundBrush` | 交替泳道背景使用的画刷。 | `null` |
| `ShowTaskLabels` | 是否在任务条内显示标签。 | `true` |
| `TaskCornerRadius` | 任务条的圆角半径。 | `4.0` |

## 另请参阅

- [甘特图](/controls/data-display/charts/scheduling/gantt-chart)
- [事件时间线图](/controls/data-display/charts/scheduling/timeline-chart)
