---
id: theme-river-chart
title: 主题河流图
description: 通过在笛卡尔图表中堆叠居中的 StackedAreaSeries 实例来构建主题河流布局。
doc-type: reference
tags:
  - avalonia pro
---

import chartsAnalyticsThemeriver from '/img/controls/charts/charts-flow-themeriver.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

主题河流图用于可视化各类别随时间的变化。在 `Avalonia.Controls.Charts` 中，可以通过在 `CartesianChart` 内组合多个 `StackedAreaSeries` 实例来构建此布局。

<Image light={chartsAnalyticsThemeriver} maxWidth={400} position="center" cornerRadius="true" alt="主题河流图，展示堆叠的有机流带，反映各类别数据量如何随时间流动和变化。" />

## 使用场景
- **主题趋势**：可视化新闻或社交媒体中主题随时间的热度变化。
- **资源分配**：展示预算或人力在项目之间的变动情况。
- **使用模式**：跟踪不同类型网络流量的数据量。

## 代码示例

### XAML
```xml
<CartesianChart xmlns="https://github.com/avaloniaui" Name="ThemeRiverSample" Title="Data Stream" Height="400"
                                           ShowLegend="True" LegendPosition="Bottom">
                        <CartesianChart.Series>
                            <!-- Dummy Series (Transparent Spacer for Wiggle/Centering) -->
                            <StackedAreaSeries Title="" ItemsSource="{Binding ThemeRiverDummy}"
                                                      CategoryPath="Category" ValuePath="Value"
                                                      StackGroup="River"
                                                      Fill="Transparent"
                                                      StrokeThickness="0" />

                            <!-- Visible Data Series -->
                            <StackedAreaSeries Title="Stream A" ItemsSource="{Binding ThemeRiverSeriesA}"
                                                      CategoryPath="Category" ValuePath="Value"
                                                      StackGroup="River"
                                                      Fill="#FF6B6B" Stroke="#E05555" StrokeThickness="1" />

                            <StackedAreaSeries Title="Stream B" ItemsSource="{Binding ThemeRiverSeriesB}"
                                                      CategoryPath="Category" ValuePath="Value"
                                                      StackGroup="River"
                                                      Fill="#4ECDC4" Stroke="#3EBDB4" StrokeThickness="1" />

                            <StackedAreaSeries Title="Stream C" ItemsSource="{Binding ThemeRiverSeriesC}"
                                                      CategoryPath="Category" ValuePath="Value"
                                                      StackGroup="River"
                                                      Fill="#FFE66D" Stroke="#EED55D" StrokeThickness="1" />
                        </CartesianChart.Series>

                        <CartesianChart.HorizontalAxis>
                             <CategoryAxis ShowGridLines="False" />
                        </CartesianChart.HorizontalAxis>

                        <CartesianChart.VerticalAxis>
                             <NumericalAxis ShowGridLines="False" IsVisible="False" />
                        </CartesianChart.VerticalAxis>
                    </CartesianChart>
```

### 数据模型 (C#)
```csharp
using System.Collections.Generic;

public class ThemeRiverViewModel
{
    public List<ThemeRiverItem> ThemeRiverDummy { get; } = new();
    public List<ThemeRiverItem> ThemeRiverSeriesA { get; } = new();
    public List<ThemeRiverItem> ThemeRiverSeriesB { get; } = new();
    public List<ThemeRiverItem> ThemeRiverSeriesC { get; } = new();

    public ThemeRiverViewModel()
    {
        GenerateThemeRiverData();
    }

    private void GenerateThemeRiverData()
    {
        const int count = 30;
        const double center = 50.0;

        for (var i = 0; i < count; i++)
        {
            var category = $"T{i}";
            var streamA = 10 + 5 * System.Math.Sin(i * 0.3);
            var streamB = 15 + 8 * System.Math.Sin(i * 0.5 + 1);
            var streamC = 12 + 6 * System.Math.Cos(i * 0.4);
            var offset = center - (streamA + streamB + streamC) / 2;

            ThemeRiverDummy.Add(new ThemeRiverItem { Category = category, Value = offset });
            ThemeRiverSeriesA.Add(new ThemeRiverItem { Category = category, Value = streamA });
            ThemeRiverSeriesB.Add(new ThemeRiverItem { Category = category, Value = streamB });
            ThemeRiverSeriesC.Add(new ThemeRiverItem { Category = category, Value = streamC });
        }
    }
}

public class ThemeRiverItem
{
    public string Category { get; set; } = string.Empty;
    public double Value { get; set; }
}
```

## 常用属性 (`StackedAreaSeries`)

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Title` | 图例中显示的系列名称。 | `null` |
| `ItemsSource` | 单个流的数据点集合。 | `null` |
| `CategoryPath` | 用于水平类别或时间段的属性。 | `null` |
| `ValuePath` | 决定流厚度的属性。 | `null` |
| `StackGroup` | 相关面积系列共享的堆叠组。 | `null` |
| `Fill` | 流区域的画笔。 | 取决于主题 |
| `Stroke` | 流轮廓的画笔。 | 取决于主题 |
