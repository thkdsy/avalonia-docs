---
id: dendrogram-chart
title: 系统树图
description: 树状图，通过显示项目如何逐步合并成分支来说明层次聚类，用于统计和生物学分析。
doc-type: reference
tags:
  - avalonia pro
---

import chartsHierarchicalDendrogram from '/img/controls/charts/charts-hierarchical-dendrogram.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

系统树图是常用于说明层次聚类产生的簇排列的树状图。它们展示项目如何合并成单个分支。

<Image light={chartsHierarchicalDendrogram} maxWidth={400} position="center" cornerRadius="true" alt="系统树图，显示层次聚类，分支从叶节点向根合并。" />

## 何时使用
- **聚类分析**：可视化统计聚类算法的结果。
- **系统发育树**：显示不同物种之间的进化关系。
- **结构合并**：表示源自多个部分但汇聚成少数几组的数据。

## 代码示例

### XAML
```xml
<DendrogramChart xmlns="https://github.com/avaloniaui" Name="DendrogramChartSample" Title="聚类分析" Height="300"
                          ItemsSource="{Binding DendrogramData}"
                          LabelPath="Name"
                          ChildrenPath="Children" />
```

### 数据模型 (C#)
```csharp
public class TreeNode
{
    public string Name { get; set; } = string.Empty;
    public ObservableCollection<TreeNode> Children { get; set; } = new();
}

public ObservableCollection<TreeNode> DendrogramData { get; } = new()
{
    new TreeNode { Name = "根节点", Children = {
        new TreeNode { Name = "A", Children = {
            new TreeNode { Name = "A1" },
            new TreeNode { Name = "A2" }
        }},
        new TreeNode { Name = "B", Children = {
            new TreeNode { Name = "B1" }
        }}
    }}
};
```

## 通用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 层次聚类数据。 | `null` |
| `LabelPath` | 叶节点或节点名称的属性。 | `null` |
| `ChildrenPath` | 嵌套聚类项目的路径。 | `null` |
| `DistancePath` | 聚类距离或合并高度的属性。 | `null` |
| `Orientation` | 图表方向，`Horizontal` 或 `Vertical`。 | `Horizontal` |
| `LinkStyle` | 连接项目的线条样式，`Elbow` 或 `Straight`。 | `Elbow` |
| `LeafSpacing` | 叶节点之间的间距。 | `25.0` |
