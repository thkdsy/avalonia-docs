---
id: legend-chart
title: 图例
description: 通过标签和颜色标识图表中的数据系列，支持多种位置、方向和可选的交互式系列切换。
doc-type: reference
tags:
  - avalonia pro
---

import chartsFeaturesLegend from '/img/controls/charts/charts-legend-right.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

图例组件帮助用户识别图表中的不同数据系列。它可以放置在图表区域周围，并样式化以匹配应用程序的主题。

<Image light={chartsFeaturesLegend} maxWidth={400} position="center" cornerRadius="true" alt="带有图例面板的图表，在图表区域旁边显示带有颜色编码的系列名称。" />

## 使用时机
- **多系列图表**：当显示多个系列时必不可少。
- **交互式切换**：当用户需要通过单击图例项来显示/隐藏系列时。
- **复杂可视化**：帮助解释颜色或图案编码（例如在饼图或地图中）。

## 代码示例

### XAML
```xml
<CartesianChart xmlns="https://github.com/avaloniaui" Name="RightLegendSample"
                        IsTooltipEnabled="True"
                        Title="右对齐"
                        Height="250"
                        ShowLegend="True"
                        LegendPosition="Right"
                        LegendAlignment="Center"
                        ToggleSeriesVisibility="True">
    <CartesianChart.HorizontalAxis>
        <CategoryAxis />
    </CartesianChart.HorizontalAxis>
    <CartesianChart.VerticalAxis>
        <NumericalAxis />
    </CartesianChart.VerticalAxis>
    <CartesianChart.Series>
        <AreaSeries Title="收入"
                           ItemsSource="{Binding Data1}" />
        <AreaSeries Title="利润"
                           ItemsSource="{Binding Data3}" />
    </CartesianChart.Series>
</CartesianChart>
```

### 数据模型 (C#)
```csharp
public ObservableCollection<int> Data1 { get; } = new()
{
    10, 20, 30, 40, 50
};

public ObservableCollection<int> Data3 { get; } = new()
{
    5, 15, 10, 20, 10
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ShowLegend` | 切换图例的可见性。 | `false` |
| `LegendPosition` | `None`、`Top`、`Bottom`、`Left`、`Right` 或 `Floating`。 | `None` |
| `LegendAlignment` | `Near`、`Center` 或 `Far`。 | `Center` |
| `LegendOffset` | 应用于浮动图例的像素偏移。 | `0,0` |
| `ToggleSeriesVisibility` | 允许单击图例切换系列或分类可见性。刻度图例（如 `ShapeMap`、`CalendarHeatmapChart` 和 `WaffleChart`）会强制此值为 `false`。 | `true` |

## 图例控件属性

`ChartLegend` 是图表使用的可重用图例控件。

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Items` | 要显示的图例项集合。 | `null` |
| `Orientation` | 图例项的布局方向，`Horizontal` 或 `Vertical`。 | `Vertical` |
| `MarkerSize` | 每个图例标记的大小（像素）。 | `12.0` |
| `ItemSpacing` | 图例项之间的间距（像素）。 | `8.0` |

## 图例项模型

图例项由 `ChartLegendItem` 表示。内置系列会自动创建图例项，并选择与渲染系列样式匹配的标记形状，例如折线、带、蜡烛图、雷达、OHLC 或点数图标记。自定义系列可以重写 `CreateLegendItem` 来更改标记、源或切换行为。

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Text` | 图例项的显示文本。 | `null` |
| `Fill` | 标记的填充画刷。 | `null` |
| `Stroke` | 标记的描边画刷。 | `null` |
| `SecondaryFill` | 复合标记（如金融标记或带）的次要填充画刷。 | `null` |
| `SecondaryStroke` | 复合标记（如金融标记或带）的次要描边画刷。 | `null` |
| `MarkerShape` | 标记形状：`Rectangle`、`Circle`、`Line`、`Candlestick`、`Band`、`Radar`、`Ohlc` 或 `PointAndFigure`。 | `Rectangle` |
| `IsVisible` | 图例项表示的可见性状态。 | `true` |
| `SeriesIndex` | 关联的系列索引。 | `0` |
| `Source` | 图例项表示的系列、技术指标或图表项目。 | `null` |
| `ToggleAction` | 图例项切换时调用的可选操作。 | `null` |

## 事件

| 事件 | 描述 |
| :--- | :--- |
| `LegendItemClicked` | 图例项切换其关联源的可见性后引发。事件数据包含被单击的 `Item` 和 `IsNowVisible`。 |

## 另请参阅

- [图表导出](/controls/data-display/charts/shared-elements/export-chart)
- [标记](/controls/data-display/charts/shared-elements/markers-chart)
