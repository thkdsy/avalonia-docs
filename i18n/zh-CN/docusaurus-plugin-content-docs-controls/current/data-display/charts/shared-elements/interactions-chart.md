---
id: interactions-chart
title: 交互
description: 使用户能够在密集或交互式图表数据中进行缩放、平移、选择、悬停高亮和通过轨迹球参考线检查数值。
doc-type: reference
tags:
  - avalonia pro
---

import chartsFeaturesZoom from '/img/controls/charts/charts-zoom.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

图表交互允许用户通过缩放、平移、选择、悬停高亮和轨迹球检查来动态探索数据。

<Image light={chartsFeaturesZoom} maxWidth={400} position="center" cornerRadius="true" alt="带有交互式缩放和平移控件的图表，使用户能够聚焦于密集数据集的特定区域。" />

## 使用时机
- **大数据可视化**：探索具有数千个数据点的折线图。
- **深入分析**：缩放到特定时间窗口进行详细研究。
- **交互式报告**：让用户能够自主聚焦于关注的区域。
- **悬停检查**：淡化非悬停项目，使活动数据点或段更容易识别。

## 代码示例

### XAML
```xml
<StackPanel Spacing="15">
    <WrapPanel Orientation="Horizontal">
        <Button Content="后退" Margin="0,0,10,10" />
        <Button Content="重置" Margin="0,0,10,10" />
        <TextBlock Text="缩放：X=100%, Y=100%"
                   VerticalAlignment="Center"
                   Margin="0,0,20,10" />
        <CheckBox Name="ShowRangeSelectorCheckBox"
                  Content="显示范围选择器"
                  IsChecked="True"
                  VerticalAlignment="Center"
                  Margin="0,0,0,10" />
    </WrapPanel>

    <CartesianChart xmlns="https://github.com/avaloniaui" Name="ChartXY"
                           Height="350"
                           ZoomMode="XY"
                           IsZoomEnabled="True"
                           IsPanEnabled="True"
                           ShowRangeSelector="{Binding #ShowRangeSelectorCheckBox.IsChecked}">
        <CartesianChart.HorizontalAxis>
            <DateTimeAxis LabelFormat="MMM dd" ShowGridLines="True" />
        </CartesianChart.HorizontalAxis>
        <CartesianChart.VerticalAxis>
            <NumericalAxis LabelFormat="N0" ShowGridLines="True" />
        </CartesianChart.VerticalAxis>
        <CartesianChart.Series>
            <LineSeries ItemsSource="{Binding ZoomData}"
                               CategoryPath="Date"
                               ValuePath="Value"
                               Stroke="#FF9800"
                               StrokeThickness="2" />
        </CartesianChart.Series>
    </CartesianChart>

    <Border BorderBrush="{DynamicResource CardBorderBrush}"
            BorderThickness="1"
            CornerRadius="4"
            Background="{DynamicResource CardBackgroundBrush}">
        <ScrollViewer Height="100">
            <StackPanel Spacing="2" Margin="5">
                <TextBlock Text="交互事件："
                           FontWeight="Bold"
                           FontSize="12"
                           Foreground="{DynamicResource AccentBrush}" />
            </StackPanel>
        </ScrollViewer>
    </Border>
</StackPanel>
```

### 数据模型 (C#)
```csharp
using System;

public class DateTimePoint
{
    public DateTime Date { get; set; }
    public double Value { get; set; }
}

public ObservableCollection<DateTimePoint> ZoomData { get; } = CreateZoomData();

private static ObservableCollection<DateTimePoint> CreateZoomData()
{
    var data = new ObservableCollection<DateTimePoint>();
    var date = new DateTime(2023, 1, 1);
    var random = new Random(42);
    var value = 100.0;

    for (var i = 0; i < 365; i++)
    {
        value += random.NextDouble() * 10 - 4.5;
        value = Math.Max(50, Math.Min(200, value));
        data.Add(new DateTimePoint { Date = date, Value = value });
        date = date.AddDays(1);
    }

    return data;
}
```

## 常用属性

### 缩放和平移

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `IsZoomEnabled` | 启用缩放到图表中的功能。 | `false` |
| `IsPanEnabled` | 启用平移（滚动）图表视图的功能。 | `false` |
| `ZoomMode` | `X`、`Y` 或 `XY` 轴缩放。 | `XY` |
| `ZoomSensitivity` | 非负的鼠标滚轮缩放灵敏度。`0` 禁用滚轮缩放；无效值强制为 `0`。 | `0.1` |
| `ShowRangeSelector` | 显示用于缩放的范围选择器控件。 | `true` |
| `ZoomHistoryLimit` | 为缩放回退导航保留的视口状态最大数量。设置为 `0` 以保留所有推送的状态。 | `20` |
| `CanGoBackZoom` | 只读状态，指示 `GoBackZoom()` 是否能恢复上一个视口。 | `false` |

### 范围选择器

当 `IsZoomEnabled`、`ShowRangeSelector` 和当前 `ZoomMode` 需要时，`CartesianChart` 会创建嵌入的 `ChartRangeSelector` 控件。当需要单独的范围选择表面时，直接使用 `ChartRangeSelector`。

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Orientation` | 选择器方向，`Horizontal` 或 `Vertical`。 | `Horizontal` |
| `Minimum` | 选择器表示的最小数据值。 | `0.0` |
| `Maximum` | 选择器表示的最大数据值。 | `100.0` |
| `SelectedMinimum` | 所选范围的起始值。此属性默认双向绑定。 | `0.0` |
| `SelectedMaximum` | 所选范围的结束值。此属性默认双向绑定。 | `100.0` |
| `GripSize` | 可拖动范围手柄的大小（像素）。 | `24.0` |
| `PlotAreaOffset` | 用于将选择器轨道与图表绘图区域对齐的偏移量。 | `0.0` |
| `SmallChange` | 键盘导航增量。 | `1.0` |
| `KeyboardStepRatio` | 可选，在设置了比例转换器的情况下，归一化选择器空间中的键盘移动步长。`1.0` 表示完整轨道长度。未设置时，键盘移动使用 `SmallChange`。 | `null` |
| `ValueToRatio` | 可选，从数据值到归一化选择器位置的转换器。用于非线性轴和刻度中断。 | `null` |
| `RatioToValue` | 可选，从归一化选择器位置到数据值的转换器。 | `null` |

| 事件 | 描述 |
| :--- | :--- |
| `RangeDragStarted` | 用户开始拖动选择器滑块或手柄时引发。 |
| `RangeDragCompleted` | 活动的范围拖动完成时引发。 |

### 悬停高亮

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `IsHighlightEnabled` | 启用悬停高亮。在系列上，悬停数据点会使该系列中的其他项目变暗。在支持的独立图表上，悬停一个段或单元格会使该图表中的其他项目变暗。 | `false` |

支持的独立图表包括 `BubbleCloud`、`PackedBubbleChart`、`NightingaleRoseChart`、`RadialBarChart`、`SemiDonutChart`、`SunburstChart`、比较图、漏斗图、网格图、`TreeMapChart`、`FinancialChart`、`PolarAreaChart`、`PolarChart` 和 `RadarChart`。

### 轨迹球

`CartesianChart` 可以在指针在绘图区域上移动时显示轨迹球参考线和数值工具提示。

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `TrackballMode` | 轨迹球线模式：`None`、`Vertical` 或 `Horizontal`。 | `None` |
| `TrackballDisplayMode` | 工具提示显示模式：`FloatAllPoints` 或 `GroupAllPoints`。 | `FloatAllPoints` |
| `TrackballLineStroke` | 轨迹球参考线使用的画刷。当为 `null` 时，使用 `DimGray`。 | `null` |
| `TrackballLineStrokeThickness` | 轨迹球参考线的粗细。 | `1.0` |

### 选择

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `IsSelectionEnabled` | 启用对支持的系列或图表项目的指针选择。 | `false` |
| `SelectionMode` | 选择行为，例如 `None`、`Single`、`SingleDeselect` 或 `Multiple`。 | `SingleDeselect` |
| `SelectedIndex` | 主要选中项目的双向索引，未选择任何内容时为 `-1`。 | `-1` |
| `SelectedIndexes` | 多选场景中已选索引的只读快照。 | 空集合 |
| `SelectionBrush` | 选中项目使用的画刷。 | `#314A6E` |
| `SelectionStroke` | 选中项目的可选轮廓画刷。 | `null` |
| `SelectionStrokeThickness` | 选中项目的轮廓粗细。 | `2.0` |

选择 API 可在可选择的图表控件和可选择系列上使用。`SelectionChanging` 在选择提交前引发，可通过更新事件数据来取消或编辑。`SelectionChanged` 在应用的选择更改后引发。

### 事件和方法

| 成员 | 描述 |
| :--- | :--- |
| `DataPointClicked` | 点击数据点时引发。事件数据在可用时包含 `Series`、`DataPointIndex`、`Category`、`Value` 和原始 `DataItem`。 |
| `DataPointHovered` | 指针经过数据点或离开所有数据点后，在悬停去抖延迟后引发。此事件不需要 `IsHighlightEnabled`；该属性仅控制高亮视觉效果。事件数据包含 `Source` 和 `DataPointIndex`。 |
| `GoBackZoom()` | 恢复上一个缩放视口，并在应用了历史记录条目时返回 `true`。 |
| `ResetZoom()` | 清除活动视口和缩放历史。 |
| `ClearSelection()` | 清除可选择的图表、系列或图层上的当前选择。 |
| `TrySelectDataPoint(index)` | 尝试按索引在可选择图表或系列上选择一个数据点。 |
| `IsDataPointSelected(index)` | 返回数据点索引当前是否被选中。 |
| `TrySelectItem(item)` | 尝试在可选择项目的控件（如 `ShapeLayer`）上选择一个数据项。 |
| `IsItemSelected(item)` | 返回数据项在可选择项目的控件上是否被选中。 |
| `SelectionChanging` | 在选择更改前引发。事件数据包含可编辑的 `NewSelection` 和 `NewIndexes`，先前的 `OldSelection` 和 `OldIndexes`，以及 `Cancel`。 |
| `SelectionChanged` | 在选择更改后引发。事件数据包含 `NewSelection`、`OldSelection`、`NewIndexes` 和 `OldIndexes` 的快照。 |
| `ZoomChanged` | 缩放视口更改时引发。事件数据包含 `Axis`、先前和当前的缩放因子、先前和当前的缩放位置以及可见视口边界。 |
| `ZoomReset` | 缩放重置后引发。事件数据包含先前的视口边界和先前的缩放因子。 |
| `SeriesAdded` | `CartesianChart` 在系列添加到其 `Series` 集合后引发。 |
| `SeriesRemoved` | `CartesianChart` 在系列从其 `Series` 集合中移除后引发。 |

## 交互控件
- **鼠标滚轮**：在光标位置放大/缩小。
- **Ctrl + 拖动**：在图表区域上平移。
- **Shift + 拖动**：绘制矩形以缩放到特定区域。
- **悬停**：当 `IsHighlightEnabled` 为 `true` 时，高亮活动数据点或段。
- **双击**：将缩放和平移重置为默认视图。

图表还支持捏合缩放、鼠标滚轮缩放和选择缩放。

## 另请参阅

- [工具提示](/controls/data-display/charts/shared-elements/tooltip-chart)
- [十字准线](/controls/data-display/charts/shared-elements/crosshairs-chart)
- [轴自定义](/controls/data-display/charts/shared-elements/axis-customization-chart)
