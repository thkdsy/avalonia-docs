---
id: icicle-chart
title: 冰柱图
description: 将层次数据可视化为每层相邻的矩形，显示父子关系，用于分类法和结构分析。
doc-type: reference
tags:
  - avalonia pro
---

import chartsHierarchicalIcicle from '/img/controls/charts/charts-hierarchical-icicle.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

冰柱图使用并排放置的矩形来可视化层次数据。层次深度的每一层由一行或一列表示，显示跨层的父子关系。

<Image light={chartsHierarchicalIcicle} maxWidth={400} position="center" cornerRadius="true" alt="冰柱图，将层次级别显示为相邻的矩形行，宽度表示相对值。" />

## 何时使用
- **结构分析**：检查代码库、目录结构或大型分类法。
- **性能分析**：可视化调用栈或执行路径。
- **关系发现**：在大型树中查找叶节点值的根本原因。

## 代码示例

### XAML
```xml
<IcicleChart xmlns="https://github.com/avaloniaui" Name="IcicleChartSample" Title="文件系统" Height="300"
                      ItemsSource="{Binding IcicleData}"
                      ValuePath="Size"
                      LabelPath="Name" />
```

### 数据模型 (C#)
```csharp
public record TreeMapItem(string Name, double Size);

public ObservableCollection<TreeMapItem> IcicleData { get; } = new()
{
    new("源码", 60),
    new("测试", 25),
    new("文档", 15)
};
```

## 通用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 层次数据源。 | `null` |
| `ValuePath` | 决定矩形宽度的属性名称。 | `null` |
| `LabelPath` | 文本标签的属性名称。 | `null` |
| `ChildrenPath` | 子节点集合的路径。 | `null` |
| `Orientation` | 图表方向，`Horizontal` 或 `Vertical`。 | `Vertical` |
| `TileGap` | 相邻矩形之间的间距。 | `1.0` |
