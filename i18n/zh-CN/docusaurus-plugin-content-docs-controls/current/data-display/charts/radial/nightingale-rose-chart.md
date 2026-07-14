---
id: nightingale-rose-chart
title: 南丁格尔玫瑰图
description: 具有相等角度和可变半径的极坐标面积图，用于在圆形布局中比较类别大小。
doc-type: reference
tags:
  - avalonia pro
---

import chartsRadialRose from '/img/controls/charts/charts-radial-rose.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

南丁格尔玫瑰图是一种具有相等角度和可变半径的极坐标面积图。当你想要柱状图的圆形替代方案同时保持每个类别一个值时，它非常有用。

<Image light={chartsRadialRose} maxWidth={400} position="center" cornerRadius="true" alt="南丁格尔玫瑰图，每个圆形切片中具有堆叠子段，比较周期性类别的构成。" />

## 何时使用
- **季节性总结**：比较跨月或季度的一项测量值。
- **类别比较**：显示相对大小，无需使用笛卡尔轴。
- **圆形仪表板**：当类别形成循环时使用径向布局。

## 代码示例

### XAML
```xml
<NightingaleRoseChart xmlns="https://github.com/avaloniaui" Name="NightingaleRoseSample"
                               Title="月度销售"
                               Height="350"
                               ShowLabels="True"
                               ItemsSource="{Binding NightingaleData}"
                               ValuePath="Value"
                               LabelPath="Label" />
```

### 数据模型 (C#)
```csharp
public record RadialPoint(string Label, double Value);

public ObservableCollection<RadialPoint> NightingaleData { get; } = new()
{
    new("一月", 120.0), new("二月", 180.0), new("三月", 160.0),
    new("四月", 200.0), new("五月", 280.0), new("六月", 350.0),
    new("七月", 380.0), new("八月", 340.0), new("九月", 250.0),
    new("十月", 180.0), new("十一月", 140.0), new("十二月", 160.0)
};
```

## 通用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 段的集合。 | `null` |
| `ValuePath` | 每个段的半径大小。 | `null` |
| `LabelPath` | 每个段的类别标签。 | `null` |
| `InnerRadiusFactor` | 内半径比率，`0.0` 创建实心玫瑰图。 | `0.0` |
| `ShowLabels` | 是否显示段标签。 | `true` |
| `ShowValues` | 是否在标签旁显示数值。 | `false` |
| `IsHighlightEnabled` | 启用悬停高亮显示段。 | `false` |
