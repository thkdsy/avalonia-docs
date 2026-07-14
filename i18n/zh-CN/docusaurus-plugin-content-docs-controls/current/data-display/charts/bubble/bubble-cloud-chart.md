---
id: bubble-cloud-chart
title: 气泡云图
description: 以有机的紧凑布局排列带大小的气泡，无需坐标轴，适用于排名类别和注意力图。
doc-type: reference
tags:
  - avalonia pro
---

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

气泡云图将气泡放置在聚集的、无坐标轴的布局中，气泡大小承载主要的定量含义。

## 使用场景

- **类别强调**：无需精确坐标轴即可展示哪些类别占主导地位。
- **注意力图**：以紧凑的可视化形式呈现重要主题、细分市场或产品。
- **仪表板磁贴**：作为柱状图或表格的更具有机感的替代方案。

## 代码示例

### XAML

```xml
<BubbleCloud xmlns="https://github.com/avaloniaui" Title="Topic volume"
                      Height="320"
                      ItemsSource="{Binding Topics}"
                      LabelPath="Name"
                      ValuePath="Count" />
```

### 数据模型 (C#)

```csharp
public record TopicBubble(string Name, double Count);

public ObservableCollection<TopicBubble> Topics { get; } = new()
{
    new("Support", 120),
    new("Billing", 75),
    new("Shipping", 55)
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 气泡项的集合。 | `null` |
| `LabelPath` | 气泡标签的路径。 | `null` |
| `ValuePath` | 用于气泡大小的值的路径。 | `null` |
| `MinBubbleSize` | 气泡最小半径（像素）。 | `30.0` |
| `MaxBubbleSize` | 气泡最大半径（像素）。 | `80.0` |
| `IsHighlightEnabled` | 启用气泡的悬停高亮。 | `false` |

## 另请参阅

- [打包气泡图](/controls/data-display/charts/bubble/packed-bubble-chart)
- [词云图](/controls/data-display/charts/analytics/word-cloud-chart)
