---
id: sunburst-chart
title: 旭日图
description: 将层次数据可视化为同心环，每个环代表一个层级，段显示每个节点在其父节点中的比例。
doc-type: reference
tags:
  - avalonia pro
---

import chartsHierarchicalSunburst from '/img/controls/charts/charts-hierarchical-sunburst.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

旭日图通过一系列同心环来可视化层次数据。每个环代表层次中的一个层级，内圈为根层级。

<Image light={chartsHierarchicalSunburst} maxWidth={400} position="center" cornerRadius="true" alt="旭日图，具有同心环，每个环代表一个层次级别，段显示比例。" />

## 何时使用
- **嵌套数据**：可视化具有多个级别的复杂层次结构。
- **空间效率**：当你需要树图的紧凑替代方案时。
- **下钻**：有效显示每个层级的分段细分。

## 代码示例

### XAML
```xml
<SunburstChart xmlns="https://github.com/avaloniaui" Name="SunburstChartSample"
                        Title="组织结构"
                        Height="350"
                        ItemsSource="{Binding SunburstData}"
                        ValuePath="Size"
                        LabelPath="Name"
                        ChildrenPath="Children" />
```

### 数据模型 (C#)
```csharp
public class SunburstNode
{
    public string Name { get; set; } = string.Empty;
    public double Size { get; set; }
    public ObservableCollection<SunburstNode> Children { get; set; } = new();

    public SunburstNode(string name, double size = 0)
    {
        Name = name;
        Size = size;
    }
}

public ObservableCollection<SunburstNode> SunburstData { get; } = new()
{
    new SunburstNode("工程部", 40)
    {
        Children = new()
        {
            new SunburstNode("前端", 15)
            {
                Children = new()
                {
                    new SunburstNode("React", 8),
                    new SunburstNode("Angular", 7)
                }
            },
            new SunburstNode("后端", 18),
            new SunburstNode("DevOps", 7)
        }
    },
    new SunburstNode("销售部", 30)
    {
        Children = new()
        {
            new SunburstNode("专业版", 18),
            new SunburstNode("中小企业", 12)
        }
    },
    new SunburstNode("市场部", 20)
    {
        Children = new()
        {
            new SunburstNode("数字营销", 12),
            new SunburstNode("品牌", 8)
        }
    },
    new SunburstNode("人事部", 10)
};
```

## 通用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Title` | 图表标题。 | `null` |
| `ItemsSource` | 根级数据项目的集合。 | `null` |
| `ValuePath` | 表示段大小的属性路径。 | `null` |
| `LabelPath` | 段标签的属性路径。 | `null` |
| `ChildrenPath` | 子项目集合的路径。 | `null` |
| `InnerRadiusFactor` | 中心孔的相对大小，从 `0.0` 到 `1.0`。 | `0.2` |
| `RingThickness` | 每个环的厚度。 | `40.0` |
| `GapAngle` | 段之间的角度间隔。 | `2.0` |
| `IsHighlightEnabled` | 启用悬停高亮显示段。 | `false` |
| `IsSelectionEnabled` | 是否启用数据点选择。 | `false` |
| `SelectionMode` | 选择模式，例如 `None`、`Single`、`SingleDeselect` 或 `Multiple`。 | `SingleDeselect` |
| `SelectionBrush` | 用于高亮选定段的画笔。 | `FromRgb(49, 74, 110)` |
| `SelectionStroke` | 用于勾勒选定段的画笔。 | 使用主题默认值。 |
| `SelectionStrokeThickness` | 选定段轮廓的粗细。 | `2.0` |
| `SelectedIndex` | 选定数据点的索引。 | `-1` |
