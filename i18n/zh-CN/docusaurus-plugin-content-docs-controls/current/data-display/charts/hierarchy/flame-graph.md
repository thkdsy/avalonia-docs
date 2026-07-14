---
id: flame-graph
title: 火焰图
description: 从下到上可视化层次栈或调用数据，常用于性能分析和跟踪分析。
doc-type: reference
tags:
  - avalonia pro
---

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

火焰图以堆叠矩形显示层次化的开销、持续时间或样本数据，根在底部，更深层的调用在其上方。

## 何时使用

- **性能分析输出**：显示调用树中的 CPU、内存或持续时间热点。
- **跟踪检查**：了解时间在嵌套操作中的累积位置。
- **层次开销审查**：比较昂贵分支的深度和宽度。

## 代码示例

### XAML

```xml
<FlameGraph xmlns="https://github.com/avaloniaui" Title="CPU 性能分析"
                             Height="320"
                             ItemsSource="{Binding StackTraceData}"
                             ValuePath="Duration"
                             LabelPath="MethodName"
                             ChildrenPath="SubCalls" />
```

### 数据模型 (C#)

```csharp
public class FlameNode
{
    public string MethodName { get; set; } = string.Empty;
    public double Duration { get; set; }
    public ObservableCollection<FlameNode> SubCalls { get; set; } = new();
}

public ObservableCollection<FlameNode> StackTraceData { get; } = new()
{
    new FlameNode
    {
        MethodName = "Program.Main",
        Duration = 2000,
        SubCalls =
        {
            new FlameNode
            {
                MethodName = "App.OnFrameworkInitializationCompleted",
                Duration = 1950,
                SubCalls =
                {
                    new FlameNode { MethodName = "App.Initialize", Duration = 50 },
                    new FlameNode { MethodName = "Window.Show", Duration = 100 },
                    new FlameNode { MethodName = "Dispatcher.MainLoop", Duration = 1800 }
                }
            }
        }
    }
};
```

## 通用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 火焰图的根集合。 | `null` |
| `ValuePath` | 控制矩形宽度的值的路径。 | `null` |
| `LabelPath` | 项目标签的路径。 | `null` |
| `ChildrenPath` | 子集合的路径。 | `null` |

## 另请参阅

- [树图](/controls/data-display/charts/hierarchy/treemap-chart)
- [流程图](/controls/data-display/charts/hierarchy/flow-chart)
