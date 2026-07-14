---
id: sankey-chart
title: 桑基图
description: 使用比例宽度的链接表示每个连接的数量，可视化数据、能量或材料在阶段之间的流动。
doc-type: reference
tags:
  - avalonia pro
---

import chartsFlowSankey from '/img/controls/charts/charts-flow-sankey.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

桑基图可视化数据、能量或材料在阶段之间的流动。连接链接的宽度与流动量成比例。

<Image light={chartsFlowSankey} maxWidth={400} position="center" cornerRadius="true" alt="桑基图，显示阶段之间的能量流动，链接宽度与传输量成比例。" />

## 何时使用
- **能源审计**：显示能源从来源到消费的分配方式。
- **网络分析**：可视化用户通过网站的路径（用户体验旅程）。
- **预算编制**：跟踪资金如何从收入来源流向各种支出。

## 代码示例

### XAML
```xml
<SankeyChart xmlns="https://github.com/avaloniaui" Name="SankeySample" Title="能源流动" Height="350"
                      ItemsSource="{Binding SankeyData}"
                      SourcePath="Source"
                      TargetPath="Target"
                      ValuePath="Value" />
```

### 数据模型 (C#)
```csharp
public record FlowItem(string Source, string Target, double Value);

public ObservableCollection<FlowItem> SankeyData { get; } = new()
{
    new("太阳能", "电网", 40),
    new("风能", "电网", 30),
    new("煤炭", "电网", 10),
    new("电网", "工业", 30),
    new("电网", "居民", 25),
    new("电网", "交通", 15),
    new("电网", "损耗", 10)
};
```

## 通用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 流动数据集合。 | `null` |
| `SourcePath` | 起始节点的属性名称。 | `null` |
| `TargetPath` | 结束节点的属性名称。 | `null` |
| `ValuePath` | 流动幅度的属性名称。 | `null` |
| `NodeWidth` | 每个节点列的宽度。 | `20` |
| `NodePadding` | 列中节点之间的垂直空间。 | `10` |
