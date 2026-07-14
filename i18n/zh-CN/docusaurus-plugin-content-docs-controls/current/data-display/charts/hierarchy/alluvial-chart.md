---
id: alluvial-chart
title: 冲积图
description: 使用垂直节点列之间的流动带表示跨连续阶段的结构变化，类似于桑基图。
doc-type: reference
tags:
  - avalonia pro
---

import chartsFlowAlluvial from '/img/controls/charts/charts-flow-alluvial.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

冲积图表示随时间或跨类别的结构变化。它们类似于桑基图，但通常组织成不同的垂直列。

<Image light={chartsFlowAlluvial} maxWidth={400} position="center" cornerRadius="true" alt="冲积图，具有垂直节点列，由流动带连接，表示类别间的阶段过渡。" />

## 何时使用
- **工作流分析**：跟踪项目如何通过连续的流程阶段。
- **分类流动**：可视化一个类别的成员如何属于其他类别（例如，选民隶属关系的变化）。
- **结构变化**：展示群体分组在两个时间点之间的变化。

## 代码示例

### XAML
```xml
<AlluvialChart xmlns="https://github.com/avaloniaui" Name="AlluvialChartSample" Title="类别流动" Height="350"
                        Nodes="{Binding AlluvialNodes}"
                        Links="{Binding AlluvialLinks}" />
```

### 数据模型 (C#)
```csharp
public ObservableCollection<AlluvialNode> AlluvialNodes { get; } = new()
{
    new() { Id = "Jan", Label = "一月", Step = 0, Value = 50 },
    new() { Id = "Feb", Label = "二月", Step = 0, Value = 60 },
    new() { Id = "CatA", Label = "类别 A", Step = 1, Value = 55 },
    new() { Id = "CatB", Label = "类别 B", Step = 1, Value = 55 }
};

public ObservableCollection<AlluvialLink> AlluvialLinks { get; } = new()
{
    new() { Source = "Jan", Target = "CatA", Value = 30 },
    new() { Source = "Jan", Target = "CatB", Value = 20 },
    new() { Source = "Feb", Target = "CatA", Value = 25 },
    new() { Source = "Feb", Target = "CatB", Value = 35 }
};
```

`AlluvialChart` 使用强类型的 `AlluvialNode` 和 `AlluvialLink` 类。每个节点定义了 `Id`、`Label`、`Step` 和 `Value`。

## 通用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Nodes` | 分类节点（列）的集合。 | `null` |
| `Links` | 节点之间连接的集合。 | `null` |
