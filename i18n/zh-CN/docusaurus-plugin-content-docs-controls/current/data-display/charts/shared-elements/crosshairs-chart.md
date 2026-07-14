---
id: crosshairs-chart
title: 十字准线
description: 跟随光标在图表上移动的交互式参考线，支持数据点与轴标签精确对齐，实现高精度读取。
doc-type: reference
tags:
  - avalonia pro
---

import chartsFeaturesCrosshairs from '/img/controls/charts/charts-custom-crosshairs.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

十字准线是跟随用户光标移动的交互式参考线。它们帮助用户在高精度图表中将数据点与轴标签对齐。

十字准线标签使用配置的轴格式。连续水平轴、对数轴、刻度中断和次要垂直轴都会反映在显示的坐标标签中。

<Image light={chartsFeaturesCrosshairs} maxWidth={400} position="center" cornerRadius="true" alt="带有交互式十字准线参考线的折线图，跟随光标以将数据点与轴坐标对齐。" />

## 使用时机
- **金融图表**：在蜡烛图（K线图）上精确定位价格和时间。
- **工程数据**：在高密度折线图上测量值。
- **科学图表**：将特定的峰值或谷值与坐标对齐。

## 代码示例

### XAML
```xml
<CartesianChart xmlns="https://github.com/avaloniaui" Name="CustomCrosshairChart" Height="300"
                                             CrosshairMode="Both"
                                             ShowCrosshairLabels="True"
                                             CrosshairStroke="DodgerBlue"
                                             CrosshairStrokeThickness="3"
                                             CrosshairLabelBackground="DodgerBlue"
                                             CrosshairLabelForeground="White"
                                             CrosshairLabelFontSize="14">
                        <CartesianChart.CrosshairDashStyle>
                            <DashStyle Dashes="10, 5" Offset="0" />
                        </CartesianChart.CrosshairDashStyle>
                        <CartesianChart.Series>
                            <LineSeries Title="Series 2" ItemsSource="{Binding CrosshairData}"
                                                 Stroke="DodgerBlue" StrokeThickness="2"
                                                 MarkerSize="6" MarkerFill="DodgerBlue"
                                                 MarkerShape="Square"  ShowMarkers="True"/>
                        </CartesianChart.Series>
                        <CartesianChart.HorizontalAxis>
                            <CategoryAxis Title="Category" />
                        </CartesianChart.HorizontalAxis>
                        <CartesianChart.VerticalAxis>
                            <NumericalAxis Title="Value" />
                        </CartesianChart.VerticalAxis>
                    </CartesianChart>
```

### 数据模型 (C#)
```csharp
public ObservableCollection<double> CrosshairData { get; } = new()
{
    18.5, 21.2, 24.8, 22.1, 19.7, 23.4, 26.1
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `CrosshairMode` | `None`、`Vertical`、`Horizontal` 或 `Both`。 | `None` |
| `ShowCrosshairLabels` | 切换轴上数值标签的显示。 | `true` |
| `CrosshairStroke` | 参考线使用的画刷。当为 `null` 时，使用暗灰色画刷。 | `null` |
| `CrosshairStrokeThickness` | 参考线的宽度。 | `1.0` |
| `CrosshairDashStyle` | 参考线的虚线样式。当为 `null` 时，图表使用 `4,4` 虚线模式。 | `null` |
| `CrosshairLabelBackground` | 十字准线标签背景使用的画刷。当为 `null` 时，使用半透明深色画刷。 | `null` |
| `CrosshairLabelForeground` | 十字准线标签文本使用的画刷。当为 `null` 时，使用白色。 | `null` |
| `CrosshairLabelFontSize` | 十字准线标签的字号。 | `10.0` |
