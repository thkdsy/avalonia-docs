---
id: polar-area-chart
title: 极坐标面积图
description: 类似于饼图，但使用段半径而非角度来编码数值，在圆形轴上为每个类别提供相等的角度空间。
doc-type: reference
tags:
  - avalonia pro
---

import chartsRadialPolararea from '/img/controls/charts/charts-radial-polar.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

极坐标面积图（或称梳状图）类似于饼图，但使用段的半径而非角度来表示值。每个段具有相等的角度。

<Image light={chartsRadialPolararea} maxWidth={400} position="center" cornerRadius="true" alt="极坐标面积图，具有相等角度的段和不同半径，在圆形布局中表示季节性值大小。" />

## 何时使用
- **周期性趋势**：可视化季节数据或风模式。
- **排名类别**：在圆形布局中比较多个类别的大小。
- **历史分析**：可视化随时间推移的死亡率原因等经典图表类型。

## 代码示例

### XAML
```xml
<PolarAreaChart xmlns="https://github.com/avaloniaui" Name="PolarAreaChartSample" Title="技能水平" Height="300"
                                             ItemsSource="{Binding PolarChartData}"
                                             LabelPath="Label" ValuePath="Value" />
```

### 数据模型 (C#)
```csharp
public record RadialPoint(string Label, double Value);

public ObservableCollection<RadialPoint> PolarChartData { get; } = new()
{
    new("速度", 85),
    new("力量", 70),
    new("敏捷", 60),
    new("智力", 75),
    new("耐力", 90),
    new("精神", 65)
};
```

## 通用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 段的集合。 | `null` |
| `ValuePath` | 决定切片半径的属性。 | `null` |
| `LabelPath` | 切片名称的属性。 | `null` |
| `ShowLabels` | 是否显示段标签。 | `true` |
| `LabelFontSize` | 段标签使用的字号。 | `11.0` |
| `LabelForeground` | 用于段标签的画笔。为 `null` 时，图表使用有效的标签前景色。 | `null` |
| `StartAngle` | 第一个段的起始角度（度）。 | `-90.0` |
| `Stroke` | 段的轮廓画笔。为 `null` 时，图表使用白色轮廓。 | `null`（白色） |
| `StrokeThickness` | 段轮廓的粗细。 | `1.0` |
| `IsHighlightEnabled` | 启用极坐标面积段的悬停高亮。 | `false` |
