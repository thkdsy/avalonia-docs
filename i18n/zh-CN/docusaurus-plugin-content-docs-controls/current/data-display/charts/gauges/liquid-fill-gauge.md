---
id: liquid-fill-gauge
title: 液体填充仪表
description: 将百分比表示为圆形形状内的动画液位，适用于显示填充量或容量指标的主题仪表板。
doc-type: reference
tags:
  - avalonia pro
---

import chartsGaugesLiquid from '/img/controls/charts/charts-gauges-liquid.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

液体填充仪表是一种装饰性圆形仪表，将百分比表示为特定水平的"液体"。它们广受欢迎，用于可视化储罐液位、用水量或主题进度。

<Image light={chartsGaugesLiquid} maxWidth={400} position="center" cornerRadius="true" alt="液体填充仪表，显示一个圆形容器中充满动画液体以表示百分比数值。" />

## 何时使用
- **主题仪表板**：可视化"填充"概念，如筹款或容量。
- **环境应用**：显示水位或液体容器状态。
- **吸引人的 UI**：为现代应用程序添加有趣、动画化的指标指示器。

## 代码示例

### XAML
```xml
<WrapPanel Orientation="Horizontal" HorizontalAlignment="Center">
    <LiquidFillGauge xmlns="https://github.com/avaloniaui" Value="35" Width="140" Height="180" Title="CPU Usage" Margin="15" />
    <LiquidFillGauge Value="68" Width="140" Height="180" Title="Memory" Margin="15" />
    <LiquidFillGauge Value="85" Width="140" Height="180" Title="Storage" Margin="15" />
</WrapPanel>
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Value` | 当前值。 | `50.0` |
| `MinValue` | 最小值。 | `0.0` |
| `MaxValue` | 最大值。 | `100.0` |
| `ValueBrush` | 液体值区域的画笔。 | 渐变/取决于主题 |
| `TrackBrush` | 空白背景区域的画笔。 | `null` |
| `WaveAmplitude` | 波动效果的振幅。 | `5.0` |
| `WaveFrequency` | 波动效果的频率。 | `2.0` |
| `ShowPercentage` | 是否显示百分比文本。 | `true` |
| `IsWaveAnimationEnabled` | 是否对波动进行动画处理。 | `true` |

## 另请参阅

- [仪表图](/controls/data-display/charts/gauges/gauge-chart)
- [进度甜甜圈图](/controls/data-display/charts/gauges/progress-donut-chart)
