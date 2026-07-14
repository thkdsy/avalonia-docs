---
id: polar-chart
title: 极坐标图
description: 在极坐标系中绘制任意角度和半径值，适用于螺旋线、玫瑰曲线和方向数据。
doc-type: reference
tags:
  - avalonia pro
---

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

`PolarChart` 承载一个或多个 `PolarLineSeries`，并使用角度和半径而不是笛卡尔轴来映射每个点。

## 何时使用

- **数学曲线**：绘制螺旋线、玫瑰曲线、心脏线及类似函数。
- **方向测量**：在整个角度范围内绘制值。
- **径向分析**：当固定雷达辐条过于受限时，使用自由形式的角度。

## 代码示例

### XAML

```xml
<PolarChart xmlns="https://github.com/avaloniaui" Title="阿基米德螺旋" Height="300">
    <PolarChart.Series>
        <PolarLineSeries ItemsSource="{Binding SpiralData}"
                                  AnglePath="Angle"
                                  RadiusPath="Radius"
                                  StrokeThickness="2" />
    </PolarChart.Series>
</PolarChart>
```

### 数据模型 (C#)

```csharp
public record PolarPoint(double Angle, double Radius);

public ObservableCollection<PolarPoint> SpiralData { get; } = new()
{
    new(0, 0),
    new(45, 10),
    new(90, 20),
    new(135, 30)
};
```

## 通用属性 (`PolarChart`)

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Series` | `PolarLineSeries` 项目的内容集合。 | 空集合 |
| `ShowGridLines` | 是否绘制角度和径向网格线。 | `true` |
| `GridLineBrush` | 用于网格的画笔。 | `null` |
| `GridLineStrokeThickness` | 网格线的粗细。 | `1.0` |
| `RadiusAxisMin` | 半径轴的最小值。 | `0.0` |
| `RadiusAxisMax` | 半径轴的最大值。为 `NaN` 时，图表从数据中计算。 | `NaN` |
| `StartAngle` | 视觉起始角度（度）。 | `-90.0` |
| `IsHighlightEnabled` | 启用极坐标点的图表级悬停高亮。 | `false` |

## 通用属性 (`PolarLineSeries`)

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 极坐标数据点的集合。 | `null` |
| `AnglePath` | 角度值（度）的路径。 | `null` |
| `RadiusPath` | 半径值的路径。 | `null` |
| `ShowMarkers` | 是否在每个点绘制标记。 | `false` |
| `MarkerSize` | 标记大小（像素）。 | `8.0` |
| `IsClosed` | 是否将最后一个点连接回第一个点。 | `false` |

## 另请参阅

- [雷达图](/controls/data-display/charts/radial/radar-chart)
- [径向折线图](/controls/data-display/charts/radial/radial-line-chart)
