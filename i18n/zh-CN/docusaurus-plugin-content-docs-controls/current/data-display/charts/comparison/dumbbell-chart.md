---
id: dumbbell-chart
title: 哑铃图
description: 使用连线和标记连接每个品类的两个值，适用于前后对比或低到高比较。
doc-type: reference
tags:
  - avalonia pro
---

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

哑铃图连接每个品类的两个标记，使您可以一目了然地比较低值和高值之间的差距。

## 何时使用

- **前后对比**：比较同一指标在两个时间点的值。
- **范围审查**：显示每个品类两个相关值之间的跨度。
- **目标与实际**：将测量值与基准配对。

## 代码示例

### XAML

```xml
<DumbbellChart xmlns="https://github.com/avaloniaui" Title="计划与实际"
                        Height="300"
                        ItemsSource="{Binding PlannedVsActual}"
                        LabelPath="Label"
                        LowValuePath="Planned"
                        HighValuePath="Actual" />
```

### 数据模型 (C#)

```csharp
public record RangeComparison(string Label, double Planned, double Actual);

public ObservableCollection<RangeComparison> PlannedVsActual { get; } = new()
{
    new("团队 A", 42, 55),
    new("团队 B", 38, 41),
    new("团队 C", 50, 62)
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 比较项的集合。 | `null` |
| `LowValuePath` | 较低或第一个值的路径。 | `null` |
| `HighValuePath` | 较高或第二个值的路径。 | `null` |
| `LabelPath` | 品类标签的路径。 | `null` |
| `LowBrush` | 用于第一个标记的画刷。 | `#2196F3` |
| `HighBrush` | 用于第二个标记的画刷。 | `#4CAF50` |
| `ConnectorBrush` | 用于连接线的画刷。 | `#9E9E9E` |
| `MarkerSize` | 标记大小（像素）。 | `12.0` |
| `ConnectorThickness` | 连接线的粗细。 | `3.0` |
| `Orientation` | 布局方向，`Horizontal`（水平）或 `Vertical`（垂直）。 | `Horizontal` |
| `IsHighlightEnabled` | 启用哑铃项的悬停高亮。 | `false` |

## 另请参阅

- [斜率图](/controls/data-display/charts/analytics/slope-chart)
- [发散柱状图](/controls/data-display/charts/comparison/diverging-bar-chart)
