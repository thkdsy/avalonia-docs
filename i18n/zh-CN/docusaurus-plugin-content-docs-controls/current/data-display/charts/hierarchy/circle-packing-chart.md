---
id: circle-packing-chart
title: 圆形填充图
description: 将层次数据表示为嵌套圆圈，大圆包含较小的子圆，显示各层级的分组和相对大小。
doc-type: reference
tags:
  - avalonia pro
---

import chartsHierarchicalCirclepacking from '/img/controls/charts/charts-hierarchical-circle-packing.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

圆形填充图是树图的一种变体，其中节点表示为圆圈。较大的圆代表父类别，子类别作为较小的圆嵌套在其中。

<Image light={chartsHierarchicalCirclepacking} maxWidth={400} position="center" cornerRadius="true" alt="圆形填充图，具有嵌套圆圈，父类别包含按比例大小的子圆。" />

## 何时使用
- **聚类**：以视觉美观、有机的方式可视化组和子组。
- **美学概览**：仪表板中优先使用"气泡"视觉效果而非刚性网格。
- **关系邻近性**：显示类别内不同项目的关联程度。

## 代码示例

### XAML
```xml
<CirclePackingChart xmlns="https://github.com/avaloniaui" Name="CirclePackingChartSample" Title="包大小" Height="350"
                             ItemsSource="{Binding CirclePackingData}"
                             ValuePath="Size"
                             LabelPath="Name" />
```

### 数据模型 (C#)
```csharp
public record TreeMapItem(string Name, double Size);

public ObservableCollection<TreeMapItem> CirclePackingData { get; } = new()
{
    new("核心", 40),
    new("工具", 25),
    new("界面", 35)
};
```

## 通用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 层次数据源。 | `null` |
| `ValuePath` | 圆圈的大小/直径。 | `null` |
| `LabelPath` | 气泡附近或内部显示的文本。 | `null` |
| `ChildrenPath` | 每个节点的子集合路径。 | `null` |
| `CirclePadding` | 填充圆圈之间的间距。 | `3.0` |
| `Palette` | 类别的自定义画笔集合。 | 自动生成 |
