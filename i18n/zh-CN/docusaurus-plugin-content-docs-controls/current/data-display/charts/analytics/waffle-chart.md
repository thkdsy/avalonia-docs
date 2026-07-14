---
id: waffle-chart
title: 华夫饼图
description: 以正方形网格形式可视化百分比或比例，展示部分与整体的关系以及目标完成情况。
doc-type: reference
tags:
  - avalonia pro
---

import chartsAnalyticsWaffle from '/img/controls/charts/charts-analytics-waffle.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

华夫饼图（方形饼图）使用正方形网格来可视化百分比或比例。它们以网格形式展示部分与整体的关系。

<Image light={chartsAnalyticsWaffle} maxWidth={400} position="center" cornerRadius="true" alt="华夫饼图，显示一个 10x10 的正方形网格，填充的正方形代表占总数的百分比。" />

## 使用场景
- **目标完成**：可视化项目接近 100% 目标的程度。
- **人口比例**：展示不同群体在人口中的分布。
- **项目跟踪**：显示冲刺中已完成任务的百分比。

## 代码示例

### XAML
```xml
<WrapPanel HorizontalAlignment="Center">
    <WaffleChart xmlns="https://github.com/avaloniaui" Title="Completion" Value="72" Width="150" Height="150" Rows="10" Columns="10" Margin="0,0,20,8" />
    <WaffleChart Title="Progress" Value="45" Width="150" Height="150" Rows="10" Columns="10" Margin="0,0,0,8" />
</WrapPanel>
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Value` | 要显示的当前值。 | `0` |
| `MaxValue` | 用于百分比计算的最大值。 | `100` |
| `Rows` | 华夫饼网格的行数。 | `10` |
| `Columns` | 华夫饼网格的列数。 | `10` |
| `FilledBrush` | 用于活动（已填充）正方形的画笔。 | 取决于主题 |
| `EmptyBrush` | 用于非活动（空白）正方形的画笔。 | 取决于主题 |
| `CellGap` | 华夫饼单元格之间的间距。 | `2.0` |
| `CellCornerRadius` | 华夫饼单元格的圆角半径。 | `2.0` |
| `ShowPercentage` | 是否显示百分比文本。 | `true` |
| `Label` | 在百分比文本下方显示的可选标签。 | `null` |
