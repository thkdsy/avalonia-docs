---
id: variance-chart
title: 差异图
description: 显示从基线向上和向下延伸的柱条，高于和低于参考线的值使用不同颜色。
doc-type: reference
tags:
  - avalonia pro
---

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

差异图显示在基线值上下延伸的柱条，对正向和负向偏差使用不同颜色。它们显示值在何处超出或未达到目标或参考点。

## 何时使用
- **预算分析**：显示实际支出与计划支出的对比，超支和节约使用颜色编码。
- **绩效跟踪**：可视化各品类 KPI 与目标的偏差。
- **损益分析**：显示相对于盈亏平衡点的收益和损失。

## 代码示例

### XAML
```xml
<CartesianChart xmlns="https://github.com/avaloniaui" Title="月度损益" Height="250">
    <CartesianChart.HorizontalAxis>
        <CategoryAxis />
    </CartesianChart.HorizontalAxis>
    <CartesianChart.VerticalAxis>
        <NumericalAxis />
    </CartesianChart.VerticalAxis>
    <CartesianChart.Series>
        <VarianceSeries Title="损益"
                                 ItemsSource="{Binding ProfitData}"
                                 CategoryPath="Month"
                                 ValuePath="Amount"
                                 Baseline="0"
                                 PositiveBrush="Green"
                                 NegativeBrush="Red"
                                 BarWidth="0.6" />
    </CartesianChart.Series>
</CartesianChart>
```

### 数据模型 (C#)
```csharp
public record ProfitItem(string Month, double Amount);

public ObservableCollection<ProfitItem> ProfitData { get; } = new()
{
    new("一月", 150),
    new("二月", -80),
    new("三月", 220),
    new("四月", -30),
    new("五月", 180),
    new("六月", -120)
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Title` | 图例中显示的系列名称。 | `null` |
| `ItemsSource` | 要显示的数据项集合。 | `null` |
| `CategoryPath` | 用于 X 轴的属性路径。 | `null` |
| `ValuePath` | 用于 Y 轴的属性路径。 | `null` |
| `Baseline` | 区分正向和负向差异的参考值。 | `0` |
| `PositiveBrush` | 用于基线上方柱条的画刷。 | `null` |
| `NegativeBrush` | 用于基线下方柱条的画刷。 | `null` |
| `BarWidth` | 每个柱子的宽度，以分类带宽度的比例表示（0.0 到 1.0）。 | `0.6` |
| `BarCornerRadius` | 柱子圆角半径。 | `2` |

## 另请参阅

- [柱状图](/controls/data-display/charts/cartesian/bar-chart)
- [范围柱状图](/controls/data-display/charts/cartesian/range-bar-chart)
- [瀑布图](/controls/data-display/charts/cartesian/waterfall-chart)
