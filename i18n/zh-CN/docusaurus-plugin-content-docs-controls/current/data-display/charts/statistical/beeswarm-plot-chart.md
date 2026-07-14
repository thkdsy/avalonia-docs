---
id: beeswarm-plot-chart
title: 蜂群图
description: 将每个分类中的单个观测值显示为不重叠的点，无需完全抖动噪声即可保留点密度。
doc-type: reference
tags:
  - avalonia pro
---

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

蜂群图在每个分类内排列点以避免重叠，同时保留单个观测值的分布。

## 使用时机

- **原始观测显示**：显示每个点，而非仅显示汇总统计。
- **分类比较**：比较各组的分布和聚类情况。
- **分布细节**：揭示简单散点图可能隐藏的密集堆叠。

## 代码示例

### XAML

```xml
<BeeswarmPlotChart xmlns="https://github.com/avaloniaui" Title="按组分组的测试分数"
                                    Height="300"
                                    ItemsSource="{Binding BeeswarmData}"
                                    CategoryPath="Category"
                                    ValuePath="Value" />
```

### 数据模型 (C#)

```csharp
public record BeeswarmPoint(string Category, double Value);

public ObservableCollection<BeeswarmPoint> BeeswarmData { get; } = new()
{
    new("A", 62),
    new("A", 65),
    new("B", 74),
    new("B", 79)
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 观测值的集合。 | `null` |
| `CategoryPath` | 分组分类的路径。 | `null` |
| `ValuePath` | 数值的路径。 | `null` |
| `PointRadius` | 每个点的半径。 | `5.0` |
| `Fill` | 填充点使用的画刷。 | `null` |
| `Stroke` | 点轮廓使用的画刷。 | `null` |
| `StrokeThickness` | 点轮廓的粗细。 | `1.0` |
| `ShowCategoryLabels` | 是否绘制分类标签。 | `true` |
| `ShowAxes` | 是否绘制数值轴。 | `true` |

## 另请参阅

- [散点条图](/controls/data-display/charts/statistical/strip-plot-chart)
- [小提琴图](/controls/data-display/charts/statistical/violin-plot-chart)
