---
id: heatmap-chart
title: 热力图
description: 使用二维矩阵中的颜色编码单元格表示数据值，突出显示模式、相关性和异常值。
doc-type: reference
tags:
  - avalonia pro
---

import chartsAnalyticsHeatmap from '/img/controls/charts/charts-analytics-heatmap.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

热力图使用颜色编码的单元格在二维矩阵中表示数据值，突出显示两个维度上的模式、相关性和异常值。

<Image light={chartsAnalyticsHeatmap} maxWidth={400} position="center" cornerRadius="true" alt="热力图，显示由颜色编码单元格组成的二维矩阵，表示跨行和列的数据值。" />

## 使用场景
- **相关性矩阵**：可视化变量之间的关系。
- **密度映射**：展示两个类别之间的频率或强度。
- **矩阵数据**：可视化行和列交叉点处的数值。

## 代码示例

### XAML
```xml
<HeatmapChart xmlns="https://github.com/avaloniaui" Title="Correlation Matrix" Height="300"
                       ItemsSource="{Binding HeatmapData}"
                       RowPath="Row" ColumnPath="Col" ValuePath="Val"/>
```

### 数据模型 (C#)
```csharp
using System;

public record HeatmapItem(string Row, string Col, double Val);

public ObservableCollection<HeatmapItem> HeatmapData { get; } = CreateHeatmapData();

private static ObservableCollection<HeatmapItem> CreateHeatmapData()
{
    var data = new ObservableCollection<HeatmapItem>();
    const int size = 10;

    for (var row = 0; row < size; row++)
    {
        for (var column = 0; column < size; column++)
        {
            var value = Math.Abs(Math.Sin(row * 0.5) * Math.Cos(column * 0.5) * 100);
            if (row == column)
            {
                value = 100;
            }

            data.Add(new($"R{row + 1}", $"C{column + 1}", value));
        }
    }

    return data;
}
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Title` | 图表标题。 | `null` |
| `ItemsSource` | 表示矩阵数据的集合。 | `null` |
| `RowPath` | 行标识符的属性路径。 | `null` |
| `ColumnPath` | 列标识符的属性路径。 | `null` |
| `ValuePath` | 单元格值的属性路径。 | `null` |
| `LowBrush` | 用于最低值的画笔。 | `#E3F2FD` |
| `HighBrush` | 用于最高值的画笔。 | `#1565C0` |
| `ShowLabels` | 是否在每个单元格内显示数值。 | `true` |
| `CellGap` | 单元格之间的间距。 | `2.0` |
| `CellCornerRadius` | 每个单元格的圆角半径。 | `CornerRadius(4)` |
| `LabelFontSize` | 行标签、列标签和单元格值的字体大小。 | `11.0` |
| `LabelForeground` | 用于行标签和列标签的画笔。单元格值使用对比感知文本。 | `null` |
| `IsHighlightEnabled` | 启用热力图单元格的悬停高亮。 | `false` |
