---
id: tooltip-chart
title: 工具提示
description: 悬停时显示数据点的详细信息，支持使用 DataTemplate 自定义工具提示内容。
doc-type: reference
tags:
  - avalonia pro
---

import chartsFeaturesTooltip from '/img/controls/charts/charts-tooltips.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

工具提示在用户悬停数据点时提供详细信息。它们在不干扰主图表区域的情况下增加了精确度和上下文信息。

默认的笛卡尔和金融图表工具提示通过图表的水平轴格式显示分类和日期值。

<Image light={chartsFeaturesTooltip} maxWidth={400} position="center" cornerRadius="true" alt="带有交互式工具提示弹出窗口的图表，在悬停时显示数据点的精确值和分类。" />

## 使用时机
- **高密度数据**：在拥挤的折线图或散点图中精确定位数值。
- **额外上下文**：显示未映射到轴的元数据（例如"更新日期"）。
- **悬停交互**：指针在数据点上时显示详细信息。

## 代码示例

### XAML
```xml
<CartesianChart xmlns="https://github.com/avaloniaui" Name="PerSeriesTooltipsChart" Height="250" IsTooltipEnabled="True">
    <CartesianChart.HorizontalAxis>
        <CategoryAxis Title="Quarter" />
    </CartesianChart.HorizontalAxis>
    <CartesianChart.VerticalAxis>
        <NumericalAxis Title="Revenue" />
    </CartesianChart.VerticalAxis>
    <CartesianChart.Series>
        <BarSeries Title="2023（工具提示开启）"
                          ItemsSource="{Binding Series1Data}"
                          IsTooltipEnabled="True" />
        <BarSeries Title="2024（工具提示关闭）"
                          ItemsSource="{Binding Series2Data}"
                          IsTooltipEnabled="False" />
    </CartesianChart.Series>
</CartesianChart>
```

### 数据模型 (C#)
```csharp
public ObservableCollection<double> Series1Data { get; } = new()
{
    120, 150, 180, 200
};

public ObservableCollection<double> Series2Data { get; } = new()
{
    140, 170, 160, 220
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `IsTooltipEnabled` | 工具提示可见性的全局切换。 | `true` |
| `TooltipTemplate` | 工具提示 UI 的自定义 DataTemplate（在系列上）。 | 系统默认值 |

## 数据项约定

如果命中数据项有一个名为 `TooltipText` 的非空字符串属性，默认工具提示将显示该文本，而不是生成的系列、分类和值内容。

当工具提示需要自定义布局、控件或多个绑定字段时，在系列上使用 `TooltipTemplate`。
