---
id: mindmap-chart
title: 思维导图 / 头脑风暴
description: 通过围绕中心主题排列 FlowChart 节点和链接来构建思维导图。
doc-type: reference
tags:
  - avalonia pro
---

import chartsFlowMindmap from '/img/controls/charts/charts-flow-mindmap.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

思维导图是用于头脑风暴和项目规划的发散性图表，从中心主题向外辐射到相关想法和子任务。在 `Avalonia.Controls.Charts` 中，此布局构建在 `FlowChart` 之上。

<Image light={chartsFlowMindmap} maxWidth={400} position="center" cornerRadius="true" alt="思维导图，从中心主题节点向外辐射到连接的子主题和想法。" />

## 何时使用
- **创意生成**：在会议期间捕获和组织想法。
- **项目范围**：绘制不同模块及其需求。
- **知识表示**：可视化复杂概念及其相互关联性。

## 代码示例

### XAML
```xml
<FlowChart xmlns="https://github.com/avaloniaui" Name="MindmapSample"
                  Title="Project Alpha 头脑风暴"
                  Height="600"
                  Nodes="{Binding MindmapNodes}"
                  Edges="{Binding MindmapEdges}"
                  CornerRadius="20" />
```

### 数据模型 (C#)
```csharp
using Avalonia.Media;

public ObservableCollection<FlowNode> MindmapNodes { get; } = new()
{
    new()
    {
        Id = "root",
        Text = "Project Alpha",
        X = 350,
        Y = 250,
        Shape = FlowShape.Circle,
        Background = Brushes.MediumPurple,
        Foreground = Brushes.White,
        Width = 120,
        Height = 120
    },
    new()
    {
        Id = "res",
        Text = "调研",
        X = 200,
        Y = 150,
        Shape = FlowShape.RoundedRect,
        Background = Brushes.Salmon,
        Width = 100
    },
    new()
    {
        Id = "re1",
        Text = "用户规格",
        X = 50,
        Y = 100,
        Shape = FlowShape.Rectangle,
        Background = Brushes.Snow,
        Width = 100
    },
    new()
    {
        Id = "re2",
        Text = "竞争对手",
        X = 50,
        Y = 200,
        Shape = FlowShape.Rectangle,
        Background = Brushes.Snow,
        Width = 100
    },
    new()
    {
        Id = "des",
        Text = "设计",
        X = 500,
        Y = 150,
        Shape = FlowShape.RoundedRect,
        Background = Brushes.SkyBlue,
        Width = 100
    },
    new()
    {
        Id = "de1",
        Text = "UI/UX",
        X = 650,
        Y = 100,
        Shape = FlowShape.Rectangle,
        Background = Brushes.Snow,
        Width = 100
    },
    new()
    {
        Id = "de2",
        Text = "架构",
        X = 650,
        Y = 200,
        Shape = FlowShape.Rectangle,
        Background = Brushes.Snow,
        Width = 100
    },
    new()
    {
        Id = "imp",
        Text = "实施",
        X = 350,
        Y = 400,
        Shape = FlowShape.RoundedRect,
        Background = Brushes.LightGreen,
        Width = 120
    },
    new()
    {
        Id = "im1",
        Text = "前端",
        X = 250,
        Y = 500,
        Shape = FlowShape.Rectangle,
        Background = Brushes.Snow,
        Width = 100
    },
    new()
    {
        Id = "im2",
        Text = "后端",
        X = 450,
        Y = 500,
        Shape = FlowShape.Rectangle,
        Background = Brushes.Snow,
        Width = 100
    }
};

public ObservableCollection<FlowEdge> MindmapEdges { get; } = new()
{
    new() { SourceId = "root", TargetId = "res", ShowArrow = false },
    new() { SourceId = "res", TargetId = "re1", ShowArrow = false },
    new() { SourceId = "res", TargetId = "re2", ShowArrow = false },
    new() { SourceId = "root", TargetId = "des", ShowArrow = false },
    new() { SourceId = "des", TargetId = "de1", ShowArrow = false },
    new() { SourceId = "des", TargetId = "de2", ShowArrow = false },
    new() { SourceId = "root", TargetId = "imp", ShowArrow = false },
    new() { SourceId = "imp", TargetId = "im1", ShowArrow = false },
    new() { SourceId = "imp", TargetId = "im2", ShowArrow = false }
};
```

## 通用属性 (`FlowChart`)

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Nodes` | 表示主题的 `FlowNode` 项目集合。 | `null` |
| `Edges` | 表示关系的 `FlowEdge` 项目集合。 | `null` |
| `NodeCornerRadius` | 渲染节点框的圆角。 | `10.0` |
| `Groups` | 可选的 `FlowGroup` 容器集合。 | `null` |
| `FlowNode.Shape` | 节点形状，如 `Rectangle`、`Circle` 或 `Diamond`。 | `Rectangle` |
