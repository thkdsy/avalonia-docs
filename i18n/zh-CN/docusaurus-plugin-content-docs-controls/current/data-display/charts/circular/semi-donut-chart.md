---
id: semi-donut-chart
title: 半环形图
description: 在 180 度弧中显示比例数据，常用于仪表板仪表和汇总指标，无需完整的圆形。
doc-type: reference
tags:
  - avalonia pro
---

import chartsPieSemidonut from '/img/controls/charts/charts-pie-semidonut.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

半环形图在 180 度弧中显示数据。它们在仪表板设计中特别受欢迎，用于在空间受限的区域中衡量进度或显示汇总指标。

<Image light={chartsPieSemidonut} maxWidth={400} position="center" cornerRadius="true" alt="半环形图显示为 180 度弧，带有彩色段和中心标签显示汇总指标。" />

## 何时使用
- **KPI 仪表**：可视化单个指标相对于目标或总量的情况。
- **仪表板标题**：在页面顶部提供品类的快速摘要。
- **角度比较**：比较整体的各个部分，不需要或不希望使用完整的圆形。

## 代码示例

### XAML
```xml
<SemiDonutChart xmlns="https://github.com/avaloniaui" Name="SemiDonutChartSample" Height="200"
                         ItemsSource="{Binding SemiDonutChartData}"
                         ValuePath="Value"
                         LabelPath="Label"
                         CenterValue="$980"
                         CenterLabel="总收入" />
```

### 数据模型 (C#)
```csharp
public record SemiDonutPoint(string Label, double Value);

public ObservableCollection<SemiDonutPoint> SemiDonutChartData { get; } = new()
{
    new("产品 A", 450),
    new("产品 B", 320),
    new("产品 C", 210)
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 段的数据源。 | `null` |
| `ValuePath` | 值的属性路径。 | `null` |
| `LabelPath` | 标签的属性路径。 | `null` |
| `InnerRadiusFactor`| 内半径（孔大小）与外半径的比率（0.0 到 1.0）。 | `0.6` |
| `CenterLabel` | 弧中心显示的标签文本。 | `null` |
| `CenterValue` | 中心显示的值文本。 | `null` |
| `GapAngle` | 段之间的间隙角度（度）。 | `2.0` |
| `IsHighlightEnabled` | 启用段的悬停高亮。 | `false` |
