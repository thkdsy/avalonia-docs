---
id: indented-tree-chart
title: 缩进树图
description: 在图表容器中使用类似文件资源管理器的缩进布局表示层次结构，支持高级样式和交互。
doc-type: reference
tags:
  - avalonia pro
---

import chartsHierarchicalIndentedtree from '/img/controls/charts/charts-hierarchical-tree.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

缩进树图使用类似于标准文件资源管理器或树视图的布局来表示层次结构，但在图表容器内提供高级样式和交互。

<Image light={chartsHierarchicalIndentedtree} maxWidth={400} position="center" cornerRadius="true" alt="缩进树图，显示类似文件资源管理器的层次结构，父节点和子节点通过缩进偏移。" />

## 何时使用
- **文件系统浏览器**：为本地或云存储构建自定义导航器。
- **物料清单（BOM）**：显示制造中的多层产品结构。
- **设置/配置**：以可视化方式对复杂的嵌套配置选项进行分组。

## 代码示例

### XAML
```xml
<IndentedTreeChart xmlns="https://github.com/avaloniaui" Name="IndentedTreeChartSample" Height="320"
                            ItemsSource="{Binding IndentedTreeData}"
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

public ObservableCollection<TreeNode> IndentedTreeData { get; } = new()
{
    new TreeNode { Name = "源码", Children = {
        new TreeNode { Name = "组件", Children = {
            new TreeNode { Name = "Button.cs" },
            new TreeNode { Name = "Chart.cs" }
        }},
        new TreeNode { Name = "模型", Children = {
            new TreeNode { Name = "User.cs" }
        }},
        new TreeNode { Name = "Program.cs" }
    }},
};
```

## 通用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 根级节点。 | `null` |
| `ValuePath` | 可选数值，在每个标签旁边显示。 | `null` |
| `LabelPath` | 项目文本的属性名称。 | `null` |
| `ChildrenPath` | 子集合的属性名称。 | `null` |
| `IndentSize` | 每个层次级别的水平偏移量。 | `20.0` |
| `RowHeight` | 每行渲染的高度。 | `24.0` |
| `ShowLines` | 是否显示节点之间的连接线。 | `true` |
| `ShowIcons` | 是否显示文件夹和叶子图标。 | `true` |
