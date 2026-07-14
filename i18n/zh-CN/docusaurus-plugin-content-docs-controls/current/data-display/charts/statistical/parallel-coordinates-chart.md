---
id: parallel-coordinates-chart
title: 平行坐标图
description: 将多变量记录显示为跨平行轴的线条，适用于跨多个维度比较模式。
doc-type: reference
tags:
  - avalonia pro
---

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

平行坐标图将每条记录映射到一系列垂直轴上，使多维模式和异常值可以在一个视图中进行比较。

## 使用时机

- **多变量比较**：比较每个项目的多个数值维度。
- **模式检测**：发现跨指标的异常值、聚类和主要形状。
- **模型诊断**：检查记录一次在多个输入上的变化情况。

## 代码示例

### XAML

```xml
<ParallelCoordinatesChart xmlns="https://github.com/avaloniaui" Title="车辆比较"
                                           Height="320"
                                           ItemsSource="{Binding Vehicles}">
    <ParallelCoordinatesChart.Axes>
        <ParallelAxis Header="功率" ValuePath="Power" Minimum="0" Maximum="400" />
        <ParallelAxis Header="续航" ValuePath="Range" Minimum="0" Maximum="600" />
        <ParallelAxis Header="效率" ValuePath="Efficiency" Minimum="0" Maximum="100" />
    </ParallelCoordinatesChart.Axes>
</ParallelCoordinatesChart>
```

### 数据模型 (C#)

```csharp
public record VehicleStats(double Power, double Range, double Efficiency);

public ObservableCollection<VehicleStats> Vehicles { get; } = new()
{
    new(220, 480, 76),
    new(310, 420, 64),
    new(180, 520, 82)
};
```

## 常用属性（`ParallelCoordinatesChart`）

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Axes` | `ParallelAxis` 定义的内容集合。 | 空集合 |
| `ItemsSource` | 多变量记录的集合。 | `null` |
| `BrushPath` | 可选，每条线使用的 `IBrush` 或颜色字符串的路径。 | `null` |
| `LegendLabelPath` | 可选，用于图例标签的路径。 | `null` |
| `StrokeThickness` | 数据线的粗细。 | `2.0` |
| `CurveTension` | 曲线线的张力值。 | `0.0` |

## 常用属性（`ParallelAxis`）

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Header` | 轴标题。 | `null` |
| `ValuePath` | 绑定到此轴的值的路径。 | `null` |
| `Minimum` | 轴刻度的最小值。 | `0.0` |
| `Maximum` | 轴刻度的最大值。 | `100.0` |

## 另请参阅

- [雷达图](/controls/data-display/charts/radial/radar-chart)
- [三元图](/controls/data-display/charts/engineering/ternary-chart)
