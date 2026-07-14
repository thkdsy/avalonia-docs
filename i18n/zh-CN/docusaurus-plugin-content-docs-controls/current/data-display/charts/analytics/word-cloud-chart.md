---
id: word-cloud-chart
title: 词云
description: 根据词频或重要性改变单词大小来呈现文本数据，提供定性内容的可视化摘要。
doc-type: reference
tags:
  - avalonia pro
---

import chartsAnalyticsWordCloud from '/img/controls/charts/charts-analytics-word-cloud.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

词云根据词频或重要性改变单词大小来呈现文本数据。它们为定性内容或热门话题提供了即时的可视化摘要。

<Image light={chartsAnalyticsWordCloud} maxWidth={400} position="center" cornerRadius="true" alt="词云，根据词频以不同字体大小显示单词，更突出的单词显示得更大。" />

## 使用场景
- **搜索趋势**：可视化查询日志中最常见的关键词。
- **情感分析**：突出显示客户评论中的显著词语。
- **内容摘要**：展示长文章或文档的主要主题。

## 代码示例

### XAML
```xml
<WordCloudChart xmlns="https://github.com/avaloniaui" Title="Popular Topics" Height="300"
                                             ItemsSource="{Binding WordCloudData}"
                                             WordPath="Word" WeightPath="Count"/>
```

### 数据模型 (C#)
```csharp
public record WordItem(string Word, double Count);

public ObservableCollection<WordItem> WordCloudData { get; } = new()
{
    new("Technology", 80),
    new("Innovation", 65),
    new("Digital", 55),
    new("Cloud", 50),
    new("AI", 70),
    new("Data", 60),
    new("Security", 45),
    new("Mobile", 40),
    new("Development", 35),
    new("Analytics", 30),
    new("Performance", 25),
    new("User", 45),
    new("Interface", 40)
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 单词/权重数据的集合。 | `null` |
| `WordPath` | 实际文本单词的属性名。 | `null` |
| `WeightPath` | 决定字体大小的数值属性。 | `null` |
| `MinFontSize` | 最小字体大小（像素）。 | `12.0` |
| `MaxFontSize` | 最大字体大小（像素）。 | `48.0` |
| `MaxWords` | 渲染的最大单词数。 | `50` |
| `RotateWords` | 是否允许部分单词垂直旋转。 | `true` |
