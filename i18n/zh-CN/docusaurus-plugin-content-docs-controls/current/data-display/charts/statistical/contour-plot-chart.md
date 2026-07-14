---
id: contour-plot-chart
title: 等高线图
description: 将二维标量场显示为等高线和可选填充带，适用于曲面、强度图和插值。
doc-type: reference
tags:
  - avalonia pro
---

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

等高线图在两个空间维度上插值，并将结果渲染为等值线、填充区域或两者兼具。

## 使用时机

- **曲面估计**：通过散点测量值可视化标量场。
- **热点分析**：在二维空间中揭示峰值、谷值和梯度。
- **工程地图**：显示压力、温度或浓度曲面。

## 代码示例

### XAML

```xml
<ContourPlot xmlns="https://github.com/avaloniaui" Title="温度场"
                              Height="320"
                              ItemsSource="{Binding ContourData}"
                              XPath="X"
                              YPath="Y"
                              ValuePath="Temperature"
                              ContourLevels="10" />
```

### 数据模型 (C#)

```csharp
public record ContourPoint(double X, double Y, double Temperature);

public ObservableCollection<ContourPoint> ContourData { get; } = new()
{
    new(0, 0, 18),
    new(0, 10, 24),
    new(10, 0, 21),
    new(10, 10, 28)
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 采样点的集合。 | `null` |
| `XPath` | X 坐标的路径。 | `null` |
| `YPath` | Y 坐标的路径。 | `null` |
| `ValuePath` | 标量值的路径。 | `null` |
| `ContourLevels` | 要计算的等高线级别数。 | `8` |
| `ShowFill` | 是否填充等高线区域。 | `true` |
| `ShowLines` | 是否绘制等高线。 | `true` |

## 另请参阅

- [六边形分箱图](/controls/data-display/charts/engineering/hexbin-chart)
- [密度图](/controls/data-display/charts/statistical/density-plot-chart)
