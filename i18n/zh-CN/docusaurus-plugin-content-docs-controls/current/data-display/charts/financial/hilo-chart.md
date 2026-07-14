---
id: hilo-chart
title: Hilo 图
description: 以垂直线显示每个周期的最高价和最低价，提供聚焦于价格波动性的视图，不包含开盘价或收盘价。
doc-type: reference
tags:
  - avalonia pro
---

import chartsFinancialHilo from '/img/controls/charts/charts-financial-hilo.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

Hilo 图显示给定周期的最高价和最低价。通过省略开盘价和收盘价，它提供了对总价格波动性和范围的聚焦视图。

<Image light={chartsFinancialHilo} maxWidth={400} position="center" cornerRadius="true" alt="Hilo 图，以垂直线显示日期范围内的每日价格范围，连接最高价和最低价。" />

## 何时使用
- **波动性分析**：强调最高价和最低价之间的价差。
- **支撑与阻力**：识别市场难以进一步突破的关键价格水平。
- **简化交易**：当需要比 OHLC 或蜡烛图更简洁的替代方案时。

## 代码示例

### XAML
```xml
<CartesianChart xmlns="https://github.com/avaloniaui" Name="HiloChartSample" Title="Price Range" Height="300">
    <CartesianChart.HorizontalAxis>
        <DateTimeAxis LabelFormat="MM/dd" Title="Date" />
    </CartesianChart.HorizontalAxis>
    <CartesianChart.VerticalAxis>
        <NumericalAxis LabelFormat="N0" Title="Price" />
    </CartesianChart.VerticalAxis>
    <CartesianChart.Series>
        <HiloSeries ItemsSource="{Binding HiloData}"
                             HighPath="High" LowPath="Low"
                             CategoryPath="Date"
                             Stroke="#2196F3" StrokeThickness="3" />
    </CartesianChart.Series>
</CartesianChart>
```

### 数据模型（C#）
```csharp
using System;

public record FinancialPoint(DateTime Date, double Open, double High, double Low, double Close);

public ObservableCollection<FinancialPoint> HiloData { get; } = new(GenerateFinancialData(30));

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
| `HighPath` | 最高价属性的路径。 | `null` |
| `LowPath` | 最低价属性的路径。 | `null` |
| `CategoryPath` | 日期/类别属性的路径。 | `null` |
| `Stroke` | 垂直范围线的颜色。 | 取决于主题 |
| `StrokeThickness`| 价格线的宽度。 | `2` |
