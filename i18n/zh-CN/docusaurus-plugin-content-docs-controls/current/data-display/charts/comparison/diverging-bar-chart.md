---
id: diverging-bar-chart
title: 发散柱状图
description: 从基线向左右延伸柱条，以比较正值和负值或对立响应。
doc-type: reference
tags:
  - avalonia pro
---

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

发散柱状图使用居中的基线，使值可以从同一原点向相反方向延伸。

## 何时使用

- **情感分析**：显示零附近的正向和负向响应。
- **差异视图**：比较相对于基线的超额和不足表现。
- **平衡比较**：突出方向性差异，无需两个单独的图表。

## 代码示例

### XAML

```xml
<DivergingBarChart xmlns="https://github.com/avaloniaui" Title="净情感值"
                            Height="300"
                            ItemsSource="{Binding SentimentData}"
                            LabelPath="Label"
                            ValuePath="Score" />
```

### 数据模型 (C#)

```csharp
public record SentimentPoint(string Label, double Score);

public ObservableCollection<SentimentPoint> SentimentData { get; } = new()
{
    new("产品 A", 24),
    new("产品 B", -12),
    new("产品 C", 8)
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 要比较的项集合。 | `null` |
| `ValuePath` | 相对于基线绘制的值的路径。 | `null` |
| `LabelPath` | 品类标签的路径。 | `null` |
| `Baseline` | 中心基线值。 | `0.0` |
| `PositiveBrush` | 用于基线上方值的画刷。 | `null` |
| `NegativeBrush` | 用于基线下方值的画刷。 | `null` |
| `BarHeight` | 柱条高度，以行高的比例表示。 | `0.7` |
| `ShowValues` | 是否在柱条内部绘制值标签。 | `true` |
| `IsHighlightEnabled` | 启用柱条的悬停高亮。 | `false` |

## 另请参阅

- [柱状图](/controls/data-display/charts/cartesian/bar-chart)
- [镜像柱状图](/controls/data-display/charts/comparison/mirror-bar-chart)
