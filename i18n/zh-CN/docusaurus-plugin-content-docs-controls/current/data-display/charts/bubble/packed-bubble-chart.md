---
id: packed-bubble-chart
title: 打包气泡图
description: 在紧凑区域内紧密排列带大小的气泡，无需坐标轴，适用于比例化的类别比较。
doc-type: reference
tags:
  - avalonia pro
---

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

打包气泡图通过气泡大小展示类别量级，同时将圆圈紧密打包在一个单一框架中。

## 使用场景

- **部分与整体视图**：无需柱状图即可按大小比较类别。
- **紧凑仪表板**：将许多类别适配到一个方形卡片中。
- **标签优先比较**：当空间允许时，将类别名称保留在气泡内部。

## 代码示例

### XAML

```xml
<PackedBubbleChart xmlns="https://github.com/avaloniaui" Title="Segment size"
                            Height="320"
                            ItemsSource="{Binding Segments}"
                            LabelPath="Name"
                            ValuePath="Value"
                            ShowLabels="True" />
```

### 数据模型 (C#)

```csharp
public record SegmentBubble(string Name, double Value);

public ObservableCollection<SegmentBubble> Segments { get; } = new()
{
    new("Desktop", 42),
    new("Mobile", 30),
    new("Web", 18)
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 气泡项的集合。 | `null` |
| `ValuePath` | 用于气泡大小的值的路径。 | `null` |
| `LabelPath` | 气泡标签的路径。 | `null` |
| `MinBubbleSize` | 气泡最小大小（像素）。 | `20.0` |
| `MaxBubbleSize` | 气泡最大大小（像素）。 | `80.0` |
| `ShowLabels` | 是否在空间允许时在气泡内部绘制标签。 | `true` |
| `LabelFontSize` | 气泡标签的基准字体大小。实际渲染大小受每个气泡半径限制。 | `12.0` |
| `LabelForeground` | 用于气泡标签的画笔。当为 `null` 时，标签使用白色。 | `null` |
| `IsHighlightEnabled` | 启用气泡的悬停高亮。 | `false` |

## 另请参阅

- [气泡云图](/controls/data-display/charts/bubble/bubble-cloud-chart)
- [气泡图](/controls/data-display/charts/bubble/bubble-chart)
