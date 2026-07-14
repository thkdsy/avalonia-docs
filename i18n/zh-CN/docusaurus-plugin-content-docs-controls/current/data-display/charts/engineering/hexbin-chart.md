---
id: hexbin-chart
title: 六边形分箱图
description: 将密集的二维点云聚合成六边形分箱，用于可视化浓度分布而避免过度绘制。
doc-type: reference
tags:
  - avalonia pro
---

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

六边形分箱图将附近的点分组到六边形中，使得即使数千个点重叠，密集的散点数据仍然可读。

## 何时使用

- **密集散点数据**：用密度分箱替代难以辨认的点云。
- **空间集中度**：显示观测值在二维空间中的聚类位置。
- **探索性分析**：在不平滑原始数据的情况下揭示热点和梯度。

## 代码示例

### XAML

```xml
<HexbinChart xmlns="https://github.com/avaloniaui" Title="Request concentration"
                              Height="320"
                              ItemsSource="{Binding HexbinData}"
                              XPath="X"
                              YPath="Y"
                              HexRadius="16" />
```

### 数据模型（C#）

```csharp
public record SamplePoint(double X, double Y);

public ObservableCollection<SamplePoint> HexbinData { get; } = new()
{
    new(12, 20),
    new(14, 22),
    new(13, 21),
    new(28, 35)
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | X 和 Y 点的集合。 | `null` |
| `XPath` | X 值的路径。 | `null` |
| `YPath` | Y 值的路径。 | `null` |
| `HexRadius` | 每个六边形的半径（以像素为单位）。 | `20.0` |
| `ColorScale` | 用于编码密度的颜色刻度。 | `Blues` |
| `ShowAxes` | 是否绘制图表轴。 | `true` |

## 另请参阅

- [气泡图](/controls/data-display/charts/bubble/bubble-chart)
- [等高线图](/controls/data-display/charts/statistical/contour-plot-chart)
