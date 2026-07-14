---
id: technical-indicators-chart
title: 图表技术指标
description: 在笛卡尔图表上叠加 SMA、EMA、WMA、布林带和其他技术指标，用于分析金融或时间序列数据的趋势和波动性。
doc-type: reference
tags:
  - avalonia pro
---

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

技术指标是添加到 `CartesianChart` 的分析叠加层，用于帮助识别数据中的趋势、动量和波动性。它们从目标系列计算派生值，并直接在图表上渲染结果。指标被添加到 `CartesianChart` 的 `TechnicalIndicators` 集合中。

指标遵循目标系列的轴上下文。这包括当目标系列使用 `YAxisPosition="Secondary"` 时的连续水平轴和次级 Y 轴缩放。

## 何时使用
- **趋势分析**：使用移动平均线平滑嘈杂的价格或传感器数据，如简单移动平均线（SMA）、指数移动平均线（EMA）或加权移动平均线（WMA）。
- **波动性评估**：使用布林带可视化移动平均线周围的标准差带。
- **金融图表**：向蜡烛图或 OHLC 图添加标准技术分析叠加层。

## 代码示例

### XAML
```xml
<CartesianChart xmlns="https://github.com/avaloniaui" Title="Stock Analysis" Height="400">
    <CartesianChart.Series>
        <CandlestickSeries x:Name="PriceSeries"
                                     ItemsSource="{Binding StockData}"
                                     DatePath="Date"
                                     OpenPath="Open" HighPath="High"
                                     LowPath="Low" ClosePath="Close" />
    </CartesianChart.Series>
    <CartesianChart.TechnicalIndicators>
        <SMAIndicator TargetSeries="{Binding #PriceSeries}"
                               Period="20" Stroke="Orange" StrokeThickness="2" />
        <BollingerBandsIndicator TargetSeries="{Binding #PriceSeries}"
                                          Period="20" StandardDeviations="2"
                                          Stroke="Blue" StrokeThickness="1"
                                          UpperBandStroke="Gray" LowerBandStroke="Gray">
            <BollingerBandsIndicator.BandFill>
                <SolidColorBrush Color="#33808080" />
            </BollingerBandsIndicator.BandFill>
        </BollingerBandsIndicator>
    </CartesianChart.TechnicalIndicators>
</CartesianChart>
```

### 数据模型（C#）
```csharp
public record StockPoint(string Date, double Open, double High, double Low, double Close);

public ObservableCollection<StockPoint> StockData { get; } = new()
{
    new("Jan", 100, 110, 95, 105),
    new("Feb", 105, 115, 100, 112),
    new("Mar", 112, 120, 108, 118),
    // ...
};
```

## 常用属性（`ChartTechnicalIndicator`）

以下属性为所有技术指标类型所共用。

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `TargetSeries` | 计算此指标的 `CartesianSeries`。 | `null` |
| `IsVisible` | 指标是否可见。 | `true` |
| `Stroke` | 用于指标主线的画笔。 | `Blue` |
| `StrokeThickness` | 指标线的粗细。 | `2.0` |
| `StrokeDashStyle` | 指标线的虚线样式。类型为 `DashStyle?`。 | `null` |
| `StrokeLineCap` | 指标线的线帽样式。 | `Round` |
| `StrokeLineJoin` | 指标线的线连接样式。 | `Round` |
| `Title` | 在图例和工具提示中显示的名称。 | 因指标而异 |

## 常用属性（`SMAIndicator`）

简单移动平均线（SMA）计算前 *n* 个数据点的未加权平均值。它是最基本的平滑技术。

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Period` | 移动平均窗口中使用的数据点数量。 | `14` |
| `Title` | 图例和工具提示标题。 | `"SMA"` |

### XAML

```xml
<CartesianChart xmlns="https://github.com/avaloniaui" Height="250">
    <CartesianChart.Series>
        <LineSeries x:Name="PriceSeries"
                           ItemsSource="{Binding StockData}"
                           ValuePath="Close" />
    </CartesianChart.Series>
    <CartesianChart.TechnicalIndicators>
        <SMAIndicator TargetSeries="{Binding #PriceSeries}"
                             Period="20" Stroke="Orange" StrokeThickness="2" />
    </CartesianChart.TechnicalIndicators>
</CartesianChart>
```

## 常用属性（`EMAIndicator`）

指数移动平均线（EMA）赋予近期数据点更高的权重，使其比 SMA 对新信息更敏感。

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Period` | EMA 计算中使用的数据点数量。 | `14` |
| `Title` | 图例和工具提示标题。 | `"EMA"` |

### XAML

```xml
<CartesianChart xmlns="https://github.com/avaloniaui" Height="250">
    <CartesianChart.Series>
        <LineSeries x:Name="PriceSeries"
                           ItemsSource="{Binding StockData}"
                           ValuePath="Close" />
    </CartesianChart.Series>
    <CartesianChart.TechnicalIndicators>
        <EMAIndicator TargetSeries="{Binding #PriceSeries}"
                             Period="12" Stroke="Green" StrokeThickness="2" />
    </CartesianChart.TechnicalIndicators>
</CartesianChart>
```

## 常用属性（`WMAIndicator`）

加权移动平均线（WMA）为数据点分配线性递增的权重，最近的数据点获得最高的权重。

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Period` | WMA 计算中使用的数据点数量。 | `14` |
| `Title` | 图例和工具提示标题。 | `"WMA"` |

### XAML

```xml
<CartesianChart xmlns="https://github.com/avaloniaui" Height="250">
    <CartesianChart.Series>
        <LineSeries x:Name="PriceSeries"
                           ItemsSource="{Binding StockData}"
                           ValuePath="Close" />
    </CartesianChart.Series>
    <CartesianChart.TechnicalIndicators>
        <WMAIndicator TargetSeries="{Binding #PriceSeries}"
                             Period="14" Stroke="Purple" StrokeThickness="2" />
    </CartesianChart.TechnicalIndicators>
</CartesianChart>
```

## 常用属性（`BollingerBandsIndicator`）

布林带由一条简单移动平均线（中轨）和两条标准差带（上轨和下轨）组成。它们用于衡量波动性并识别超买或超卖状态。

在图例中显示时，`BollingerBandsIndicator` 为中间的 SMA 创建一个线条项，为上下轨创建一个带区项。

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Period` | 移动平均计算的数据点数量。 | `20` |
| `StandardDeviations` | 上下轨的标准差数量。 | `2.0` |
| `UpperBandStroke` | 上轨线的画笔。 | `Gray` |
| `LowerBandStroke` | 下轨线的画笔。 | `Gray` |
| `BandFill` | 用于填充上下轨之间区域的画笔。 | 半透明 `Gray` |
| `Title` | 图例和工具提示标题。 | `"Bollinger Bands"` |

### XAML
```xml
<CartesianChart xmlns="https://github.com/avaloniaui" Height="250">
    <CartesianChart.Series>
        <LineSeries x:Name="PriceSeries"
                           ItemsSource="{Binding StockData}"
                           ValuePath="Close" />
    </CartesianChart.Series>
    <CartesianChart.TechnicalIndicators>
        <BollingerBandsIndicator TargetSeries="{Binding #PriceSeries}"
                                        Period="20" StandardDeviations="2"
                                        Stroke="Blue" StrokeThickness="1"
                                        UpperBandStroke="LightGray"
                                        LowerBandStroke="LightGray">
            <BollingerBandsIndicator.BandFill>
                <SolidColorBrush Color="#20808080" />
            </BollingerBandsIndicator.BandFill>
        </BollingerBandsIndicator>
    </CartesianChart.TechnicalIndicators>
</CartesianChart>
```

## 另请参阅

- [趋势线图](/controls/data-display/charts/shared-elements/trendline-chart)
- [折线图](/controls/data-display/charts/cartesian/line-chart)
- [轴自定义](/controls/data-display/charts/shared-elements/axis-customization-chart)
