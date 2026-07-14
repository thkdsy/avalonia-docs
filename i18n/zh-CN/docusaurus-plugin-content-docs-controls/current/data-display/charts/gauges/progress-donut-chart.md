---
id: progress-donut-chart
title: 进度甜甜圈图
description: 甜甜圈图的变体，用于显示朝向单个100%目标的进度，常见于仪表板和健身应用。
doc-type: reference
tags:
  - avalonia pro
---

import chartsGaugesProgressDonut from '/img/controls/charts/charts-pie-progressdonut.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

进度甜甜圈图是甜甜圈图的一个专门变体，旨在显示朝向单个100%目标的进度。它们通常用于健身应用或系统仪表板。

<Image light={chartsGaugesProgressDonut} maxWidth={400} position="center" cornerRadius="true" alt="进度甜甜圈图，显示按比例填充的弧形以表示朝向100%目标的完成进度。" />

## 何时使用
- **目标完成度**：显示用户距离目标还有多远。
- **指标汇总**：可视化基于百分比的数据（例如，已用磁盘空间）。
- **KPI 仪表板**：为关键绩效指标提供快速的视觉检查。

## 代码示例

### XAML
```xml
<WrapPanel Orientation="Horizontal" HorizontalAlignment="Center">
    <ProgressDonutChart xmlns="https://github.com/avaloniaui" Value="75" Width="180" Height="180" IsTooltipEnabled="True" Title="Downloads" Margin="10" />
    <ProgressDonutChart Value="42" Width="180" Height="180" IsTooltipEnabled="True" Title="Uploads" Margin="10" />
    <ProgressDonutChart Value="90" Width="180" Height="180" IsTooltipEnabled="True" Title="Active" Margin="10" />
</WrapPanel>
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Value` | 当前值（0 到 MaxValue）。 | `0` |
| `MaxValue` | 最大值。 | `100.0` |
| `ValueBrush` | 进度弧的画笔。 | 取决于主题 |
| `TrackBrush` | 环形空白部分的画笔。 | 取决于主题 |
| `RingThickness` | 环的宽度。 | `20.0` |
| `StartAngle` | 起始角度（度，-90 = 顶部）。 | `-90.0` |
| `ShowPercentage` | 是否在中心显示百分比。 | `true` |
| `CenterLabel` | 自定义中心标签（覆盖百分比）。 | `null` |
| `IsValueAnimationEnabled` | 是否对值弧的变化进行动画处理。 | `false` |
| `ShowGlow` | 是否在动画值弧周围绘制发光效果。 | `false` |
