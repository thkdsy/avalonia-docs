---
id: population-pyramid-chart
title: 人口金字塔图
description: 按年龄段或其他有序品类，背靠背显示两个人口分布。
doc-type: reference
tags:
  - avalonia pro
---

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

人口金字塔图可视化两个对立的分布，最常见的是男性和女性人口在不同年龄组中的分布。

## 何时使用

- **人口统计分析**：按性别或地区比较年龄分布。
- **段比较**：显示有序带中的配对分布。
- **规划视图**：快速发现年轻或年长群体的集中程度。

## 代码示例

### XAML

```xml
<PopulationPyramidChart xmlns="https://github.com/avaloniaui" Title="年龄分布"
                                 Height="350"
                                 ItemsSource="{Binding PopulationData}"
                                 AgeLabelPath="AgeGroup"
                                 MaleValuePath="Male"
                                 FemaleValuePath="Female" />
```

### 数据模型 (C#)

```csharp
public record PopulationBand(string AgeGroup, double Male, double Female);

public ObservableCollection<PopulationBand> PopulationData { get; } = new()
{
    new("0-9岁", 12, 11),
    new("10-19岁", 14, 13),
    new("20-29岁", 16, 17)
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 人口带的集合。 | `null` |
| `AgeLabelPath` | 带标签的路径。 | `null` |
| `MaleValuePath` | 左侧人口值的路径。 | `null` |
| `FemaleValuePath` | 右侧人口值的路径。 | `null` |
| `MaleBrush` | 用于左侧柱条的画刷。 | `null` |
| `FemaleBrush` | 用于右侧柱条的画刷。 | `null` |
| `BarGap` | 相邻柱条之间的间隙。 | `2.0` |
| `ShowLabels` | 是否沿中心线显示带标签。 | `true` |
| `LabelFontSize` | 带标签使用的字号。 | `10.0` |
| `IsHighlightEnabled` | 启用人口带的悬停高亮。 | `false` |

## 另请参阅

- [镜像柱状图](/controls/data-display/charts/comparison/mirror-bar-chart)
- [龙卷风图](/controls/data-display/charts/comparison/tornado-chart)
