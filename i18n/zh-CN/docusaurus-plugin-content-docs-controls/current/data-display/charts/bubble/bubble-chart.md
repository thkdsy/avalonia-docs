---
id: bubble-chart
title: 气泡图
description: 像散点图一样绘制 X 和 Y 值，第三个值编码为气泡大小。
doc-type: reference
tags:
  - avalonia pro
---

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

气泡图使用 `CartesianChart` 中的 `BubbleSeries` 绘制两个数值维度。每个标记使用第三个度量值来确定大小。

## 使用场景

- **三变量对比**：在一个视图中展示 X、Y 和数值大小之间的关系。
- **投资组合分析**：同时比较价格、利润和成交量。
- **机会地图**：通过大小和位置突出异常值。

## 代码示例

### XAML

```xml
<CartesianChart xmlns="https://github.com/avaloniaui" Title="Product portfolio" Height="300">
    <CartesianChart.HorizontalAxis>
        <NumericalAxis Title="Price" />
    </CartesianChart.HorizontalAxis>
    <CartesianChart.VerticalAxis>
        <NumericalAxis Title="Revenue" />
    </CartesianChart.VerticalAxis>
    <CartesianChart.Series>
        <BubbleSeries ItemsSource="{Binding BubbleData}"
                               CategoryPath="Price"
                               ValuePath="Revenue"
                               SizePath="Units"
                               Title="Products" />
    </CartesianChart.Series>
</CartesianChart>
```

### 数据模型 (C#)

```csharp
public record ProductBubble(double Price, double Revenue, double Units);

public ObservableCollection<ProductBubble> BubbleData { get; } = new()
{
    new(15, 120, 40),
    new(22, 180, 65),
    new(30, 140, 25)
};
```

## 常用属性 (`BubbleSeries`)

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 数据点的集合。 | `null` |
| `CategoryPath` | X 轴值的路径。 | `null` |
| `ValuePath` | Y 轴值的路径。 | `null` |
| `SizePath` | 用于气泡大小的值的路径。 | `null` |
| `MinBubbleSize` | 气泡最小直径（像素）。 | `10.0` |
| `MaxBubbleSize` | 气泡最大直径（像素）。 | `50.0` |
| `Fill` | 用于气泡的画笔。 | 取决于主题 |
| `Stroke` | 用于气泡轮廓的画笔。 | 取决于主题 |

## 另请参阅

- [散点图](/controls/data-display/charts/cartesian/scatter-chart)
- [打包气泡图](/controls/data-display/charts/bubble/packed-bubble-chart)
