---
id: mirror-bar-chart
title: 镜像柱状图
description: 围绕中心线将两个柱状系列背靠背放置，适用于人口统计和并排比较。
doc-type: reference
tags:
  - avalonia pro
---

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

镜像柱状图在中心线的两侧绘制两个柱状系列，使每个品类可以进行对称比较。

## 何时使用

- **背靠背比较**：比较每个品类的两个群体、区域或产品组。
- **人口统计布局**：显示对立分布，如男性和女性按年龄段分布。
- **双侧排名**：对比配对测量值，无需堆叠或分组柱条。

## 代码示例

### XAML

```xml
<MirrorBarChart xmlns="https://github.com/avaloniaui" Title="区域比较"
                         Height="320"
                         ItemsSource="{Binding MirrorData}"
                         LabelPath="Category"
                         LeftValuePath="West"
                         RightValuePath="East"
                         LeftTitle="西部"
                         RightTitle="东部" />
```

### 数据模型 (C#)

```csharp
public record MirrorBarItem(string Category, double West, double East);

public ObservableCollection<MirrorBarItem> MirrorData { get; } = new()
{
    new("Q1", 22, 18),
    new("Q2", 28, 24),
    new("Q3", 19, 27)
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 镜像比较项的集合。 | `null` |
| `LeftValuePath` | 在左侧渲染的值的路径。 | `null` |
| `RightValuePath` | 在右侧渲染的值的路径。 | `null` |
| `LabelPath` | 品类标签的路径。 | `null` |
| `LeftBrush` | 用于左侧柱条的画刷。 | `#E91E63` |
| `RightBrush` | 用于右侧柱条的画刷。 | `#2196F3` |
| `BarHeight` | 柱条高度，以行高的比例表示。 | `0.7` |
| `CenterGap` | 两个镜像侧之间的间隙。 | `40.0` |
| `LeftTitle` | 左侧的可选标题。 | `null` |
| `RightTitle` | 右侧的可选标题。 | `null` |
| `IsHighlightEnabled` | 启用镜像柱条的悬停高亮。 | `false` |

## 另请参阅

- [人口金字塔图](/controls/data-display/charts/comparison/population-pyramid-chart)
- [龙卷风图](/controls/data-display/charts/comparison/tornado-chart)
