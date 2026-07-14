---
id: linear-gauge-chart
title: 线性仪表图
description: 沿水平或垂直条可视化一个值，用于比较指标或显示进度。
doc-type: reference
tags:
  - avalonia pro
---

import chartsGaugesLinear from '/img/controls/charts/charts-gauges-linear-1.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

线性仪表图沿水平或垂直条可视化一个值。它们适合并排比较指标或以节省空间的线性格式显示进度。

<Image light={chartsGaugesLinear} maxWidth={400} position="center" cornerRadius="true" alt="线性仪表图，带水平进度条和刻度标记，显示沿数值范围的当前值。" />

## 何时使用
- **性能条**：在紧凑仪表板中比较多个指标。
- **容量指示器**：显示存储级别、音频级别或储罐容量。
- **进度跟踪**：在直线中可视化一系列目标。

## 代码示例

### XAML
```xml
<StackPanel Spacing="15" Margin="10">
    <LinearGaugeChart xmlns="https://github.com/avaloniaui" Title="Temperature" Value="72" MinValue="0" MaxValue="100" Height="60"
                             NeedleBrush="#F44336">
        <LinearGaugeChart.ValueBrush>
            <LinearGradientBrush StartPoint="0%,0%" EndPoint="100%,0%">
                <GradientStop Offset="0" Color="#4CAF50" />
                <GradientStop Offset="0.5" Color="#FFEB3B" />
                <GradientStop Offset="1" Color="#F44336" />
            </LinearGradientBrush>
        </LinearGaugeChart.ValueBrush>
    </LinearGaugeChart>

    <LinearGaugeChart Title="Humidity" Value="45" MinValue="0" MaxValue="100" Height="60"
                             NeedleBrush="#00BCD4">
        <LinearGaugeChart.ValueBrush>
            <LinearGradientBrush StartPoint="0%,0%" EndPoint="100%,0%">
                <GradientStop Offset="0" Color="#2196F3" />
                <GradientStop Offset="1" Color="#00BCD4" />
            </LinearGradientBrush>
        </LinearGaugeChart.ValueBrush>
    </LinearGaugeChart>

    <LinearGaugeChart Title="Pressure" Value="88" MinValue="0" MaxValue="100" Height="60"
                             NeedleBrush="#E91E63">
        <LinearGaugeChart.ValueBrush>
            <LinearGradientBrush StartPoint="0%,0%" EndPoint="100%,0%">
                <GradientStop Offset="0" Color="#9C27B0" />
                <GradientStop Offset="1" Color="#E91E63" />
            </LinearGradientBrush>
        </LinearGaugeChart.ValueBrush>
    </LinearGaugeChart>
</StackPanel>
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Value` | 要显示的当前值。 | `50.0` |
| `MinValue` | 刻度的最小值。 | `0.0` |
| `MaxValue` | 刻度的最大值。 | `100.0` |
| `Orientation` | 仪表的方向，`Horizontal` 或 `Vertical`。 | `Horizontal` |
| `ShowScale` | 是否显示刻度。 | `true` |
| `TrackBrush` | 轨道的颜色。 | 使用主题默认值。 |
| `NeedleBrush` | 指针指示器的颜色。 | 使用主题默认值。 |
| `ValueBrush` | 值的颜色。 | 使用主题默认值。 |
| `TrackThickness` | 轨道的粗细。 | `20.0` |
| `Ranges` | 可选的彩色范围集合，绘制在指示器后方。 | `null` |
| `ShowMajorTicks` | 是否显示主刻度。 | `false` |
| `ShowMinorTicks` | 是否显示次刻度。 | `false` |
| `MajorTickInterval` | 主刻度之间的间隔。 | `20.0` |
| `MinorTickCount` | 两个主刻度之间的次刻度数量。 | `4` |
