---
id: heikin-ashi-chart
title: Heikin-Ashi 图
description: 修改版的蜡烛图，使用平均OHLC值来过滤市场噪音，并与标准蜡烛图相比显示平滑的趋势方向。
doc-type: reference
tags:
  - avalonia pro
---

import chartsFinancialHeikinashi from '/img/controls/charts/charts-financial-heikin.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

Heikin-Ashi 图是日本蜡烛图的一种变体。它们使用修改后的公式计算开盘价、最高价、最低价和收盘价，以过滤市场噪音并显示平滑的趋势方向。

<Image light={chartsFinancialHeikinashi} maxWidth={400} position="center" cornerRadius="true" alt="Heikin-Ashi 图，使用平均OHLC值显示平滑蜡烛线，展示市场趋势方向。" />

## 何时使用
- **趋势识别**：以较少的波动性发现市场趋势的起点和终点。
- **波段交易**：在波动市场中识别回调与反转。
- **长期分析**：平滑日常价格波动以获得更广阔的视野。

## 代码示例

### XAML
```xml
<HeikinAshiChart xmlns="https://github.com/avaloniaui" Name="HeikinAshiChartSample" Title="Smoothed Price Trend" Height="300"
                                              ItemsSource="{Binding HeikinAshiData}"
                                              HighPath="High" LowPath="Low"
                                              OpenPath="Open" ClosePath="Close"
                                              DatePath="Date" />
```

### 数据模型（C#）
```csharp
using System;

public record FinancialPoint(DateTime Date, double Open, double High, double Low, double Close);

public ObservableCollection<FinancialPoint> HeikinAshiData { get; } = new(GenerateFinancialData(40));

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
| `ItemsSource` | 价格数据的数据源。 | `null` |
| `OpenPath` | 开盘价的路径。 | `null` |
| `ClosePath` | 收盘价的路径。 | `null` |
| `HighPath` | 最高价的路径。 | `null` |
| `LowPath` | 最低价的路径。 | `null` |
| `DatePath` | 每个数据项关联的日期或时间值的路径。 | `null` |
| `BullishBrush` | 看涨蜡烛的颜色。 | `0x4C, 0xAF, 0x50` (绿色) |
| `BearishBrush` | 看跌蜡烛的颜色。 | `0xF4, 0x43, 0x36` (红色) |
| `CandleWidth` | 蜡烛宽度占可用槽位宽度的比例，范围 `0` 到 `1`。 | `0.7` |
