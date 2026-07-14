---
id: gradient-ring-chart
title: 渐变环图
description: 绘制多个同心进度环，每个环代表一个带标签的值相对于共享最大值的比例。
doc-type: reference
tags:
  - avalonia pro
---

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

渐变环图渲染多个同心进度环，每项对应一个环，以便在紧凑视图中比较分组的状态指示器。

## 何时使用

- **多指标状态**：在一个紧凑控件中显示多个进度度量。
- **能力仪表板**：比较少量类别中的完成度或健康状态。
- **环形汇总**：当值共享相同刻度时，替代一组堆叠的进度甜甜圈图。

## 代码示例

### XAML

```xml
<GradientRingChart xmlns="https://github.com/avaloniaui" Title="Release readiness"
                            Width="280"
                            Height="280"
                            ItemsSource="{Binding RingMetrics}"
                            LabelPath="Label"
                            ValuePath="Value" />
```

### 数据模型（C#）

```csharp
public record RingMetric(string Label, double Value);

public ObservableCollection<RingMetric> RingMetrics { get; } = new()
{
    new("API", 86),
    new("UI", 72),
    new("Docs", 64)
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 环数据项的集合。 | `null` |
| `LabelPath` | 项标签的路径。 | `null` |
| `ValuePath` | 数值的路径。 | `null` |
| `MaxValue` | 用于归一化的共享最大值。 | `100.0` |
| `RingThickness` | 每个环的厚度。 | `15.0` |
| `RingGap` | 相邻环之间的间隙。 | `8.0` |
| `ShowLabels` | 是否显示图例标签。 | `true` |
| `ShowValues` | 是否显示数值。 | `true` |

## 另请参阅

- [进度甜甜圈图](/controls/data-display/charts/gauges/progress-donut-chart)
- [仪表图](/controls/data-display/charts/gauges/gauge-chart)
