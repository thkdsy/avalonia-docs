---
id: calendar-heatmap-chart
title: 日历热力图
description: 以日历网格形式展示每日活动强度，适用于贡献历史、使用追踪和习惯连续记录。
doc-type: reference
tags:
  - avalonia pro
---

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

日历热力图以逐周网格的形式呈现每日数值，类似于贡献图和活动追踪器。

## 使用场景

- **活动追踪**：按天展示提交、登录、锻炼或交易记录。
- **季节性回顾**：发现一年中的淡季和活跃期。
- **留存视图**：无需完整时间序列图表，即可突出连续记录和断档。

## 代码示例

### XAML

```xml
<CalendarHeatmapChart xmlns="https://github.com/avaloniaui" Title="Repository activity"
                               Height="180"
                               ItemsSource="{Binding DailyActivity}"
                               DatePath="Date"
                               ValuePath="Count"
                               WeeksToShow="26" />
```

### 数据模型 (C#)

```csharp
using System;

public record DailyCount(DateTime Date, double Count);

public ObservableCollection<DailyCount> DailyActivity { get; } = new()
{
    new(DateTime.Today.AddDays(-3), 5),
    new(DateTime.Today.AddDays(-2), 2),
    new(DateTime.Today.AddDays(-1), 7)
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 每日活动项的集合。 | `null` |
| `DatePath` | 每个项中日期值的路径。 | `null` |
| `ValuePath` | 数值强度值的路径。 | `null` |
| `CellSize` | 每个日期单元格的大小（像素）。 | `12.0` |
| `CellGap` | 日期单元格之间的间距。 | `2.0` |
| `EmptyCellBrush` | 用于无值日期的画笔。 | `null` |
| `LowBrush` | 用于低强度值的画笔。 | `null` |
| `MediumBrush` | 用于中等强度值的画笔。 | `null` |
| `HighBrush` | 用于高强度值的画笔。 | `null` |
| `MaxBrush` | 用于最高强度值的画笔。 | `null` |
| `ShowMonthLabels` | 是否在网格上方显示月份标签。 | `true` |
| `ShowDayLabels` | 是否显示星期标签。 | `true` |
| `LabelFontSize` | 月份标签、星期标签和图例文本的字体大小。 | `10.0` |
| `LabelForeground` | 用于日历和图例标签的画笔。当为 `null` 时，图表使用有效的标签前景色。 | `null` |
| `WeeksToShow` | 显示的周数。 | `52` |
| `IsHighlightEnabled` | 启用日历单元格的悬停高亮。 | `false` |
| `ReferenceDate` | 用作日历锚点的可选结束日期。 | `null` |

## 另请参阅

- [热力图](/controls/data-display/charts/analytics/heatmap-chart)
- [时间线图](/controls/data-display/charts/scheduling/timeline-chart)
