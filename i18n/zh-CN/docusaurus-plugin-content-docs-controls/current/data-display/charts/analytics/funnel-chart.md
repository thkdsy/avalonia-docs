---
id: funnel-chart
title: 漏斗图
description: 可视化数据在线性流程中逐步缩减的过程，适用于识别流程中的瓶颈。
doc-type: reference
tags:
  - avalonia pro
---

import chartsAnalyticsFunnel from '/img/controls/charts/charts-analytics-funnel.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

漏斗图用于可视化数据从一个阶段传递到另一个阶段时的逐步缩减过程。使用它们可以识别线性流程中的瓶颈。

<Image light={chartsAnalyticsFunnel} maxWidth={400} position="center" cornerRadius="true" alt="漏斗图，展示从潜在客户到成交的销售管道各阶段数据的逐步缩减。" />

## 使用场景
- **销售管道**：跟踪潜在客户从线索到成交的全过程。
- **转化率**：监控网站访客在结账流程中的转化情况。
- **招聘流程**：可视化候选人在招聘各阶段的分布情况。

## 代码示例

### XAML
```xml
<FunnelChart xmlns="https://github.com/avaloniaui" Title="Sales Pipeline" Height="300"
                      ItemsSource="{Binding FunnelData}"
                      LabelPath="Stage" ValuePath="Value"/>
```

### 数据模型 (C#)
```csharp
public record FunnelItem(string Stage, double Value);

public ObservableCollection<FunnelItem> FunnelData { get; } = new()
{
    new("Visitors", 1000),
    new("Leads", 500),
    new("Qualified", 200),
    new("Proposal", 80),
    new("Closed", 30)
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Title` | 顶部显示的名称。 | `null` |
| `ItemsSource` | 流程阶段的集合。 | `null` |
| `ValuePath` | 表示每个阶段数量的属性路径。 | `null` |
| `LabelPath` | 阶段名称的属性路径。 | `null` |
| `NeckWidth` | 漏斗"颈部"宽度比例（0-1）。 | `0.3` |
| `SegmentGap` | 漏斗段之间的间距。 | `2.0` |
| `ShowLabels` | 是否在分段上显示标签。 | `true` |
| `ShowValues` | 是否在图表上显示数值。 | `true` |
| `LabelFontSize` | 分段标签的字体大小。 | `12.0` |
| `LabelForeground` | 分段标签的画笔。当为 `null` 时，标签使用白色。 | `null` |
| `IsHighlightEnabled` | 启用漏斗分段的悬停高亮。 | `false` |
| `IsSelectionEnabled` | 是否启用分段选择。 | `false` |
| `SelectionMode` | 分段的选择行为。 | `SingleDeselect` |
| `SelectionBrush` | 选中分段的画笔。 | `#314A6E` |
| `SelectionStroke` | 选中分段的轮廓画笔。 | `null` |
| `SelectionStrokeThickness` | 选中分段的轮廓粗细。 | `2.0` |
| `SelectedIndex` | 选中分段的索引，未选择时为 `-1`。 | `-1` |
