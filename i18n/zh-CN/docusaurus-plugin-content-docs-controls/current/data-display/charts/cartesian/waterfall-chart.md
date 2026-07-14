---
id: waterfall-chart
title: 瀑布图
description: 显示随着值被增加或减少的累计总和，适用于可视化顺序正向或负向变化如何影响初始值。
doc-type: reference
tags:
  - avalonia pro
---

import chartsCartesianWaterfall from '/img/controls/charts/charts-cartesian-waterfall.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

瀑布图显示随着值被增加或减少的累计总和。它有助于理解初始值如何受到一系列中间正向或负向值的影响。

<Image light={chartsCartesianWaterfall} maxWidth={400} position="center" cornerRadius="true" alt="瀑布图，浮动柱条显示对累计总和的顺序正向和负向变化。" />

## 何时使用
- **财务分析**：可视化随时间变化的损益（P&L）报表。
- **库存跟踪**：显示库存水平随增加和减少的变化。
- **流程步骤**：建模顺序变量的累积效果。

## 代码示例

### XAML
```xml
<CartesianChart xmlns="https://github.com/avaloniaui" Name="WaterfallChartSample" Title="季度损益分析" Height="300" ShowLegend="False">
                        <CartesianChart.HorizontalAxis><CategoryAxis /></CartesianChart.HorizontalAxis>
                        <CartesianChart.VerticalAxis><NumericalAxis /></CartesianChart.VerticalAxis>
                        <CartesianChart.Series>
                            <WaterfallSeries Title="损益"
                                                      ItemsSource="{Binding WaterfallData}"
                                                      CategoryPath="Category" ValuePath="Value"
                                                      TotalCategory="净利润" />
                        </CartesianChart.Series>
                    </CartesianChart>
```

### 数据模型 (C#)
```csharp
public record WaterfallFinancialPoint(string Category, double Value);

public ObservableCollection<WaterfallFinancialPoint> WaterfallData { get; } = new()
{
    new("收入", 500.0),
    new("销售成本", -200.0),
    new("市场费用", -50.0),
    new("研发费用", -80.0),
    new("管理费用", -40.0),
    new("净利润", 130.0)
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 变化的集合。 | `null` |
| `CategoryPath` | 标签/品类的路径。 | `null` |
| `ValuePath` | 变化值（正向或负向）的路径。 | `null` |
| `PositiveBrush` | 正向变化的画刷。 | 取决于主题 |
| `NegativeBrush` | 负向变化的画刷。 | 取决于主题 |
| `TotalBrush` | 总计（最终）柱条的画刷。 | 取决于主题 |
| `BarWidth` | 每个柱条的宽度，以可用品类槽位的比例表示。 | `0.7` |
| `ShowConnectorLines` | 是否绘制连续柱条之间的连接线。 | `true` |
| `TotalCategory` | 代表最终总计的品类名称。 | `null` |
