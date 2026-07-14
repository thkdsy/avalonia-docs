---
id: arc-diagram-chart
title: 弧线图
description: 网络可视化，节点线性排列，连接以弯曲弧线绘制，适用于显示有序数据集中的关系。
doc-type: reference
tags:
  - avalonia pro
---

import chartsFlowArc from '/img/controls/charts/charts-flow-arc.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

弧线图是一种网络可视化，节点沿轴线线性排列。连接以弯曲弧线绘制，其粗细或颜色表示连接强度。

<Image light={chartsFlowArc} maxWidth={400} position="center" cornerRadius="true" alt="弧线图，节点在水平轴上排列，由弯曲弧线连接，表示项目之间的关系。" />

## 何时使用
- **序列分析**：显示固定顺序中项目之间的关系（如书籍中的章节）。
- **依赖映射**：可视化单行上的调用栈或结构关系。
- **类别邻近性**：突出线性数据集中的交互集群。

## 代码示例

### XAML
```xml
<ArcDiagramChart xmlns="https://github.com/avaloniaui" Name="ArcDiagramSample" Title="连接关系" Height="300"
                          Nodes="{Binding ArcNodes}"
                          Links="{Binding ArcLinks}"
                          NodeIdPath="Id"
                          NodeLabelPath="Label"
                          SourcePath="Source"
                          TargetPath="Target"
                          LinkValuePath="Value" />
```

### 数据模型 (C#)
```csharp
public record ArcNode(string Id, string Label);
public record ArcLink(string Source, string Target, double Value);

public ObservableCollection<ArcNode> ArcNodes { get; } = new()
{
    new("1", "第 1 章"),
    new("2", "第 2 章"),
    new("3", "第 3 章"),
    new("4", "第 4 章"),
    new("5", "第 5 章")
};

public ObservableCollection<ArcLink> ArcLinks { get; } = new()
{
    new("1", "2", 1),
    new("1", "3", 3),
    new("2", "4", 2),
    new("3", "5", 4),
    new("2", "5", 1)
};
```

## 通用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Nodes` | 轴上的项目集合。 | `null` |
| `Links` | 节点之间的关系。 | `null` |
| `NodeIdPath` | 每个节点的唯一标识符路径。 | `null` |
| `NodeLabelPath` | 每个节点的文本路径。 | `null` |
| `SourcePath` | 每个连接的源节点标识符路径。 | `null` |
| `TargetPath` | 每个连接的目标节点标识符路径。 | `null` |
| `LinkValuePath` | 决定弧线粗细/大小的路径。 | `null` |
| `NodeSize` | 节点标记的大小。 | `16.0` |
| `ArcThickness` | 连接弧线的基础粗细。 | `2.0` |
| `ArcOpacity` | 连接弧线的不透明度。 | `0.5` |
