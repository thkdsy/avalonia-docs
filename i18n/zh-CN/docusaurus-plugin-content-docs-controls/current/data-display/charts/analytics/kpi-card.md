---
id: kpi-card
title: KPI 卡片
description: 以聚焦的格式展示关键业务指标，结合大数值与趋势指示器和迷你趋势线。
doc-type: reference
tags:
  - avalonia pro
---

import chartsAnalyticsKpi from '/img/controls/charts/charts-analytics-kpi.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

KPI 卡片以聚焦的格式展示关键业务指标。它们通常结合大数值、趋势指示器和用于上下文的小型趋势线。

<Image light={chartsAnalyticsKpi} maxWidth={400} position="center" cornerRadius="true" alt="KPI 卡片，展示大号指标数值，附带趋势指示器和用于上下文的迷你趋势线图。" />

## 使用场景
- **高管仪表板**：一目了然地提供主要业务目标的状态。
- **性能监控**：跟踪实时指标，如活跃用户数或服务器负载。
- **财务概览**：在报告顶部展示收入、支出和增长情况。

## 代码示例

### XAML
```xml
<WrapPanel Orientation="Horizontal" HorizontalAlignment="Center">
    <KpiCard xmlns="https://github.com/avaloniaui" Title="Revenue" Width="180" Height="160" Margin="10"
                    Value="{Binding Kpi1.Value}" Unit="{Binding Kpi1.Unit}"
                    Delta="{Binding Kpi1.Delta}" Subtitle="{Binding Kpi1.Subtitle}"
                    SparklineData="{Binding Kpi1.SparklineData}" />
    <KpiCard Title="Users" Width="180" Height="160" Margin="10"
                    Value="{Binding Kpi2.Value}" Unit="{Binding Kpi2.Unit}"
                    Delta="{Binding Kpi2.Delta}" Subtitle="{Binding Kpi2.Subtitle}"
                    SparklineData="{Binding Kpi2.SparklineData}" />
    <KpiCard Title="Bounce Rate" Width="180" Height="160" Margin="10"
                    Value="{Binding Kpi3.Value}" Unit="{Binding Kpi3.Unit}"
                    Delta="{Binding Kpi3.Delta}" Subtitle="{Binding Kpi3.Subtitle}"
                    SparklineData="{Binding Kpi3.SparklineData}"
                    NegativeBrush="{Binding Kpi3.NegativeBrush}" />
</WrapPanel>
```

### 数据模型 (C#)
```csharp
using Avalonia.Media;

public record KpiItem(
    double Value,
    string? Unit,
    double Delta,
    string Subtitle,
    double[] SparklineData,
    IBrush? NegativeBrush = null);

public KpiItem Kpi1 { get; } = new(
    124500,
    "$",
    12.5,
    "vs last month",
    [10.0, 12, 11, 14, 13, 15, 16, 14, 18, 20.0]);

public KpiItem Kpi2 { get; } = new(
    4532,
    null,
    8.2,
    "New users",
    [50.0, 55, 52, 58, 60, 65, 62, 70, 75, 80.0]);

public KpiItem Kpi3 { get; } = new(
    24.5,
    "%",
    -2.1,
    "Bounce rate",
    [30.0, 28, 29, 27, 26, 25, 24, 25, 24, 24.5],
    Brushes.Green);
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Value` | 要显示的主要数值。 | `0` |
| `Delta` | 与上一期相比的变化值（正或负）。 | `0` |
| `DeltaType` | Delta 的显示方式：`Percentage`（百分比）或 `Absolute`（绝对值）。 | `Percentage` |
| `SparklineData` | 迷你图的值数组。 | `null` |
| `Unit` | 数值的后缀（例如 "$"、"%"、"pts"）。 | `null` |
| `Subtitle` | 在 delta 下方显示的文本。 | `null` |
| `ValueFormat` | 用于主要值的数字格式字符串。 | `"N0"` |
| `PositiveBrush` | Delta 为正时使用的画笔。 | `null`（绿色，见下方说明） |
| `NegativeBrush` | Delta 为负时使用的画笔。 | `null`（红色，见下方说明） |
| `SparklineBrush` | 用于迷你趋势图的画笔。 | `null`（蓝色，见下方说明） |
| `ShowSparkline` | 是否显示迷你趋势图。 | `true` |

:::note
当 `PositiveBrush`、`NegativeBrush` 或 `SparklineBrush` 为 `null` 时，卡片会回退到内置强调色：绿色表示正 delta，红色表示负 delta，蓝色用于趋势图。
:::
