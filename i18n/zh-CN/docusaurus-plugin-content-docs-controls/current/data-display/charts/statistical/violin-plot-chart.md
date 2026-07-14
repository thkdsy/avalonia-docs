---
id: violin-plot-chart
title: 小提琴图
description: 将箱线图与核密度估计相结合，同时显示跨分类的统计摘要和概率分布形状。
doc-type: reference
tags:
  - avalonia pro
---

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

小提琴图将箱线图与核密度图相结合，同时显示数据的统计摘要和不同值处的概率密度。

## 使用时机

- **深度分布**：当您需要查看数据点最频繁（密度）的位置时。
- **比较**：比较多个组的范围（箱线图）和形状（密度）。
- **多模态数据**：识别箱线图可能隐藏的具有多个峰值（模态）的数据。

## 代码示例

### XAML

```xml
<ViolinPlotChart xmlns="https://github.com/avaloniaui" Name="ViolinPlotSample"
                           Title="响应时间"
                           Height="300"
                           CategoryPath="Group"
                           ValuesPath="DataPoints"
                           ItemsSource="{Binding ViolinSeries}" />
```

### 数据模型 (C#)

```csharp
public record ViolinGroup(string Group, ObservableCollection<double> DataPoints);

public ObservableCollection<ViolinGroup> ViolinSeries { get; } = new()
{
    new("后端", new() { 12, 15, 12, 18, 25, 30, 12, 14 }),
    new("前端", new() { 50, 55, 60, 50, 45, 80, 50, 52 })
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 数据组的集合。 | `null` |
| `ValuesPath` | 每个分类值集合的路径。支持的类型有 `IEnumerable<double>`、`IEnumerable<int>` 和 `double[]`。 | `null` |
| `CategoryPath` | 分类名称的路径。 | `null` |
| `ShowMedian` | 是否在嵌入的箱线图内显示中位线。仅在启用 `ShowBoxPlot` 且至少有五个值时适用。 | `true` |
| `ViolinWidth` | 每个小提琴主体的宽度因子。 | `0.8` |
| `Fill` | 小提琴主体使用的画刷。 | `null` |
| `Stroke` | 小提琴轮廓使用的画刷。 | `null` |
| `StrokeThickness` | 小提琴轮廓的粗细。 | `1.5` |
| `ShowBoxPlot` | 是否显示内部箱线图。仅当分类至少有五个值时才绘制四分位数叠加层。 | `true` |

:::note
当 `Fill` 或 `Stroke` 为 `null` 时，图表会回退使用该分类的调色板画刷。`Fill` 回退以降低的不透明度绘制。
:::
