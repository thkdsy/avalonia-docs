---
id: venn-diagram-chart
title: 韦恩图
description: 可视化集合重叠和交集值，适用于比较成员关系和共享段。
doc-type: reference
tags:
  - avalonia pro
---

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

韦恩图显示集合如何重叠、哪些区域是唯一的，以及每个交集拥有多少值。

## 何时使用

- **集合比较**：显示产品受众、功能或实验之间的重叠。
- **共享成员关系**：突出显示唯一和交叉的段。
- **选择工作流**：允许用户在图中的区域进行检查或选择。

## 代码示例

### XAML

```xml
<VennDiagramChart xmlns="https://github.com/avaloniaui" Title="受众重叠"
                                   Height="320"
                                   ItemsSource="{Binding VennItems}"
                                   IsSelectionEnabled="True" />
```

### 数据模型 (C#)

```csharp
public ObservableCollection<VennItem> VennItems { get; } = new()
{
    new() { Name = "新闻通讯", Sets = ["A"], Value = 45 },
    new() { Name = "网络研讨会", Sets = ["B"], Value = 30 },
    new() { Name = "客户", Sets = ["A", "B"], Value = 18 }
};
```

## 常用属性（`VennDiagramChart`）

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | `VennItem` 区域的集合。 | `null` |
| `IsSelectionEnabled` | 是否可以选择区域。 | `false` |
| `SelectionMode` | 选择行为。 | `SingleDeselect` |
| `SelectionBrush` | 用于选中区域的画刷。 | 取决于主题 |
| `SelectionStroke` | 用于选中区域的描边。 | `null` |
| `SelectionStrokeThickness` | 用于选中区域的描边粗细。 | `2.0` |
| `SelectedIndex` | 选中区域的索引，未选择时为 `-1`。 | `-1` |

## 常用属性（`VennItem`）

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Sets` | 集合标识符的集合，例如 `["A"]` 或 `["A", "B"]`。 | 空集合 |
| `Value` | 集合或交集表示的数值。 | `0.0` |
| `Name` | 可选区域标签。 | `null` |
| `Fill` | 区域的可选填充画刷。 | `null` |

## 另请参阅

- [圆堆积图](/controls/data-display/charts/hierarchy/circle-packing-chart)
- [议会图](/controls/data-display/charts/comparison/parliament-chart)
