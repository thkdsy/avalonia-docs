---
id: ohlc-chart
title: OHLC 图
description: 使用带水平刻度的垂直线显示开盘价、最高价、最低价和收盘价，是价格数据的标准专业可视化方式。
doc-type: reference
tags:
  - avalonia pro
---

import chartsFinancialOhlc from '/img/controls/charts/charts-financial-ohlc.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

OHLC 图显示给定周期的开盘价、最高价、最低价和收盘价。它们与蜡烛图类似，但使用带水平刻度的垂直线来表示价格范围和开盘价/收盘价。

<Image light={chartsFinancialOhlc} maxWidth={400} position="center" cornerRadius="true" alt="OHLC 图，每个周期使用带水平刻度标记的垂直线显示开盘价、最高价、最低价和收盘价。" />

## 何时使用
- **交易分析**：可视化价格行为，没有蜡烛图实体带来的"重量感"。
- **市场趋势**：发现特定时间间隔内的趋势和价格范围。
- **商品/股票跟踪**：价格数据的标准专业可视化方式。

## 代码示例

### XAML
```xml
<FinancialChart xmlns="https://github.com/avaloniaui" Name="OhlcChartSample" Title="Commodity Futures" Height="300">
    <FinancialChart.Series>
        <OhlcSeries ItemsSource="{Binding OhlcData}"
                             HighPath="High" LowPath="Low"
                             OpenPath="Open" ClosePath="Close"
                             DatePath="Date" />
    </FinancialChart.Series>
</FinancialChart>
```

### 数据模型（C#）
```csharp
using System;

public record FinancialPoint(DateTime Date, double Open, double High, double Low, double Close);

public ObservableCollection<FinancialPoint> OhlcData { get; } = new(GenerateFinancialData(30));

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
| `ItemsSource` | 金融数据点的集合。 | `null` |
| `OpenPath` | '开盘价'属性的路径。 | `null` |
| `HighPath` | '最高价'属性的路径。 | `null` |
| `LowPath` | '最低价'属性的路径。 | `null` |
| `ClosePath` | '收盘价'属性的路径。 | `null` |
| `DatePath` | 沿水平轴使用的日期或时间值的路径。值可以是 `DateTime`、`DateTimeOffset` 或可解析的日期字符串。 | `null` |
| `UpStroke` | 当 `收盘价 >= 开盘价` 时柱线的轮廓画笔。 | `#4CAF50` |
| `DownStroke` | 当 `收盘价 < 开盘价` 时柱线的轮廓画笔。 | `#F44336` |
| `StrokeThickness` | 线条的粗细。 | `2.0` |
| `TickWidth` | 开盘价和收盘价刻度标记的宽度（以像素为单位）。 | `6.0` |

## 另请参阅

- [金融图](/controls/data-display/charts/financial/financial-chart)
- [蜡烛图](/controls/data-display/charts/financial/candlestick-chart)
