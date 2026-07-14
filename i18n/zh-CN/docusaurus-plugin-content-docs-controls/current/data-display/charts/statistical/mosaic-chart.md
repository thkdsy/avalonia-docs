---
id: mosaic-chart
title: 马赛克图
description: 通过缩放段的宽度和高度来表示其在总体中的比例，从而可视化两个分类变量之间的关系。
doc-type: reference
tags:
  - avalonia pro
---

import chartsAnalyticsMosaic from '/img/controls/charts/charts-analytics-mosaic.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

马赛克图（Marimekko 图）使用区域来可视化分类之间的关系。段的宽度和高度都被缩放以表示占总数的百分比。

<Image light={chartsAnalyticsMosaic} maxWidth={400} position="center" cornerRadius="true" alt="马赛克图，矩形瓷砖按宽度和高度缩放，显示两个分类变量的比例。" />

## 使用时机
- **市场细分**：按地区（宽度）和产品类别（高度）显示销售情况。
- **资源支出**：可视化跨部门和成本类型的预算分配。
- **多因素分析**：了解两个不同定性变量如何交互。

## 代码示例

### XAML
```xml
<MosaicChart xmlns="https://github.com/avaloniaui" Title="按地区划分的销售" Height="300"
                      ItemsSource="{Binding MosaicData}"
                      GroupPath="Region"
                      SubGroupPath="Category"
                      ValuePath="Sales" />
```

### 数据模型 (C#)
```csharp
public record MosaicItem(string Region, string Category, double Sales);

public ObservableCollection<MosaicItem> MosaicData { get; } = new()
{
    new("北部", "电子产品", 450),
    new("北部", "服装", 320),
    new("北部", "家居", 210),
    new("南部", "电子产品", 380),
    new("南部", "服装", 410),
    new("南部", "家居", 180),
    new("东部", "电子产品", 290),
    new("东部", "服装", 250),
    new("东部", "家居", 350),
    new("西部", "服装", 280),
    new("西部", "家居", 150)
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 数据段的集合。 | `null` |
| `GroupPath` | 主分类（决定宽度）。 | `null` |
| `SubGroupPath` | 次分类（决定高度）。 | `null` |
| `ValuePath` | 用于缩放的数值。 | `null` |
