---
id: stacked-100-percent-bar-chart
title: 堆叠 100% 柱状图
description: 显示始终填满至 100% 的柱条，展示每个品类中各系列的相对比例，而非绝对数值。
doc-type: reference
tags:
  - avalonia pro
---

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

堆叠 100% 柱状图显示始终延伸到相同高度的柱条，展示每个品类的百分比组成。每个段代表一个系列的比例份额，使读者可以比较相对贡献。

## 何时使用
- **比例比较**：比较不同段在各品类中对整体的贡献。
- **市场份额**：展示跨部门的相对市场份额或预算分配。
- **调查结果**：显示每个问题的百分比细分，如同意/不同意回应。

## 代码示例

### XAML
```xml
<CartesianChart xmlns="https://github.com/avaloniaui" Title="浏览器市场份额" Height="250">
    <CartesianChart.HorizontalAxis>
        <CategoryAxis />
    </CartesianChart.HorizontalAxis>
    <CartesianChart.VerticalAxis>
        <NumericalAxis />
    </CartesianChart.VerticalAxis>
    <CartesianChart.Series>
        <Stacked100PercentBarSeries Title="Chrome"
                                              ItemsSource="{Binding ChromeData}"
                                              CategoryPath="Year"
                                              ValuePath="Share"
                                              StackGroup="browsers" />
        <Stacked100PercentBarSeries Title="Firefox"
                                              ItemsSource="{Binding FirefoxData}"
                                              CategoryPath="Year"
                                              ValuePath="Share"
                                              StackGroup="browsers" />
        <Stacked100PercentBarSeries Title="Safari"
                                              ItemsSource="{Binding SafariData}"
                                              CategoryPath="Year"
                                              ValuePath="Share"
                                              StackGroup="browsers" />
    </CartesianChart.Series>
</CartesianChart>
```

### 数据模型 (C#)
```csharp
public record BrowserShare(string Year, double Share);

public ObservableCollection<BrowserShare> ChromeData { get; } = new()
{
    new("2022", 65), new("2023", 63), new("2024", 66)
};

public ObservableCollection<BrowserShare> FirefoxData { get; } = new()
{
    new("2022", 20), new("2023", 18), new("2024", 15)
};

public ObservableCollection<BrowserShare> SafariData { get; } = new()
{
    new("2022", 15), new("2023", 19), new("2024", 19)
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Title` | 图例中显示的系列名称。 | `null` |
| `ItemsSource` | 要显示的数据项集合。 | `null` |
| `CategoryPath` | 用于 X 轴的属性路径。 | `null` |
| `ValuePath` | 用于 Y 轴的属性路径。 | `null` |
| `Fill` | 用于填充柱子的颜色/画刷。 | 取决于主题 |
| `Stroke` | 柱子的轮廓颜色。 | `Transparent` |
| `StackGroup` | 用于将多个系列合并到一个堆叠柱条中的组名称。 | `"default"` |
| `BarWidth` | 每个柱子的宽度，以分类带宽度的比例表示（0.0 到 1.0）。 | `0.7` |
| `BarCornerRadius` | 柱子圆角半径。 | `0` |

## 另请参阅

- [柱状图](/controls/data-display/charts/cartesian/bar-chart)
- [堆叠柱状图](/controls/data-display/charts/cartesian/stacked-bar-chart)
- [组合图](/controls/data-display/charts/cartesian/combo-chart)
