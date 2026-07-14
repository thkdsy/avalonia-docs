---
id: tornado-chart
title: 龙卷风图
description: 围绕中心线绘制双向水平柱条，常用于敏感性分析和排名比较。
doc-type: reference
tags:
  - avalonia pro
---

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

龙卷风图将左右柱条放置在共享中心线周围，使排名的并排差异保持可扫描性。

## 何时使用

- **敏感性分析**：对哪些变量对结果影响最大进行排序（向左或向右）。
- **场景比较**：比较同一品类中的两个对立值。
- **优先级审查**：首先关注最大的绝对差异。

## 代码示例

### XAML

```xml
<TornadoChart xmlns="https://github.com/avaloniaui" Title="敏感性驱动因素"
                       Height="320"
                       ItemsSource="{Binding TornadoData}"
                       LabelPath="Factor"
                       LeftValuePath="Downside"
                       RightValuePath="Upside" />
```

### 数据模型 (C#)

```csharp
public record TornadoFactor(string Factor, double Downside, double Upside);

public ObservableCollection<TornadoFactor> TornadoData { get; } = new()
{
    new("需求", 18, 26),
    new("价格", 12, 20),
    new("成本", 22, 14)
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 因素或品类的集合。 | `null` |
| `LeftValuePath` | 左侧值的路径。 | `null` |
| `RightValuePath` | 右侧值的路径。 | `null` |
| `LabelPath` | 品类标签的路径。 | `null` |
| `LeftBrush` | 用于左侧柱条的画刷。 | `#E91E63` |
| `RightBrush` | 用于右侧柱条的画刷。 | `#2196F3` |
| `BarHeight` | 柱条高度，以行高的比例表示。 | `0.7` |
| `CenterGap` | 两侧之间的间隙。 | `4.0` |
| `IsHighlightEnabled` | 启用龙卷风柱条的悬停高亮。 | `false` |

## 另请参阅

- [镜像柱状图](/controls/data-display/charts/comparison/mirror-bar-chart)
- [人口金字塔图](/controls/data-display/charts/comparison/population-pyramid-chart)
