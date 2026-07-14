---
id: network-chart
title: 网络图
description: 在静态布局中显示节点和边，适用于基础设施图、依赖树和固定结构可视化。
doc-type: reference
tags:
  - avalonia pro
---

import chartsFlowNetwork from '/img/controls/charts/charts-flow-network.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

网络图提供节点和边的高层视图。它们比力导向图更简单、更静态，适用于固定的结构图。

<Image light={chartsFlowNetwork} maxWidth={400} position="center" cornerRadius="true" alt="网络图，带有标签的节点由边连接，显示结构图中的静态关系。" />

## 何时使用
- **基础设施图**：显示服务器及其连接。
- **路由表**：映射网络点之间的路径。
- **依赖树**：可视化系统中模块之间的关系。

## 代码示例

### XAML
```xml
<NetworkChart xmlns="https://github.com/avaloniaui" Name="NetworkChartSample" Title="社交网络" Height="350"
                       Nodes="{Binding NetworkNodes}"
                       Edges="{Binding NetworkEdges}" />
```

### 数据模型 (C#)
```csharp
public ObservableCollection<NetworkNode> NetworkNodes { get; } = new()
{
    new() { Id = "A", Label = "Alice", X = 0, Y = 0 },
    new() { Id = "B", Label = "Bob", X = 0, Y = 0 },
    new() { Id = "C", Label = "Charlie", X = 0, Y = 0 },
    new() { Id = "D", Label = "David", X = 0, Y = 0 },
    new() { Id = "E", Label = "Eve", X = 0, Y = 0 }
};

public ObservableCollection<NetworkEdge> NetworkEdges { get; } = new()
{
    new() { Source = "A", Target = "B", Weight = 2 },
    new() { Source = "A", Target = "C", Weight = 1 },
    new() { Source = "B", Target = "D", Weight = 3 },
    new() { Source = "C", Target = "E", Weight = 1 },
    new() { Source = "D", Target = "E", Weight = 2 },
    new() { Source = "B", Target = "E", Weight = 1 }
};
```

`NetworkChart` 直接使用 `NetworkNode` 和 `NetworkEdge` 对象，因此标签和权重在这些类型上定义，而不是通过额外的属性路径设置。

## 通用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Nodes` | 图节点的集合。 | `null` |
| `Edges` | 图边的集合。 | `null` |
