---
id: trendline-chart
title: 趋势线图
description: 在笛卡尔图表系列上叠加计算的趋势线，以显示线性、指数、多项式、对数和移动平均类型的模式。
doc-type: reference
tags:
  - avalonia pro
---

import chartsFeaturesTrendlines from '/img/controls/charts/charts-trendline-1.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

趋势线用于笛卡尔图表中显示数据的一般方向或模式。它们过滤噪声以突出潜在趋势。可以拟合线性、多项式、幂、指数、对数或移动平均数据。

<Image light={chartsFeaturesTrendlines} maxWidth={400} position="center" cornerRadius="true" alt="带有叠加趋势线的折线图，显示最佳拟合线。" />

## 使用时机

- **销售预测**：基于历史趋势预测未来销售。
- **数据平滑**：识别波动股票或传感器数据中的模式。
- **绩效评估**：可视化吞吐量是总体上升还是下降。

## 代码示例

### XAML

```xml
<CartesianChart xmlns="https://github.com/avaloniaui" Title="收入增长" Height="300">
    <CartesianChart.HorizontalAxis>
        <CategoryAxis Title="年份" />
    </CartesianChart.HorizontalAxis>
    <CartesianChart.VerticalAxis>
        <NumericalAxis Title="收入（百万）" Minimum="0" />
    </CartesianChart.VerticalAxis>
    <CartesianChart.Series>
        <LineSeries Title="收入"
                           ItemsSource="{Binding LinearData}"
                           CategoryPath="Category"
                           ValuePath="Value"
                           StrokeThickness="2"
                           ShowMarkers="True">
            <LineSeries.Trendlines>
                <Trendline Type="Linear"
                                  Stroke="#E53935"
                                  StrokeThickness="2"
                                  ForwardForecast="1" />
            </LineSeries.Trendlines>
        </LineSeries>
    </CartesianChart.Series>
</CartesianChart>
```

### 数据模型 (C#)

```csharp
public record ChartDataPoint(string Category, double Value);

public ObservableCollection<ChartDataPoint> LinearData { get; } = new()
{
    new("2018", 12),
    new("2019", 15),
    new("2020", 18),
    new("2021", 20),
    new("2022", 25)
};
```

## 常用属性（`Trendline`）

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Type` | 趋势线应拟合的数据类型：`Linear`、`Exponential`、`Logarithmic`、`Power`、`Polynomial` 或 `MovingAverage`。 | `Linear` |
| `Stroke` | 绘制趋势线使用的画刷。 | `Gray` |
| `StrokeThickness` | 趋势线描边的粗细。 | `2.0` |
| `StrokeDashStyle` | 趋势线描边使用的虚线样式。 | `null` |
| `StrokeLineCap` | 趋势线端点使用的线帽样式。 | `Round` |
| `StrokeLineJoin` | 趋势线段连接处使用的线连接样式。 | `Round` |
| `IsVisible` | 趋势线是否渲染。 | `true` |
| `ForwardForecast` | 向前预测的单位数。 | `0` |
| `BackwardForecast` | 向后预测的单位数。 | `0` |
| `Period` | 对于 `MovingAverage`，要平均的点数。 | `2` |
| `PolynomialOrder` | 当 `Type` 为 `Polynomial` 时的多项式次数。 | `2` |

## ChartTrendlineSeries

`ChartTrendlineSeries` 是一个独立的系列，使用回归计算渲染趋势线叠加层。与 `Trendline` 附加属性不同，它直接添加到图表的 `Series` 集合中，可以通过 `SourceSeries` 引用另一个系列，或使用自己的 `ItemsSource`。

### XAML

```xml
<CartesianChart xmlns="https://github.com/avaloniaui" Title="显式叠加系列" Height="320" ShowLegend="True">
    <CartesianChart.HorizontalAxis>
        <CategoryAxis Title="索引" />
    </CartesianChart.HorizontalAxis>
    <CartesianChart.VerticalAxis>
        <NumericalAxis Title="值" Minimum="0" />
    </CartesianChart.VerticalAxis>

    <CartesianChart.Series>
        <ScatterSeries x:Name="StandaloneTrendlineSource"
                              Title="观测数据"
                              ItemsSource="{Binding StandaloneTrendlineData}"
                              CategoryPath="Category"
                              ValuePath="Value"
                              Fill="#455A64"
                              MarkerSize="8" />
        <ChartTrendlineSeries Title="多项式叠加"
                                     SourceSeries="{Binding #StandaloneTrendlineSource}"
                                     TrendlineType="Polynomial"
                                     PolynomialOrder="3"
                                     Stroke="#D32F2F"
                                     StrokeThickness="2.5"
                                     Extend="0.15" />
    </CartesianChart.Series>
</CartesianChart>
```

### 数据模型 (C#)

```csharp
public record ChartDataPoint(string Category, double Value);

public ObservableCollection<ChartDataPoint> StandaloneTrendlineData { get; } = new()
{
    new("1", 16),
    new("2", 20),
    new("3", 27),
    new("4", 29),
    new("5", 38),
    new("6", 43),
    new("7", 47),
    new("8", 56)
};
```

### `ChartTrendlineSeries` 属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `TrendlineType` | 回归类型：`Linear`、`Polynomial`、`Exponential`、`Logarithmic`、`Power` 或 `MovingAverage`。 | `Linear` |
| `PolynomialOrder` | 当 `TrendlineType` 为 `Polynomial` 时的多项式次数。 | `2` |
| `Period` | 当 `TrendlineType` 为 `MovingAverage` 时使用的点数。 | `2` |
| `SourceSeries` | 计算趋势线所依据的系列。如果为 `null`，则直接使用 `ItemsSource`。 | `null` |
| `Extend` | 趋势线超出数据范围的延伸距离，以分数表示（例如 `0.1` = 10%）。 | `0` |

## MovingAverageSeries

`MovingAverageSeries` 为金融或时间序列数据显示移动平均叠加层。它支持简单（SMA）、指数（EMA）、加权（WMA）和三角形（TMA）移动平均计算。它可以在 `CartesianChart` 中使用，或在 `FinancialChart` 中作为遵循金融图表日期和价格坐标的叠加层使用。

### XAML

```xml
<CartesianChart xmlns="https://github.com/avaloniaui" Title="带移动平均线的价格" Height="300">
    <CartesianChart.Series>
        <LineSeries x:Name="PriceSeries"
                             ItemsSource="{Binding PriceData}"
                             CategoryPath="Date"
                             ValuePath="Close" />
        <MovingAverageSeries Title="SMA（14）"
                                      SourceSeries="{Binding #PriceSeries}"
                                      MovingAverageType="Simple"
                                      Period="14"
                                      Stroke="Orange"
                                      StrokeThickness="2" />
    </CartesianChart.Series>
</CartesianChart>
```

### 数据模型 (C#)

```csharp
using System;

public record PricePoint(DateTime Date, double Close);

public ObservableCollection<PricePoint> PriceData { get; } = new()
{
    new(new DateTime(2026, 1, 1), 100),
    new(new DateTime(2026, 1, 2), 104),
    new(new DateTime(2026, 1, 3), 101),
    new(new DateTime(2026, 1, 4), 108),
    new(new DateTime(2026, 1, 5), 112)
};
```

### `MovingAverageSeries` 属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `MovingAverageType` | 计算类型：`Simple`、`Exponential`、`Weighted` 或 `Triangular`。 | `Simple` |
| `Period` | 移动平均窗口中使用的数据点数。 | `14` |
| `SourceSeries` | 计算移动平均所依据的系列。如果为 `null`，则直接使用 `ItemsSource`。 | `null` |

## 另请参阅

- [折线图](/controls/data-display/charts/cartesian/line-chart)
- [散点图](/controls/data-display/charts/cartesian/scatter-chart)
- [轴自定义](/controls/data-display/charts/shared-elements/axis-customization-chart)
