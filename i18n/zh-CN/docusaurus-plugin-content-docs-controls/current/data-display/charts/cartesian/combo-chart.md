---
id: combo-chart
title: 组合图
description: 在共享分类轴上组合多种笛卡尔系列类型，可选支持辅助 Y 轴。
doc-type: reference
tags:
  - avalonia pro
---

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

`ComboChart` 是 `CartesianChart` 的一种变体，用于在同一水平轴上组合多种系列类型，例如 `BarSeries`、`LineSeries` 和 `AreaSeries`。当图表本身需要公开一个可选的辅助 Y 轴时使用它。

## 何时使用

- **混合视觉编码**：在同一图表中组合柱状图、折线图或面积图。
- **双尺度比较**：在单独的 Y 轴上绘制辅助指标。
- **共享类别**：将多种系列类型对齐到相同的分类标签。

## 代码示例

### XAML

```xml
<ComboChart xmlns="https://github.com/avaloniaui" Title="收入与利润率"
                     Height="320"
                     ShowLegend="True"
                     ShowSecondaryAxis="True">
    <ComboChart.HorizontalAxis>
        <CategoryAxis Title="月份" />
    </ComboChart.HorizontalAxis>
    <ComboChart.VerticalAxis>
        <NumericalAxis Title="收入（千美元）" />
    </ComboChart.VerticalAxis>
    <ComboChart.SecondaryVerticalAxis>
        <NumericalAxis Title="利润率（%）" />
    </ComboChart.SecondaryVerticalAxis>
    <ComboChart.Series>
        <BarSeries Title="收入"
                            ItemsSource="{Binding MonthlyMetrics}"
                            CategoryPath="Month"
                            ValuePath="Revenue" />
        <LineSeries Title="利润率"
                             ItemsSource="{Binding MonthlyMetrics}"
                             CategoryPath="Month"
                             ValuePath="MarginPercent"
                             YAxisPosition="Secondary" />
    </ComboChart.Series>
</ComboChart>
```

### 数据模型 (C#)

```csharp
public record MonthlyMetric(string Month, double Revenue, double MarginPercent);

public ObservableCollection<MonthlyMetric> MonthlyMetrics { get; } = new()
{
    new("一月", 120, 18),
    new("二月", 145, 21),
    new("三月", 138, 19),
    new("四月", 166, 24)
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Series` | 图表中渲染的笛卡尔系列内容集合。 | 空集合 |
| `HorizontalAxis` | 混合系列使用的主水平轴。 | `null` |
| `VerticalAxis` | 混合系列使用的主垂直轴。 | `null` |
| `ShowSecondaryAxis` | 是否在右侧显示辅助 Y 轴。 | `false` |
| `SecondaryVerticalAxis` | 为分配给 `YAxisPosition.Secondary` 的系列提供的可选辅助垂直轴。 | `null` |

## 说明

- 系列按声明顺序渲染。
- 要针对辅助 Y 轴绘制系列，请将该系列的 `YAxisPosition` 设置为 `"Secondary"`。
- 有关 `BarWidth`、`MarkerShape` 或 `FillOpacity` 等属性，请参阅各系列专属页面。

## 另请参阅

- [柱状图](/controls/data-display/charts/cartesian/bar-chart)
- [折线图](/controls/data-display/charts/cartesian/line-chart)
- [面积图](/controls/data-display/charts/cartesian/area-chart)
- [坐标轴自定义](/controls/data-display/charts/shared-elements/axis-customization-chart)
