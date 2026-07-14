---
id: pareto-chart
title: 帕累托图
description: 将降序柱条与累积折线结合，基于 80/20 原则突出最关键的因素。
doc-type: reference
tags:
  - avalonia pro
---

import chartsStatisticalPareto from '/img/controls/charts/charts-statistical-pareto.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

帕累托图同时包含柱条和折线图，其中单个值通过柱条以降序表示，累积总数通过折线表示。

<Image light={chartsStatisticalPareto} maxWidth={400} position="center" cornerRadius="true" alt="帕累托图，包含降序柱条和累积百分比折线，突出最关键的因素。" />

## 何时使用
- **质量控制**：识别缺陷的"关键少数"原因（80/20 法则）。
- **资源管理**：确定哪些品类占用了大部分成本。
- **客户服务**：分析哪些投诉最常见。

## 代码示例

### XAML
```xml
<ParetoChart xmlns="https://github.com/avaloniaui" Name="ParetoChartSample"
                                          Title="缺陷分析"
                                          Height="250"
                                          ValuePath="Count"
                                          LabelPath="Defect"
                                          ItemsSource="{Binding ParetoData}" />
```

### 数据模型 (C#)
```csharp
public record ParetoItem(string Defect, int Count);

public ObservableCollection<ParetoItem> ParetoData { get; } = new()
{
    new("缺少零件", 45),
    new("表面损坏", 30),
    new("尺寸错误", 20),
    new("颜色误差", 15),
    new("其他", 10)
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 品类的集合。 | `null` |
| `ValuePath` | 决定柱条高度的属性。 | `null` |
| `LabelPath` | 品类名称的属性。 | `null` |
| `BarBrush` | 降序柱条的画刷。 | `#1976D2` |
| `LineBrush` | 累积百分比折线的画刷。 | `#F44336` |
| `BarWidth` | 每个柱条的宽度，以品类宽度的比例表示。 | `0.7` |
| `ShowCumulativeLine` | 切换累积百分比折线和标记的显示。 | `true` |
| `IsHighlightEnabled` | 启用帕累托柱条的悬停高亮。 | `false` |
