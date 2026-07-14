---
id: sparkline-chart
title: 迷你图
description: 不带坐标轴的小型极简图表，用于在紧凑空间中显示数据趋势，适合嵌入表格、仪表板或行内文本。
doc-type: reference
tags:
  - avalonia pro
---

import chartsAnalyticsSparkline from '/img/controls/charts/charts-analytics-sparkline.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

迷你图是不带坐标轴或刻度的紧凑图表，用于在卡片、表格单元格或仪表板磁贴等小空间中显示一系列数值的趋势。

<Image light={chartsAnalyticsSparkline} maxWidth={400} position="center" cornerRadius="true" alt="迷你图示例，展示折线、面积、柱状和盈亏趋势。" />

## 使用时机

- **行内趋势**：在数据网格或文本段落中显示数据趋势。
- **仪表板摘要**：在一屏内为多个指标提供高密度可视化上下文。
- **紧凑可视化**：当趋势的总体形态比具体数值更重要时。

## 代码示例

### XAML

```xml
<Grid ColumnDefinitions="Auto,*" RowDefinitions="Auto,Auto,Auto,Auto" Margin="10">
    <TextBlock Text="折线:" VerticalAlignment="Center" Margin="0,0,10,5" />
    <SparklineChart xmlns="https://github.com/avaloniaui" Grid.Column="1" Height="40" SparklineType="Line" ItemsSource="{Binding SparklineData}"/>
    <TextBlock Grid.Row="1" Text="面积:" VerticalAlignment="Center" Margin="0,0,10,5" />
    <SparklineChart Grid.Row="1" Grid.Column="1" Height="40" SparklineType="Area" ItemsSource="{Binding SparklineData}"/>
    <TextBlock Grid.Row="2" Text="柱状:" VerticalAlignment="Center" Margin="0,0,10,5" />
    <SparklineChart Grid.Row="2" Grid.Column="1" Height="40" SparklineType="Bar" ItemsSource="{Binding SparklineData}"/>
    <TextBlock Grid.Row="3" Text="盈亏:" VerticalAlignment="Center" Margin="0,0,10,5" />
    <SparklineChart Grid.Row="3" Grid.Column="1" Height="40" SparklineType="WinLoss" ItemsSource="{Binding SparklineWinLossData}"/>
</Grid>
```

### 数据模型 (C#)

```csharp
public ObservableCollection<double> SparklineData { get; } = new()
{
    5, 10, 8, 15, 12, 20, 18, 25, 22, 30
};

public ObservableCollection<double> SparklineWinLossData { get; } = new()
{
    1, -1, 1, 1, -1, 1, -1, -1, 1, 1
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 趋势数据的集合。 | `null` |
| `ValuePath` | 当 `ItemsSource` 包含对象而非原始数值时使用的属性路径。 | `null` |
| `SparklineType` | 迷你图的样式：`Line`、`Area`、`Bar` 或 `WinLoss`。 | `Line` |
| `LineBrush` | 用于 `Line` 和 `Area` 迷你图的画刷。 | `null`（蓝色，见下方说明） |
| `AreaFill` | 用于填充 `Area` 迷你图区域的画刷。 | `null`（半透明蓝色，见下方说明） |
| `BarBrush` | 用于 `Bar` 迷你图柱状的画刷。 | `null`（蓝色，见下方说明） |
| `WinBrush` | 用于 `WinLoss` 迷你图中正值的画刷。 | `null`（绿色，见下方说明） |
| `LossBrush` | 用于 `WinLoss` 迷你图中负值的画刷。 | `null`（红色，见下方说明） |
| `ShowMarkers` | 切换是否渲染单个数据点标记。 | `false` |
| `ShowMinMax` | 突出显示最小值和最大值。 | `true` |
| `StrokeThickness` | `Line` 和 `Area` 迷你图的线条描边宽度。 | `2.0` |

:::note
`Brush` 类型的属性在设置为 `null` 时默认使用以下颜色：

- `LineBrush`：蓝色
- `AreaFill`：蓝色，降低不透明度
- `BarBrush`：与 `LineBrush` 相同，即两者都为 `null` 时为蓝色
- `WinBrush`：绿色
- `LossBrush`：红色
:::

## 另请参阅

- [KPI 卡片](/controls/data-display/charts/analytics/kpi-card)
- [折线图](/controls/data-display/charts/cartesian/line-chart)
