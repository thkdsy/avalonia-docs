---
id: force-directed-graph
title: 力导向图
description: 使用物理模拟布局网络节点，以揭示复杂互联数据中的聚类和结构关系。
doc-type: reference
tags:
  - avalonia pro
---

import chartsFlowForceDirected from '/img/controls/charts/charts-flow-force-directed.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

力导向图使用物理模拟来布局网络节点。它们有助于揭示复杂互联数据中的聚类和结构关系。

<Image light={chartsFlowForceDirected} maxWidth={400} position="center" cornerRadius="true" alt="力导向图，节点和连接边由物理模拟排列以揭示聚类。" />

## 何时使用
- **社交网络**：可视化好友关系、关注者或社区结构。
- **知识图谱**：显示概念、实体或研究论文之间的关系。
- **系统架构**：映射微服务及其通信链接。

## 代码示例

### XAML
```xml
<ForceDirectedGraph xmlns="https://github.com/avaloniaui" Name="ForceGraphSample" Title="网络图" Height="400"
                             NodesSource="{Binding ForceNodes}"
                             EdgesSource="{Binding ForceEdges}"
                             NodeIdPath="Id"
                             NodeLabelPath="Label"
                             EdgeSourcePath="Source"
                             EdgeTargetPath="Target" />
```

### 数据模型 (C#)
```csharp
public record GraphNode(string Id, string Label);
public record GraphEdge(string Source, string Target);

public ObservableCollection<GraphNode> ForceNodes { get; } = new()
{
    new("N1", "主节点"),
    new("N2", "服务器 1"),
    new("N3", "服务器 2"),
    new("N4", "客户端 A"),
    new("N5", "客户端 B"),
    new("N6", "数据库")
};

public ObservableCollection<GraphEdge> ForceEdges { get; } = new()
{
    new("N1", "N2"),
    new("N1", "N3"),
    new("N2", "N6"),
    new("N3", "N6"),
    new("N2", "N4"),
    new("N3", "N5")
};
```

## 通用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `NodesSource` | 节点集合。 | `null` |
| `EdgesSource` | 链接集合。 | `null` |
| `NodeIdPath` | 每个节点标识符的属性路径。 | `null` |
| `NodeLabelPath` | 每个节点标签的属性路径。 | `null` |
| `EdgeSourcePath` | 每个边源标识符的属性路径。 | `null` |
| `EdgeTargetPath` | 每个边目标标识符的属性路径。 | `null` |
| `NodeRadius` | 节点圆圈的半径。 | `20.0` |
| `RepulsionForce` | 节点之间的斥力强度。 | `5000.0` |
| `AttractionForce` | 沿边的引力强度。 | `0.01` |
| `IsAnimationEnabled` | 运行模拟和动画布局过渡。 | `true` |
