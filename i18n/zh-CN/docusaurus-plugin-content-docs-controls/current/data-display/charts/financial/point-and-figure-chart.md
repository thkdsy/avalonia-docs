---
id: point-and-figure-chart
title: 点数图
description: 使用 X 和 O 的列来表示价格变动，过滤时间因素并专注于趋势反转。
doc-type: reference
tags:
  - avalonia pro
---

import chartsFinancialPointandfigure from '/img/controls/charts/charts-financial-point-figure.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

点数图（P&F）使用 X 和 O 的列来表示上涨和下跌的价格。它们过滤掉时间和小幅价格变动，仅专注于纯粹的价格运动和趋势反转。

<Image light={chartsFinancialPointandfigure} maxWidth={400} position="center" cornerRadius="true" alt="点数图，使用 X 标记列表示上涨价格，O 标记列表示下跌价格，过滤时间因素。" />

## 何时使用
- **长期趋势**：可视化宏观经济或多年的市场变化。
- **支撑/阻力**：识别清晰的供需区域。
- **价格目标**：使用传统的 P&F 计数方法进行价格预测。

## 代码示例

### XAML
```xml
<FinancialChart xmlns="https://github.com/avaloniaui" Name="PointAndFigureChartSample" Title="Trend Analysis" Height="300">
    <FinancialChart.Series>
        <PointAndFigureSeries ItemsSource="{Binding PointAndFigureData}"
                                       HighPath="High"
                                       LowPath="Low"
                                       OpenPath="Open"
                                       ClosePath="Close"
                                       DatePath="Date"
                                       BoxSize="2.0"
                                       ReversalAmount="3" />
    </FinancialChart.Series>
</FinancialChart>
```

### 数据模型（C#）
```csharp
using System;

public record FinancialPoint(DateTime Date, double Open, double High, double Low, double Close);

public ObservableCollection<FinancialPoint> PointAndFigureData { get; } = new(GenerateFinancialData(100));

private static IEnumerable<FinancialPoint> GenerateFinancialData(int count)
{
    var date = DateTime.Today.AddDays(-count);
    var price = 100.0;
    var random = new Random(42);

    for (var i = 0; i < count; i++)
    {
        var open = price;
        var close = open + (random.NextDouble() - 0.5) * 5;
        var high = Math.Max(open, close) + random.NextDouble() * 2;
        var low = Math.Min(open, close) - random.NextDouble() * 2;

        yield return new FinancialPoint(date.AddDays(i), open, high, low, close);
        price = close;
    }
}
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 价格数据的集合。 | `null` |
| `DatePath` | 沿水平轴使用的日期或时间值的路径。值可以是 `DateTime`、`DateTimeOffset` 或可解析的日期字符串。 | `null` |
| `HighPath` | 用于构建上涨列的最高价路径。 | `null` |
| `LowPath` | 用于构建下跌列的最低价路径。 | `null` |
| `ClosePath` | 用于起始水平和代表性数值的收盘价路径。未设置时，使用 `ValuePath`。 | `null` |
| `BoxSize` | 一个 X 或 O 代表的价格变动。必须为有限值且大于 `0`；极小的值会在内部增加以避免生成过多的盒子。 | `1.0` |
| `ReversalAmount` | 开始新列所需的盒子数量。低于 `1` 的值视为 `1`。 | `3` |
| `XBrush` | X 标记的画笔。 | `Green` |
| `OBrush` | O 标记的画笔。 | `Red` |

点数图渲染仅使用具有有限 `High`、`Low` 和 `Close` 值的源数据点。
