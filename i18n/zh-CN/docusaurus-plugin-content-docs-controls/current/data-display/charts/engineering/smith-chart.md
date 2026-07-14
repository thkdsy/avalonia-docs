---
id: smith-chart
title: 史密斯图
description: 在归一化反射系数网格上可视化复阻抗或导纳，用于射频和匹配分析。
doc-type: reference
tags:
  - avalonia pro
---

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

史密斯图将归一化电阻和电抗绘制在专用的圆形网格上，用于射频（RF）、天线和阻抗匹配工作。

## 何时使用

- **射频分析**：可视化阻抗随频率变化的轨迹。
- **匹配网络**：检查设计如何向中心匹配点靠近或远离。
- **传输线工作**：在熟悉的网格上检查复杂的反射行为。

## 代码示例

### XAML

```xml
<SmithChart xmlns="https://github.com/avaloniaui" Title="Impedance trace"
                             Height="320"
                             ItemsSource="{Binding ImpedanceData}"
                             ResistancePath="Resistance"
                             ReactancePath="Reactance" />
```

### 数据模型（C#）

```csharp
public record ImpedancePoint(double Resistance, double Reactance);

public ObservableCollection<ImpedancePoint> ImpedanceData { get; } = new()
{
    new(0.5, -0.3),
    new(0.8, 0.1),
    new(1.2, 0.4)
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 阻抗点的集合。 | `null` |
| `ResistancePath` | 归一化电阻值的路径。 | `null` |
| `ReactancePath` | 归一化电抗值的路径。 | `null` |
| `StrokeThickness` | 绘制轨迹的线条粗细。 | `2.0` |

## 另请参阅

- [极坐标图](/controls/data-display/charts/radial/polar-chart)
- [风玫瑰图](/controls/data-display/charts/engineering/wind-rose-chart)
