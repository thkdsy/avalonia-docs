---
id: radial-line-chart
title: 径向折线图
description: 在极坐标系中绘制数据点并用线连接，适合显示变量如何在循环或方向类别中波动。
doc-type: reference
tags:
  - avalonia pro
---

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

径向折线图在 `PolarChart` 上绘制数据点，并用线连接它们，由 `PolarLineSeries` 定义。它们非常适合显示单个变量如何在循环类别中波动。

## 何时使用
- **日常活动**：绘制 24 小时的心率或能量水平。
- **方向数据**：可视化来自 360 度传感器的读数。
- **对称性分析**：检查多变量配置文件中的模式和平衡。

## 代码示例

### XAML
```xml
<PolarChart xmlns="https://github.com/avaloniaui" Title="每小时活动" Height="350">
    <PolarChart.Series>
        <PolarLineSeries ItemsSource="{Binding RadialPoints}"
                                  AnglePath="Angle"
                                  RadiusPath="Radius"
                                  ShowMarkers="True" />
    </PolarChart.Series>
</PolarChart>
```

### 数据模型 (C#)
```csharp
public record ActivityPoint(double Angle, double Radius);

public ObservableCollection<ActivityPoint> RadialPoints { get; } = new()
{
    new(0, 10),
    new(60, 25),
    new(120, 45),
    new(180, 30),
    new(240, 60),
    new(300, 15)
};
```

## 通用属性: `PolarLineSeries`

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 要连接的点集合。 | `null` |
| `AnglePath` | 角度路径（X）。 | `null` |
| `RadiusPath` | 半径路径（Y）。 | `null` |
| `ShowMarkers` | 是否在每个数据点显示标记。 | `false` |
| `MarkerSize` | 标记的大小。 | `8.0` |
| `IsClosed` | 是否连接第一个和最后一个数据点。 | `false` |
