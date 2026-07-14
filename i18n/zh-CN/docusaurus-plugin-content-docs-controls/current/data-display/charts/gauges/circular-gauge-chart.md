---
id: circular-gauge-chart
title: 圆形仪表图
description: 在速度表样式的径向刻度上可视化单个值，适用于实时监控和目标跟踪仪表板。
doc-type: reference
tags:
  - avalonia pro
---

import chartsGaugesCircular from '/img/controls/charts/charts-gauges-circular.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

圆形仪表图在径向刻度上可视化单个值。它们是高级仪表板指标的标准选择，其中"速度表"风格的视觉直观易懂。

<Image light={chartsGaugesCircular} maxWidth={400} position="center" cornerRadius="true" alt="速度表风格的圆形仪表图，带指针指向径向刻度上的当前值。" />

## 何时使用
- **实时监控**：显示 CPU、内存或网络使用情况。
- **目标跟踪**：可视化朝着目标（如销售配额）的进度。
- **物理模拟**：表示来自物理传感器（如速度或压力）的值。

## 代码示例

### XAML
```xml
<WrapPanel Orientation="Horizontal" HorizontalAlignment="Center">
    <CircularGaugeChart xmlns="https://github.com/avaloniaui" Value="72" Width="220" Height="220" Title="CPU" Margin="10" />
    <CircularGaugeChart Value="45" Width="220" Height="220" Title="Memory" Margin="10" />
    <CircularGaugeChart Value="88" Width="220" Height="220" Title="Disk" Margin="10" />
</WrapPanel>
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Value` | 要显示的当前值。 | `0.0` |
| `MinValue` | 刻度的最小值。 | `0.0` |
| `MaxValue` | 刻度的最大值。 | `100.0` |
| `ShowValue` | 是否显示数值。 | `true` |
| `ValueFormat` | 显示值的格式字符串。 | `"{0:F0}"` |
| `StartAngle` | 仪表弧的起始角度（度）。 | `135.0` |
| `SweepAngle` | 仪表弧的扫掠角度（度）。 | `270.0` |
| `TrackBrush` | 仪表轨道的画笔。 | `null` |
| `ValueBrush` | 填充值弧的画笔。 | `null` |
| `NeedleBrush` | 指针的画笔。 | `null` |
| `TrackThickness` | 轨道弧的粗细。 | `10.0` |
| `MajorTickCount` | 主刻度标记的数量。 | `5` |

## 另请参阅

- [仪表图](/controls/data-display/charts/gauges/gauge-chart)
- [进度甜甜圈图](/controls/data-display/charts/gauges/progress-donut-chart)
