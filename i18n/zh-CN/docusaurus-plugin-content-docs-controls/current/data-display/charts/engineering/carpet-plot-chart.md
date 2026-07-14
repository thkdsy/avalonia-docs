---
id: carpet-plot-chart
title: 地毯图
description: 将两个自变量和一个因变量映射到倾斜网格中，适用于工程权衡曲面分析。
doc-type: reference
tags:
  - avalonia pro
---

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

地毯图通过绘制由相交等值线构成的扭曲网格，可视化两个自变量与第三个值之间的关系。

## 何时使用

- **工程权衡**：比较两个设计输入与结果曲面的关系。
- **性能图谱**：可视化效率、压力或温度的工作区域。
- **多变量分析**：检查不适合简单笛卡尔折线图的关系。

## 代码示例

### XAML

```xml
<CarpetPlot xmlns="https://github.com/avaloniaui" Title="Performance map"
                             Height="320"
                             ItemsSource="{Binding CarpetData}"
                             AAxisPath="Speed"
                             BAxisPath="Load"
                             YAxisPath="Efficiency" />
```

### 数据模型（C#）

```csharp
public record CarpetPoint(double Speed, double Load, double Efficiency);

public ObservableCollection<CarpetPoint> CarpetData { get; } = new()
{
    new(1000, 20, 62),
    new(1000, 40, 68),
    new(1500, 20, 70),
    new(1500, 40, 74)
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 测量点的集合。 | `null` |
| `AAxisPath` | 第一个自变量的路径。 | `null` |
| `BAxisPath` | 第二个自变量的路径。 | `null` |
| `YAxisPath` | 因变量的路径。 | `null` |
| `CarpetOffset` | 用于创建地毯效果的视觉偏移因子。 | `0.5` |
| `PlotAreaBackground` | 绘图区域的可选背景画笔。 | `null` |

## 另请参阅

- [三元图](/controls/data-display/charts/engineering/ternary-chart)
- [等高线图](/controls/data-display/charts/statistical/contour-plot-chart)
