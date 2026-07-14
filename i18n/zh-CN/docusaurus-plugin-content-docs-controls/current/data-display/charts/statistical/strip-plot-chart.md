---
id: strip-plot-chart
title: 散点条图
description: 在每个分类中显示单个观测值并带有受控抖动，适用于原始值比较和均值叠加。
doc-type: reference
tags:
  - avalonia pro
---

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

散点条图显示分类中的每个观测值，同时应用抖动以减少重叠，并可选择均值线来汇总中心。

## 使用时机

- **原始样本显示**：显示每个点而非仅显示四分位数或平均值。
- **分类分布**：比较值在每组中的紧密或分散程度。
- **混合视图**：将原始观测值与简单均值参考线结合。

## 代码示例

### XAML

```xml
<StripPlotChart xmlns="https://github.com/avaloniaui" Title="响应时间"
                                 Height="300"
                                 ItemsSource="{Binding StripData}"
                                 CategoryPath="Category"
                                 ValuePath="Value"
                                 JitterAmount="0.3"
                                 ShowMeanLine="True" />
```

### 数据模型 (C#)

```csharp
public record StripPoint(string Category, double Value);

public ObservableCollection<StripPoint> StripData { get; } = new()
{
    new("API", 120),
    new("API", 128),
    new("UI", 95),
    new("UI", 102)
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 观测值的集合。 | `null` |
| `CategoryPath` | 分组分类的路径。 | `null` |
| `ValuePath` | 数值的路径。 | `null` |
| `PointRadius` | 每个点的半径。 | `4.0` |
| `JitterAmount` | 在每个分类内应用的水平抖动因子。 | `0.3` |
| `Fill` | 填充点使用的画刷。 | `null` |
| `Stroke` | 点轮廓使用的画刷。 | `null` |
| `StrokeThickness` | 点轮廓的粗细。 | `0.5` |
| `PointOpacity` | 绘制点的透明度。 | `0.7` |
| `ShowCategoryLabels` | 是否绘制分类标签。 | `true` |
| `ShowAxes` | 是否绘制数值轴。 | `true` |
| `ShowMeanLine` | 是否为每个分类绘制均值线。 | `true` |

## 另请参阅

- [蜂群图](/controls/data-display/charts/statistical/beeswarm-plot-chart)
- [箱线图](/controls/data-display/charts/statistical/boxplot-chart)
