---
id: flow-chart
title: 流程图
description: 将工作流和决策树可视化为节点和有向边，支持可选分组和自动布局。
doc-type: reference
tags:
  - avalonia pro
---

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

`FlowChart` 渲染用于工作流、流程图和决策树的节点、边和可选分组。当节点没有明确位置时，控件可以自动布局。

## 何时使用

- **流程图**：渲染逐步的业务或操作流程。
- **决策树**：显示分支、结果和带标签的转换。
- **系统映射**：将相关节点分组到有界区域中。

## 代码示例

### XAML

```xml
<FlowChart xmlns="https://github.com/avaloniaui" Title="故障排查"
                            Height="360"
                            Nodes="{Binding FlowNodes}"
                            Edges="{Binding FlowEdges}" />
```

### 数据模型 (C#)

```csharp
public ObservableCollection<FlowNode> FlowNodes { get; } = new()
{
    new() { Id = "start", Text = "灯不亮", Shape = FlowShape.RoundedRect },
    new() { Id = "power", Text = "检查电源", Shape = FlowShape.Rectangle },
    new() { Id = "done", Text = "更换灯泡", Shape = FlowShape.Diamond }
};

public ObservableCollection<FlowEdge> FlowEdges { get; } = new()
{
    new() { SourceId = "start", TargetId = "power" },
    new() { SourceId = "power", TargetId = "done", Label = "否" }
};
```

## 通用属性 (`FlowChart`)

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Nodes` | 图表渲染的 `FlowNode` 项目集合。 | `null` |
| `Edges` | `FlowEdge` 连接集合。 | `null` |
| `Groups` | 可选的 `FlowGroup` 容器集合。 | `null` |
| `NodeCornerRadius` | 应用于圆角节点形状的圆角半径。 | `10.0` |

## 通用属性 (`FlowNode`)

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Id` | 边和组使用的唯一节点标识符。 | `null` |
| `Text` | 节点内显示的文本。 | `null` |
| `Shape` | 节点形状，如 `Rectangle` 或 `Diamond`。 | `Rectangle` |
| `X` | 明确的 X 位置。 | `0` |
| `Y` | 明确的 Y 位置。 | `0` |
| `Width` | 节点宽度。 | `120` |
| `Height` | 节点高度。 | `60` |
| `Background` | 节点的可选背景画笔。 | `null` |
| `Foreground` | 节点的可选文本画笔。 | `null` |
| `Icon` | 节点内显示的可选图标。 | `null` |

## 通用属性 (`FlowEdge`)

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `SourceId` | 源节点标识符。 | `null` |
| `TargetId` | 目标节点标识符。 | `null` |
| `Label` | 可选的边标签。 | `null` |
| `ShowArrow` | 是否绘制箭头。 | `true` |

## 通用属性 (`FlowGroup`)

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Id` | 唯一的组标识符。 | `null` |
| `Label` | 为组显示的可选标签。 | `null` |
| `Bounds` | 组容器的明确边界。 | `0,0,0,0` |
| `Background` | 组的可选背景画笔。 | `null` |
| `BorderBrush` | 组的可选边框画笔。 | `null` |
| `BorderThickness` | 组边框的粗细。 | `1.0` |
| `NodeIds` | 属于该组的节点 ID 集合。 | `null` |

## 另请参阅

- [工艺流程图](/controls/data-display/charts/hierarchy/process-flow-chart)
- [思维导图](/controls/data-display/charts/hierarchy/mindmap-chart)
