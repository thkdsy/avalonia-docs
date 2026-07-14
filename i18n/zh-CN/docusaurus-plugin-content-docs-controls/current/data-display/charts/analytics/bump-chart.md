---
id: bump-chart
title: 凹凸图
description: 展示排名随时间的变化，侧重于各个类别的相对位置而非绝对数值。
doc-type: reference
tags:
  - avalonia pro
---

import chartsAnalyticsBump from '/img/controls/charts/charts-statistical-bump.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

凹凸图是折线图的一种变体，用于展示排名随时间的变化。它们关注的是各个类别的相对位置，而非其绝对数值。

<Image light={chartsAnalyticsBump} maxWidth={400} position="center" cornerRadius="true" alt="凹凸图，通过平滑曲线展示各实体在不同时间段的排名变化。" />

## 使用场景
- **热度排名**：展示歌曲或电影在排行榜上的升降情况。
- **市场份额**：可视化品牌之间对榜首位置的竞争。
- **比赛排名**：跟踪各团队在不同阶段的相对表现。

## 代码示例

### XAML
```xml
<BumpChart xmlns="https://github.com/avaloniaui" Name="BumpChartSample"
                                        Title="Ranking Changes"
                                        Height="300"
                                        NamePath="Name"
                                        RankingsPath="Ranks"
                                        Periods="{Binding BumpPeriods}"
                                        ItemsSource="{Binding BumpData}" />
```

### 数据模型 (C#)
```csharp
public record BumpItem(string Name, int[] Ranks);

public ObservableCollection<BumpItem> BumpData { get; } = new()
{
    new("Java", new[] { 1, 2, 2, 3, 3 }),
    new("C#", new[] { 4, 3, 3, 2, 2 }),
    new("Python", new[] { 2, 1, 1, 1, 1 }),
    new("JS", new[] { 3, 4, 4, 4, 4 })
};

public ObservableCollection<string> BumpPeriods { get; } = new()
{
    "2020", "2021", "2022", "2023", "2024"
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 排名实体的集合。 | `null` |
| `NamePath` | 表示实体名称的属性。 | `null` |
| `RankingsPath` | 表示每个实体排名序列的属性。 | `null` |
| `Periods` | 水平轴上显示的标签。 | `null` |
| `StrokeThickness` | 排名线的宽度。 | `3.0` |
| `MarkerSize` | 每个时期绘制的标记大小。 | `10.0` |
| `ShowLabels` | 是否在每条线末端显示实体标签。 | `true` |
| `ShowRankNumbers` | 是否在垂直轴上显示排名数字。 | `true` |
