---
id: financial-chart
title: 金融图
description: 金融系列（如蜡烛图和OHLC图）的容器图表，提供共享轴和价格网格渲染。
doc-type: reference
tags:
  - avalonia pro
---

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

`FinancialChart` 是金融系列（如 `CandlestickSeries` 和 `OhlcSeries`）的宿主控件。它提供这些系列所使用的共享价格网格、轴和水平类别布局。兼容的叠加系列（如 `MovingAverageSeries`）可以在相同的日期和价格坐标空间中渲染。

## 何时使用

- **金融系列宿主**：在专用的图表容器中组合一个或多个基于价格的系列。
- **交易视图**：跨不同系列类型重用共享的金融轴和价格网格。
- **系列比较**：针对同一水平轴叠加兼容的金融系列。

## 代码示例

### XAML

```xml
<FinancialChart xmlns="https://github.com/avaloniaui" Title="Commodity futures" Height="300">
    <FinancialChart.Series>
        <OhlcSeries ItemsSource="{Binding OhlcData}"
                             DatePath="Date"
                             OpenPath="Open"
                             HighPath="High"
                             LowPath="Low"
                             ClosePath="Close" />
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

金融系列的日期值可以是 `DateTime`、`DateTimeOffset` 或可以解析为日期的字符串。

当 `HorizontalAxis` 是 `DateTimeAxis` 时，其最小值、最大值和标签格式将应用于金融日期域。金融图保持可见交易周期的等间距槽位宽度，而不是使用经过的日历时间作为水平距离。

自定义叠加系列可以通过实现 `IFinancialChartOverlaySeries` 在金融图坐标空间中渲染。当叠加值应影响价格轴边界时，实现 `IFinancialChartOverlayBoundsProvider`。`FinancialOverlayRenderContext` 提供金融数据、日期到索引的映射、可见价格边界、已解析的画笔以及 `TryDateToX`、`ValueToY` 和 `TryValueToPoint` 等辅助方法。

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Series` | 在图表中渲染的金融系列和兼容叠加系列的内容集合。 | 空集合 |
| `HorizontalAxis` | 用于日期或类别位置的水平轴。 | `null` |
| `VerticalAxis` | 用于价格值的垂直轴。 | `null` |
| `GridLineBrush` | 当轴未设置自己的网格线画笔时使用的默认网格线画笔。 | `null` |
| `AxisBrush` | 用于轴线和刻度的画笔。 | `null` |
| `PlotAreaBackground` | 绘图区域的可选背景画笔。 | `null` |
| `IsHighlightEnabled` | 当系列未直接启用时，启用金融数据点的悬停高亮。 | `false` |

## 另请参阅

- [蜡烛图](/controls/data-display/charts/financial/candlestick-chart)
- [OHLC 图](/controls/data-display/charts/financial/ohlc-chart)
- [趋势线图](/controls/data-display/charts/shared-elements/trendline-chart)
