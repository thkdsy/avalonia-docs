---
id: candlestick-chart
title: 蜡烛图
description: 使用蜡烛形状的符号显示每个周期的开盘价、最高价、最低价和收盘价，用于金融市场价格分析。
doc-type: reference
tags:
  - avalonia pro
---

import chartsFinancialCandlestick from '/img/controls/charts/charts-financial-candlestick.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

蜡烛图用于描述证券、衍生品或货币在一段时间内的价格变动。每根蜡烛显示特定周期的开盘价、最高价、最低价和收盘价。

<Image light={chartsFinancialCandlestick} maxWidth={400} position="center" cornerRadius="true" alt="显示OHLC价格数据的蜡烛图，包含绿色看涨和红色看跌蜡烛，覆盖多个交易周期。" />

## 何时使用
- **市场分析**：可视化价格波动和市场情绪。
- **技术分析**：识别锤子线、十字星或吞没形态等模式。
- **高低点跟踪**：显示一个周期内价格行为的完整范围。

## 代码示例

### XAML
```xml
<FinancialChart xmlns="https://github.com/avaloniaui" Name="CandlestickChartSample" Title="Stock Price (ACME)" Height="300">
    <FinancialChart.Series>
        <CandlestickSeries ItemsSource="{Binding CandlestickData}"
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

public ObservableCollection<FinancialPoint> CandlestickData { get; } = new(GenerateFinancialData(50));

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
| `UpFill` | 当 `收盘价 >= 开盘价` 时蜡烛的填充画笔。 | `#4CAF50` |
| `DownFill` | 当 `收盘价 < 开盘价` 时蜡烛的填充画笔。 | `#F44336` |
| `UpStroke` | 当 `收盘价 >= 开盘价` 时蜡烛的轮廓画笔。 | `#4CAF50` |
| `DownStroke` | 当 `收盘价 < 开盘价` 时蜡烛的轮廓画笔。 | `#F44336` |
| `CandleWidth` | 每根蜡烛的宽度，占可用槽位的比例。 | `0.8` |

## 另请参阅

- [金融图](/controls/data-display/charts/financial/financial-chart)
- [OHLC 图](/controls/data-display/charts/financial/ohlc-chart)
