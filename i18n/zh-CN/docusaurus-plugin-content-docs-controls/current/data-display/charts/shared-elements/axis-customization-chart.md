---
id: axis-customization-chart
title: 轴自定义
description: 自定义图表轴，包括标签旋转、轴和刻度样式、网格线、多轴以及数值、分类、日期和对数类型。
doc-type: reference
tags:
  - avalonia pro
---

import chartsFeaturesGridlines from '/img/controls/charts/charts-gridlines-customization.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

Avalonia 图表允许您自定义轴外观，包括标签适配、轴线样式、刻度标记、网格线配置和多轴支持。

<Image light={chartsFeaturesGridlines} maxWidth={400} position="center" cornerRadius="true" alt="带有自定义轴和网格线的笛卡尔图表。" />

## 使用时机

- **多尺度**：在同一图表上显示两个不同的度量指标（例如温度和湿度）。
- **分类数据**：当轴表示离散组而非连续数值时。
- **时间序列分析**：自定义历史数据的日期格式和间隔。

## 代码示例

### XAML

```xml
<CartesianChart xmlns="https://github.com/avaloniaui" Name="GridLinesChart" Title="虚线主网格线" Height="300">
                        <CartesianChart.HorizontalAxis>
                            <CategoryAxis ShowGridLines="True" GridLineBrush="#BBDEFB" GridLineStrokeThickness="2">
                                <CategoryAxis.GridLineDashStyle>
                                    <DashStyle Dashes="5,5"/>
                                </CategoryAxis.GridLineDashStyle>
                            </CategoryAxis>
                        </CartesianChart.HorizontalAxis>
                        <CartesianChart.VerticalAxis>
                            <NumericalAxis ShowGridLines="True" GridLineBrush="#C8E6C9" GridLineStrokeThickness="2">
                                <NumericalAxis.GridLineDashStyle>
                                    <DashStyle Dashes="10,5"/>
                                </NumericalAxis.GridLineDashStyle>
                            </NumericalAxis>
                        </CartesianChart.VerticalAxis>
                        <CartesianChart.Series>
                            <AreaSeries Title="数据" ItemsSource="{Binding SalesData}" Fill="#7E2196F3" Stroke="#2196F3" />
                        </CartesianChart.Series>
                    </CartesianChart>
```

### 数据模型 (C#)

```csharp
public ObservableCollection<int> SalesData { get; } = new() { 35, 28, 34, 32, 40, 32, 35 };
```

## 常用轴属性（`NumericalAxis` / `CategoryAxis`）

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Title` | 轴的文本标签。 | `null` |
| `IsVisible` | 切换整个轴的可见性。 | `true` |
| `TitleFontSize` | 轴标题使用的字号。 | `14.0` |
| `TitleForeground` | 轴标题使用的画刷。 | `null` |
| `ShowLabels` | 是否绘制轴标签。 | `true` |
| `LabelFontSize` | 轴标签使用的字号。 | `12.0` |
| `LabelForeground` | 轴标签使用的画刷。 | `null` |
| `ShowAxisLine` | 是否绘制轴基线。 | `true` |
| `AxisLineStroke` | 轴基线使用的画刷。当为 `null` 时，使用图表轴画刷。 | `null` |
| `AxisLineStrokeThickness` | 轴基线的粗细。 | `1.0` |
| `AxisLineDashStyle` | 轴基线的虚线样式。 | `null` |
| `ShowTickLines` | 是否在标签位置绘制刻度标记。 | `false` |
| `TickLineLength` | 刻度标记的长度（像素）。 | `5.0` |
| `TickLineStroke` | 刻度标记使用的画刷。当为 `null` 时，使用图表轴画刷。 | `null` |
| `TickLineStrokeThickness` | 刻度标记的粗细。 | `1.0` |
| `ShowGridLines` | 显示/隐藏主网格线。 | `true` |
| `ShowMinorGridLines` | 显示/隐藏次网格线。 | `false` |
| `GridLineBrush` | 主网格线使用的画刷。 | `null` |
| `GridLineStrokeThickness` | 主网格线的粗细。 | `1.0` |
| `GridLineDashStyle` | 主网格线的虚线样式。 | `null` |
| `GridLineCap` | 主网格线的线帽样式。 | `Flat` |
| `GridLineJoin` | 主网格线的线连接样式。 | `Miter` |
| `MinorGridLineBrush` | 次网格线使用的画刷。 | `null` |
| `MinorGridLineStrokeThickness` | 次网格线的粗细。 | `0.5` |
| `MinorGridLineDashStyle` | 次网格线的虚线样式。 | `null` |
| `MinorGridLineCap` | 次网格线的线帽样式。 | `Flat` |
| `MinorGridLineJoin` | 次网格线的线连接样式。 | `Miter` |
| `LabelFormat` | 标签的格式字符串（例如 `"C0"`、`"N2"`、`"yyyy"`）。 | `null` |
| `LabelRotation` | 当 `LabelFitMode` 为 `CustomRotation` 时的自定义旋转角度。 | `0.0` |
| `MinorTickCount` | 主刻度之间的次间隔数。 | `4` |
| `LabelFitMode` | 标签无法适配时的策略。可选值包括 `None`、`Hide`、`Wrap`、`MultipleRows`、`Rotate45`、`Rotate90`、`CustomRotation` 和 `Auto`。 | `None` |

## 轴类型

| 轴 | 描述 | 支持 |
| --- | --- | --- |
| `NumericalAxis` | 用于连续数值数据。 | `Minimum`、`Maximum`、`MajorStep`、`MinorStep`、`ScaleBreaks` |
| `CategoryAxis` | 用于离散分类。 | `GapLength`、`PlotMode` |
| `DateTimeAxis` | 用于日期和时间数据。 | `Minimum`、`Maximum`、`MajorStep`、`MajorStepUnit`、`DateFormat` |
| `LogarithmicAxis` | 用于范围较大的数据。 | `Minimum`、`Maximum`、`LogBase`、`MajorStep` |

## 刻度中断

使用 `NumericalAxis.ScaleBreaks` 进行内联 XAML 刻度中断，或使用 `ScaleBreaksSource` 从视图模型绑定集合。刻度中断会跳过那些会压缩可见数据范围的范围。每个 `ScaleBreak` 定义了一个被移除的值范围以及可选的断线样式。`End <= Start` 的无效范围将被忽略，重叠或相邻的中断会在轴范围归一化之前合并。

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Start` | 跳过的轴范围中的第一个值。 | `0.0` |
| `End` | 跳过的轴范围中的最后一个值。 | `0.0` |
| `Stroke` | 绘制刻度中断标记使用的画刷。 | `null` |
| `StrokeThickness` | 刻度中断标记的粗细。 | `1.0` |

## 轴特定属性

| 轴 | 属性 | 描述 | 默认值 |
| :--- | :--- | :--- | :--- |
| `NumericalAxis` | `Minimum` | 显式最小值。当为 `null` 时，图表根据数据计算。 | `null` |
| `NumericalAxis` | `Maximum` | 显式最大值。当为 `null` 时，图表根据数据计算。 | `null` |
| `NumericalAxis` | `MajorStep` | 主刻度间隔。无效或非正值会回退为自动步长。 | `null` |
| `NumericalAxis` | `MinorStep` | 次刻度间隔。 | `null` |
| `NumericalAxis` | `ScaleBreaks` | 刻度中断的内联集合。 | 空集合 |
| `NumericalAxis` | `ScaleBreaksSource` | 设置时替代 `ScaleBreaks` 的绑定集合。 | `null` |
| `DateTimeAxis` | `Minimum` | 显式最小日期。当为 `null` 时，图表根据数据计算。 | `null` |
| `DateTimeAxis` | `Maximum` | 显式最大日期。当为 `null` 时，图表根据数据计算。 | `null` |
| `DateTimeAxis` | `MajorStep` | 主刻度间隔，与 `MajorStepUnit` 组合使用。月和年步长会四舍五入为整数。 | `1.0` |
| `DateTimeAxis` | `MajorStepUnit` | `MajorStep` 使用的单位：`Second`、`Minute`、`Hour`、`Day`、`Week`、`Month` 或 `Year`。 | `Day` |
| `DateTimeAxis` | `DateFormat` | 可选的日期标签格式字符串。 | `null` |
| `LogarithmicAxis` | `Minimum` | 显式正数最小值。当为 `null` 时，图表根据数据计算。 | `null` |
| `LogarithmicAxis` | `Maximum` | 显式正数最大值。当为 `null` 时，图表根据数据计算。 | `null` |
| `LogarithmicAxis` | `LogBase` | 轴刻度使用的对数底数。 | `10.0` |
| `LogarithmicAxis` | `MajorStep` | 主刻度乘数。当为 `null` 时，轴根据 `LogBase` 计算。 | `null` |

## 连续水平轴

`CartesianChart` 可以针对连续水平 `NumericalAxis`、`LogarithmicAxis` 或 `DateTimeAxis` 渲染支持的系列。只有当每个可见非空系列都支持连续布局，且每个水平分类值都可以转换为所选轴类型时，图表才使用连续水平布局。

支持的笛卡尔系列包括 `LineSeries`、`SplineSeries`、`StepLineSeries`、`AreaSeries`、`SplineAreaSeries`、`AreaRangeSeries`、`ScatterSeries`、`ScatterLineSeries`、`BubbleSeries` 和 `ErrorBarSeries`。`ChartTrendlineSeries` 和 `MovingAverageSeries` 在设置了 `SourceSeries` 时遵循其兼容性。

对于 `DateTimeAxis`，`CategoryPath` 值必须解析为 `DateTime` 或 `DateTimeOffset`。对于 `NumericalAxis` 和 `LogarithmicAxis`，值必须解析为有限数值。`LogarithmicAxis` 要求值大于 `0`。

如果任何可见非空系列不兼容，图表将对水平轴使用分类布局。

## 绘图带

使用 `ChartAxis.PlotBands` 在水平或垂直轴上着色范围。在分类轴上，水平轴绘图带的 `Start` 和 `End` 值使用分类索引。在连续水平轴上，它们使用轴值域。

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Start` | 着色范围的起始值。当为 `NaN` 时，带从轴起点开始。 | `NaN` |
| `End` | 着色范围的结束值。当为 `NaN` 时，带到轴终点结束。 | `NaN` |
| `Fill` | 用于填充带的画刷。 | `null` |
| `Stroke` | 带边框使用的画刷。 | `null` |
| `StrokeThickness` | 带边框的粗细。 | `0.0` |
| `Opacity` | 应用于带填充的不透明度。 | `0.3` |
| `Text` | 可选，在带内绘制的文本。 | `null` |
| `Foreground` | 绘图带文本使用的画刷。 | `null` |
| `TextFontSize` | 绘图带文本使用的字号。 | `12.0` |
| `HorizontalTextAlignment` | 带内水平文本对齐方式：`Start`、`Center` 或 `End`。 | `Center` |
| `VerticalTextAlignment` | 带内垂直文本对齐方式：`Start`、`Center` 或 `End`。 | `Center` |
| `IsVisible` | 绘图带是否渲染。 | `true` |
| `IsRepeating` | 带是否按规则轴间隔重复。 | `false` |
| `RepeatEvery` | 重复带之间的轴间隔。当启用重复但值为无效值时，设置为 `1.0`。 | `NaN` |
| `RepeatUntil` | 重复带停止的轴值。 | `NaN` |
| `RenderAboveSeries` | 是否将带渲染在图表系列之上而非之后。 | `false` |

## 另请参阅

- [组合图](/controls/data-display/charts/cartesian/combo-chart)
- [折线图](/controls/data-display/charts/cartesian/line-chart)
