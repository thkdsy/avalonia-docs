---
id: radar-chart
title: 雷达图
description: 在径向轴上比较跨类别的多个定量变量，用于可视化概况和多维性能比较。
doc-type: reference
tags:
  - avalonia pro
---

import chartsRadialRadar from '/img/controls/charts/charts-radial-radar.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

雷达图比较跨多个类别的多个定性变量。它们被广泛用于可视化不同实体在共同指标上的"概况"或特征。

<Image light={chartsRadialRadar} maxWidth={400} position="center" cornerRadius="true" alt="雷达图，具有两个重叠的多边形，比较跨径向轴的多维技能分数。" />

## 何时使用
- **技能评估**：比较员工或运动员的优势/劣势。
- **产品基准测试**：比较不同产品的价格、质量和功能。
- **性能概况**：显示多面指标（如网站的 SEO、速度、安全性）。

## 代码示例

### XAML

```xml
<RadarChart xmlns="https://github.com/avaloniaui" Name="RadarChartSample" Title="团队技能" Height="350" IsTooltipEnabled="True"
                                         AxisLabels="{Binding RadarLabels}">
                        <RadarChart.Series>
                            <RadarSeries Title="玩家 A" ItemsSource="{Binding RadarSeries1}" FillOpacity="0.3" ShowMarkers="True" />
                            <RadarSeries Title="玩家 B" ItemsSource="{Binding RadarSeries2}" FillOpacity="0.3" ShowMarkers="True" />
                        </RadarChart.Series>
                     </RadarChart>
```

### 数据模型 (C#)

```csharp
// 映射到类别的轴标签
public ObservableCollection<string> RadarLabels { get; } = new()
{
    "速度", "力量", "敏捷", "防御", "耐力", "技巧"
};

// 按顺序排列的轴值
public ObservableCollection<double> RadarSeries1 { get; } = new() { 80, 90, 70, 60, 85, 75 };
public ObservableCollection<double> RadarSeries2 { get; } = new() { 60, 70, 85, 80, 65, 90 };
```

## 通用属性 (`RadarChart`)

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `AxisCount` | 图表周围显示的轴数。 | `5` |
| `ShowGridLines` | 是否绘制同心雷达网格。 | `true` |
| `GridLevels` | 同心网格级别数。 | `5` |
| `AxisLabels` | 为每个轴显示的标签。 | `null` |
| `IsHighlightEnabled` | 启用雷达点的图表级悬停高亮。 | `false` |

## 通用属性 (`RadarSeries`)

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 雷达轴的数值。 | `null` |
| `Title` | 玩家/实体的名称（用于图例）。 | `null` |
| `MaxValue` | 用于缩放系列的最大值。 | `100.0` |
| `Fill` | 用于多边形区域的画笔。为 `null` 时，图表使用系列调色板画笔并应用 `FillOpacity`。 | `null` |
| `Stroke` | 用于多边形边界的画笔。为 `null` 时，图表使用系列调色板画笔。 | `null` |
| `FillOpacity` | 应用于填充多边形区域的不透明度。 | `0.3` |
| `ShowMarkers` | 在轴交点处切换点显示。 | `true` |
| `MarkerSize` | 轴交点处标记的大小。 | `6` |
