---
id: radial-tree-chart
title: 径向树图
description: 层次布局，根节点居中，子节点在同心环中向外辐射，对大型树结构空间效率高。
doc-type: reference
tags:
  - avalonia pro
---

import chartsHierarchicalRadialtree from '/img/controls/charts/charts-hierarchical-radial-tree.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

径向树图表示层次数据，根节点居中，子节点在同心圆中向外辐射。这种布局对大型树具有很高的空间效率。

<Image light={chartsHierarchicalRadialtree} maxWidth={400} position="center" cornerRadius="true" alt="径向树图，根节点居中，子节点在同心环中向外辐射。" />

## 何时使用
- **目录可视化器**：以紧凑的圆形形式显示多层文件夹。
- **基因组图谱**：可视化许多生物实体之间的关系。
- **网络拓扑**：从中心集线器辐射映射网络中的设备。

## 代码示例

### XAML
```xml
<RadialTreeChart xmlns="https://github.com/avaloniaui" Name="RadialTreeChartSample" Title="分类学" Height="400"
                          ItemsSource="{Binding RadialTreeData}"
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

public ObservableCollection<TreeNode> RadialTreeData { get; } = new()
{
    new TreeNode { Name = "动物", Children = {
        new TreeNode { Name = "哺乳动物", Children = {
            new TreeNode { Name = "狗" },
            new TreeNode { Name = "猫" }
        }},
        new TreeNode { Name = "鸟类", Children = {
            new TreeNode { Name = "鹰" }
        }},
        new TreeNode { Name = "鱼类", Children = {
            new TreeNode { Name = "鲑鱼" }
        }}
    }}
};
```

## 通用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 中心根节点。 | `null` |
| `ValuePath` | 与每个节点关联的值的路径。 | `null` |
| `LabelPath` | 节点上文本标签的路径。 | `null` |
| `ChildrenPath` | 外部节点集合的路径。 | `null` |
| `NodeSize` | 绘制节点点的半径。 | `8.0` |
| `LevelSpacing` | 径向级别之间的首选间距。 | `60.0` |
