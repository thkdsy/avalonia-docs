---
id: density-plot-chart
title: 密度图
description: 使用核密度估计渲染平滑的分布曲线，支持可选的曲线下填充区域。
doc-type: reference
tags:
  - avalonia pro
---

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

密度图将数值分布平滑为连续曲线，使峰值和分布范围比原始点列表更易于观察。

## 使用时机

- **分布形状**：检查集中度、偏斜度和多峰值。
- **更平滑的直方图**：用连续曲线展示与直方图相同的信息。
- **采样比较**：呈现数据集的整体形状，无需绘制每个点。

## 代码示例

### XAML

```xml
<DensityPlotChart xmlns="https://github.com/avaloniaui" Title="响应时间密度"
                                   Height="300"
                                   ItemsSource="{Binding DensityData}"
                                   ValuePath="Value" />
```

### 数据模型 (C#)

```csharp
public record Measurement(double Value);

public ObservableCollection<Measurement> DensityData { get; } = new()
{
    new(12),
    new(15),
    new(18),
    new(19),
    new(24)
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 数值样本的集合。 | `null` |
| `ValuePath` | 数值样本值的路径。 | `null` |
| `Bandwidth` | 核带宽。如果设置为 `0.0` 则自动计算。 | `0.0` |
| `FillOpacity` | 曲线下填充区域的不透明度。 | `0.3` |
| `ShowArea` | 是否填充曲线下的区域。 | `true` |
| `ShowGridLines` | 是否绘制背景网格线。 | `true` |
| `Stroke` | 密度曲线使用的画刷。 | `null` |

## 另请参阅

- [直方图](/controls/data-display/charts/cartesian/histogram-chart)
- [小提琴图](/controls/data-display/charts/statistical/violin-plot-chart)
