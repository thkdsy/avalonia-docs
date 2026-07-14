---
id: gauge-chart
title: 仪表图
description: 在表盘样式仪表上显示单个值，带弧形填充和可选指针。
doc-type: reference
tags:
  - avalonia pro
---

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

仪表图在表盘样式的弧上渲染单个值，带可选指针和格式化值文本。

## 何时使用

- **运营仪表板**：在紧凑的表盘中显示利用率、负载或健康状态。
- **阈值监控**：强调单个当前读数而非时间历史。
- **状态卡片**：以强烈的视觉权重呈现一个指标。

## 代码示例

### XAML

```xml
<GaugeChart xmlns="https://github.com/avaloniaui" Title="CPU load"
                     Width="220"
                     Height="160"
                     Value="{Binding CpuLoad}"
                     MaxValue="100"
                     ShowNeedle="True" />
```

### 数据模型（C#）

```csharp
public double CpuLoad { get; set; } = 67;
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Value` | 仪表显示的当前值。 | `50.0` |
| `MinValue` | 刻度的最小值。 | `0.0` |
| `MaxValue` | 刻度的最大值。 | `100.0` |
| `ValueBrush` | 值弧的画笔。 | `null` |
| `TrackBrush` | 背景轨道的画笔。 | `null` |
| `NeedleBrush` | 指针的画笔。 | `null` |
| `ShowNeedle` | 是否绘制指针。 | `true` |
| `StartAngle` | 仪表弧的起始角度（度）。 | `135.0` |
| `SweepAngle` | 仪表弧的扫掠角度（度）。 | `270.0` |
| `TrackThickness` | 轨道弧的粗细。 | `20.0` |
| `ShowValue` | 是否显示格式化的值文本。 | `true` |
| `ValueFormat` | 值文本的格式字符串。 | `"{0:F0}"` |

## 另请参阅

- [圆形仪表](/controls/data-display/charts/gauges/circular-gauge-chart)
- [进度甜甜圈图](/controls/data-display/charts/gauges/progress-donut-chart)
