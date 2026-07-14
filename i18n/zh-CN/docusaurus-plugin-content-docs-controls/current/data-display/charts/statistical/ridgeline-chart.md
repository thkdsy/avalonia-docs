---
id: ridgeline-chart
title: 山脊线图
description: 堆叠多个分布并控制重叠量，适用于比较跨组或时间的变化形状。
doc-type: reference
tags:
  - avalonia pro
---

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

山脊线图堆叠多个面积分布并带有垂直重叠，使您可以比较跨组、时间段或场景的形状变化。

## 使用时机

- **随时间变化的分布**：比较分布从一个时期到另一个时期的变化。
- **组间比较**：在一个紧凑框架中堆叠多个相关的密度类曲线。
- **形状优先分析**：强调轮廓和重叠而非精确总量。

## 代码示例

### XAML

```xml
<RidgelineChart xmlns="https://github.com/avaloniaui" Title="随时间变化的分布" Height="360">
    <RidgelineChart.Series>
        <AreaSeries Title="2023" ItemsSource="{Binding Series2023}" CategoryPath="X" ValuePath="Y" />
        <AreaSeries Title="2024" ItemsSource="{Binding Series2024}" CategoryPath="X" ValuePath="Y" />
    </RidgelineChart.Series>
</RidgelineChart>
```

### 数据模型 (C#)

```csharp
public record CurvePoint(double X, double Y);

public ObservableCollection<CurvePoint> Series2023 { get; } = new() { new(0, 4), new(1, 8), new(2, 5) };
public ObservableCollection<CurvePoint> Series2024 { get; } = new() { new(0, 3), new(1, 9), new(2, 6) };
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Series` | `AreaSeries` 分布的内容集合。 | 空集合 |
| `Overlap` | 系列之间的重叠因子。 | `0.5` |
| `SeriesHeight` | 每个系列带的目标高度。 | `50.0` |
| `CurveType` | 曲线插值类型。 | `Spline` |

## 另请参阅

- [面积图](/controls/data-display/charts/cartesian/area-chart)
- [密度图](/controls/data-display/charts/statistical/density-plot-chart)
