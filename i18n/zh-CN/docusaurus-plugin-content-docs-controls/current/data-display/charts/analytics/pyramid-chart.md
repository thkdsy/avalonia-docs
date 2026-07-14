---
id: pyramid-chart
title: 金字塔图
description: 以堆叠三角形布局同时强调层次结构和数据量，常用于人口和流程可视化。
doc-type: reference
tags:
  - avalonia pro
---

import chartsAnalyticsPyramid from '/img/controls/charts/charts-analytics-pyramid.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

金字塔图是一种堆叠面积图或柱状图的变体，同时强调层次结构和数据量。它们是展示人口和销售管道的经典选择。

<Image light={chartsAnalyticsPyramid} maxWidth={400} position="center" cornerRadius="true" alt="金字塔图，由堆叠的三角形分段组成，表示层次化的人口或管道数据。" />

## 使用场景
- **人口金字塔**：展示某一地区按年龄和性别的分布。
- **销售管道**：可视化从线索到成交的漏斗过程。
- **生物层次结构**：展示生态系统中的能量流动或物种分布。

## 代码示例

### XAML
```xml
<PyramidChart xmlns="https://github.com/avaloniaui" Title="Population Distribution" Height="300"
                       ItemsSource="{Binding PyramidData}"
                       LabelPath="Age" ValuePath="Value"/>
```

### 数据模型 (C#)
```csharp
public record PyramidItem(string Age, double Value);

public ObservableCollection<PyramidItem> PyramidData { get; } = new()
{
    new("0-14", 15),
    new("15-24", 12),
    new("25-54", 40),
    new("55-64", 18),
    new("65+", 15)
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 数据层的集合。 | `null` |
| `ValuePath` | 每层的数值大小。 | `null` |
| `LabelPath` | 每层的文本描述。 | `null` |
| `SegmentGap` | 层级之间的垂直间距。 | `2.0` |
| `ShowLabels` | 是否在分段上显示标签。 | `true` |
| `ShowValues` | 是否在图表上显示数值。 | `true` |
| `IsHighlightEnabled` | 启用金字塔分段的悬停高亮。 | `false` |
