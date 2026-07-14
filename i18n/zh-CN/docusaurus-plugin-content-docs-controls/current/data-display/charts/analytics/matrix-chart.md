---
id: matrix-chart
title: 矩阵图
description: 使用网格可视化两个分类集之间的布尔关系，显示每个行列交叉点是否存在某个特性或状态。
doc-type: reference
tags:
  - avalonia pro
---

import chartsAnalyticsMatrix from '/img/controls/charts/charts-analytics-matrix.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

矩阵图使用网格来可视化两个分类集之间的布尔关系。它们非常适合展示在行和列交叉点处是否存在某个特性、状态或权限。

<Image light={chartsAnalyticsMatrix} maxWidth={400} position="center" cornerRadius="true" alt="矩阵图，在行列交叉点处显示按值确定大小或颜色的圆点网格。" />

## 使用场景
- **相关性表**：展示多个不同变量之间的关系。
- **日程概览**：映射人员和日期之间的可用性或事件。
- **属性对比**：可视化哪些特性（列）适用于哪些产品（行）。

## 代码示例

### XAML
```xml
<MatrixChart xmlns="https://github.com/avaloniaui" Title="Fruit Attributes" Height="300"
                                          ItemsSource="{Binding MatrixData}"
                                          ColumnLabels="{Binding MatrixColumns}"
                                          RowLabelPath="Attribute" ValuesPath="Values"
                                          CellSize="28" CellGap="25"/>
```

### 数据模型 (C#)
```csharp
public record MatrixItem(string Attribute, bool[] Values);

public ObservableCollection<string> MatrixColumns { get; } = new()
{
    "Grape", "Banana", "Orange", "Apple"
};

public ObservableCollection<MatrixItem> MatrixData { get; } = new()
{
    new("Good for juicing", new[] { false, true, true, true }),
    new("Good for smoothies", new[] { true, true, false, true }),
    new("Good for baking", new[] { true, false, true, true }),
    new("Good for jam", new[] { true, false, false, true }),
    new("Good for salads", new[] { true, false, true, true })
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 行数据的集合。 | `null` |
| `RowLabelPath` | 行标签属性的路径。 | `null` |
| `ColumnLabels` | 顶部显示的列标签列表。 | `null` |
| `ValuesPath` | 行单元格布尔值的路径。 | `null` |
| `CellSize` | 每个矩阵单元格的直径。 | `30.0` |
| `CellGap` | 单元格之间的间距。 | `2.0` |
| `TrueBrush` | 用于 `true` 值的画笔。 | `null` |
| `FalseBrush` | 用于 `false` 值的画笔。 | `null` |
| `ShowFilledCircles` | `true` 值是否填充而非描边。 | `true` |
| `ShowRowLabels` | 是否显示每行的标签。 | `true` |
| `ShowColumnLabels` | 是否显示每列的标签。 | `true` |
| `LabelFontSize` | 行和列标签的字体大小。 | `11.0` |
| `IsHighlightEnabled` | 启用矩阵单元格的悬停高亮。 | `false` |
