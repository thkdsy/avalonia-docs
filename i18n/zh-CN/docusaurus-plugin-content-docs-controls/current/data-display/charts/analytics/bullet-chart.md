---
id: bullet-chart
title: 子弹图
description: 以紧凑的横向或纵向仪表形式，展示单一绩效指标与目标值及定性范围的对比。
doc-type: reference
tags:
  - avalonia pro
---

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

子弹图以紧凑布局将主要值与目标值以及一个或多个定性范围进行对比。

## 使用场景

- **目标跟踪**：在仪表板卡片中对比实际值与目标值。
- **紧凑摘要**：当只需要一个度量值和目标值时，替代完整的柱状图。
- **阈值范围**：在主数值后方显示较差、一般和良好等区间带。

## 代码示例

### XAML

```xml
<BulletChart xmlns="https://github.com/avaloniaui" Title="Revenue attainment"
                      Width="320"
                      Height="80"
                      Value="{Binding ActualRevenue}"
                      Target="{Binding RevenueTarget}"
                      Ranges="{Binding RevenueBands}" />
```

### 数据模型 (C#)

```csharp
public double ActualRevenue { get; set; } = 72;
public double RevenueTarget { get; set; } = 85;
public double[] RevenueBands { get; } = [30, 60, 90, 100];
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Value` | 由条形图显示的主要值。 | `75.0` |
| `Target` | 目标标记值。 | `85.0` |
| `MinValue` | 标尺最小值。 | `0.0` |
| `MaxValue` | 标尺最大值。 | `100.0` |
| `Ranges` | 可选的定性范围边界集合。 | `null` |
| `Orientation` | 图表方向，`Horizontal`（横向）或 `Vertical`（纵向）。 | `Horizontal` |
| `ValueBrush` | 主数值条的画笔。 | `null` |
| `TargetBrush` | 目标标记的画笔。 | `null` |

## 另请参阅

- [KPI 卡片](/controls/data-display/charts/analytics/kpi-card)
- [线性仪表图](/controls/data-display/charts/gauges/linear-gauge-chart)
