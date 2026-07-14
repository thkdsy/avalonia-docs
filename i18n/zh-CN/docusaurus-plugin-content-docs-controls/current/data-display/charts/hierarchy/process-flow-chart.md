---
id: process-flow-chart
title: 工艺流程图
description: 使用 FlowChart 节点和边来可视化业务工作流和决策树。
doc-type: reference
tags:
  - avalonia pro
---

import chartsFlowProcess from '/img/controls/charts/charts-flow-process.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

工艺流程图用于可视化系统中步骤、决策和逻辑结果的序列。在 `Avalonia.Controls.Charts` 中，这通过 `FlowChart`、`FlowNode` 和 `FlowEdge` 实现。

<Image light={chartsFlowProcess} maxWidth={400} position="center" cornerRadius="true" alt="工艺流程图，包含开始、决策和操作节点，由方向箭头连接，显示工作流序列。" />

## 何时使用
- **工作流映射**：可视化业务流程或审批链。
- **决策树**：显示故障排除或用户旅程的逻辑路径。
- **系统架构**：映射不同模块或服务之间的连接。

## 代码示例

### XAML
```xml
<FlowChart xmlns="https://github.com/avaloniaui" Name="FlowChartSample"
                  Title="台灯故障排查"
                  Height="400"
                  Nodes="{Binding FlowNodes}"
                  Edges="{Binding FlowEdges}" />
```

### 数据模型 (C#)
```csharp
using Avalonia.Media;

public ObservableCollection<FlowNode> FlowNodes { get; } = new()
{
    new()
    {
        Id = "start",
        Text = "台灯不亮",
        Shape = FlowShape.RoundedRect,
        X = 300,
        Y = 50,
        Background = Brushes.Salmon,
        Foreground = Brushes.White,
        Width = 200
    },
    new()
    {
        Id = "check_plug",
        Text = "台灯是否已插电？",
        Shape = FlowShape.Diamond,
        X = 280,
        Y = 150,
        Width = 240,
        Height = 100,
        Background = Brushes.LightBlue
    },
    new()
    {
        Id = "plug_in",
        Text = "插入电源",
        Shape = FlowShape.Rectangle,
        X = 550,
        Y = 170,
        Background = Brushes.LightGreen
    },
    new()
    {
        Id = "check_bulb",
        Text = "灯泡是否烧坏？",
        Shape = FlowShape.Diamond,
        X = 280,
        Y = 300,
        Width = 240,
        Height = 100,
        Background = Brushes.LightBlue
    },
    new()
    {
        Id = "replace_bulb",
        Text = "更换灯泡",
        Shape = FlowShape.Rectangle,
        X = 550,
        Y = 320,
        Background = Brushes.LightGreen
    },
    new()
    {
        Id = "repair",
        Text = "购买新台灯",
        Shape = FlowShape.Rectangle,
        X = 300,
        Y = 450,
        Background = Brushes.LightGreen
    }
};

public ObservableCollection<FlowEdge> FlowEdges { get; } = new()
{
    new() { SourceId = "start", TargetId = "check_plug" },
    new() { SourceId = "check_plug", TargetId = "plug_in", Label = "否" },
    new() { SourceId = "check_plug", TargetId = "check_bulb", Label = "是" },
    new() { SourceId = "check_bulb", TargetId = "replace_bulb", Label = "是" },
    new() { SourceId = "check_bulb", TargetId = "repair", Label = "否" }
};
```

## 通用属性 (`FlowChart`)

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Nodes` | 表示流程步骤的 `FlowNode` 项目集合。 | `null` |
| `Edges` | 表示连接的 `FlowEdge` 项目集合。 | `null` |
| `NodeCornerRadius` | 渲染节点框的圆角。 | `10.0` |
| `Groups` | 可选的 `FlowGroup` 容器集合。 | `null` |
| `FlowEdge.ShowArrow` | 连接是否以箭头结束。 | `true` |
