---
id: treemap-chart
title: 树图
description: 将层次数据可视化为按值大小排列的嵌套矩形，用于比较类别内的比例，如磁盘使用率或预算。
doc-type: reference
tags:
  - avalonia pro
---

import chartsHierarchicalTreemap from '/img/controls/charts/charts-hierarchical-treemap.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

树图将层次或平面数据可视化为一组嵌套矩形。每个分支被赋予一个矩形，根据其值调整大小，并用较小的子矩形平铺。

<Image light={chartsHierarchicalTreemap} maxWidth={400} position="center" cornerRadius="true" alt="树图，显示嵌套矩形，按比例大小表示不同文件夹的磁盘使用值。" />

## 何时使用
- **资源使用**：按文件/进程可视化磁盘空间或内存消耗。
- **比例分析**：比较类别中项目的权重。
- **复杂层次**：当需要在单个视图中显示许多层次项目时。

## 代码示例

### XAML
```xml
<TreeMapChart xmlns="https://github.com/avaloniaui" Name="TreeMapSample" Title="磁盘使用情况" Height="300"
                       ItemsSource="{Binding TreeMapData}" ValuePath="Size" LabelPath="Name" />
```

### 数据模型 (C#)
```csharp
public record TreeMapItem(string Name, double Size);

public ObservableCollection<TreeMapItem> TreeMapData { get; } = new()
{
    new("文档", 45),
    new("照片", 30),
    new("视频", 50),
    new("音乐", 20),
    new("下载", 35)
};
```

## 通用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Title` | 图表标题。 | `null` |
| `ItemsSource` | 数据项目的集合。 | `null` |
| `ValuePath` | 表示区域大小的属性路径。 | `null` |
| `LabelPath` | 标签的属性路径。 | `null` |
| `TileGap` | 矩形之间的间距。 | `1.0` |
| `IsHighlightEnabled` | 启用树图节点的悬停高亮。 | `false` |
| `IsSelectionEnabled` | 启用节点选择。 | `false` |
| `SelectionMode` | 选择行为，如 `None`、`Single`、`SingleDeselect` 或 `Multiple`。 | `SingleDeselect` |
| `SelectionBrush` | 用于高亮选定段的画笔。 | `FromRgb(49, 74, 110)` |
| `SelectionStroke` | 用于勾勒选定段的画笔。 | 使用主题默认值。 |
| `SelectionStrokeThickness` | 选定段轮廓的粗细。 | `2.0` |
| `SelectedIndex` | 主要选定节点的索引。 | `-1` |
