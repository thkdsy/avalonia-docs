---
id: mekko-chart
title: 马赛克图（Mekko 图）
description: 结合可变宽度的列和堆叠段，比较总大小和内部构成。
doc-type: reference
tags:
  - avalonia pro
---

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

马赛克图，也称为 Marimekko 图，使用可变的列宽和堆叠的段高度，在一个视图中比较市场规模和构成。

## 何时使用

- **市场结构**：同时比较品类份额和内部段组合。
- **投资组合构成**：显示每组的总体规模和细分。
- **多维比较**：用单个图表替代单独的宽度和堆叠柱状图视图。

## 代码示例

### XAML

```xml
<MekkoChart xmlns="https://github.com/avaloniaui" Title="市场组合"
                             Height="320"
                             ItemsSource="{Binding MekkoData}"
                             CategoryPath="Category"
                             WidthPath="Width"
                             SegmentsPath="Segments" />
```

### 数据模型 (C#)

```csharp
public record MekkoSegment(string Name, double Value);
public record MekkoColumn(string Category, double Width, ObservableCollection<MekkoSegment> Segments);

public ObservableCollection<MekkoColumn> MekkoData { get; } = new()
{
    new("北部", 35, new() { new("零售", 18), new("在线", 12), new("合作伙伴", 5) }),
    new("南部", 25, new() { new("零售", 10), new("在线", 9), new("合作伙伴", 6) })
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | Mekko 列的集合。 | `null` |
| `CategoryPath` | 列标签的路径。 | `null` |
| `WidthPath` | 控制列宽的值的路径。 | `null` |
| `SegmentsPath` | 每列的段集合路径。 | `null` |
| `ColumnGap` | 列之间的间隙。 | `2.0` |
| `ShowLabels` | 是否显示列标签。 | `true` |
| `ShowPercentages` | 是否在段内绘制百分比标签。 | `true` |

## 另请参阅

- [堆叠柱状图](/controls/data-display/charts/cartesian/stacked-bar-chart)
- [柱状图](/controls/data-display/charts/cartesian/bar-chart)
