---
id: wind-rose-chart
title: 风玫瑰图
description: 以堆叠极坐标扇区显示方向频率分布，适用于风、交通和方向性事件分析。
doc-type: reference
tags:
  - avalonia pro
---

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

风玫瑰图按方向分组数值，并在每个方向扇区内堆叠子类别，如速度带。

## 何时使用

- **气象学**：按方向和速度带显示风的频率。
- **方向性事件**：比较围绕罗盘方向的交通、移动或信号发生情况。
- **操作分析**：在一个紧凑的极坐标图中汇总基于方向的活动。

## 代码示例

### XAML

```xml
<WindRoseChart xmlns="https://github.com/avaloniaui" Title="Wind distribution"
                                Height="320"
                                ItemsSource="{Binding WindData}"
                                DirectionPath="Direction"
                                SpeedPath="SpeedBand"
                                ValuePath="Frequency" />
```

### 数据模型（C#）

```csharp
public record WindSample(string Direction, string SpeedBand, double Frequency);

public ObservableCollection<WindSample> WindData { get; } = new()
{
    new("N", "0-10", 12),
    new("N", "10-20", 6),
    new("NE", "0-10", 8)
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 方向观测值的集合。 | `null` |
| `DirectionPath` | 方向分组的路径。 | `null` |
| `SpeedPath` | 每个方向内堆叠子分组的路径。 | `null` |
| `ValuePath` | 数值或频率的路径。 | `null` |
| `StartAngle` | 第一个扇区的起始角度（度）。 | `-90.0` |

## 另请参阅

- [极坐标图](/controls/data-display/charts/radial/polar-chart)
- [史密斯图](/controls/data-display/charts/engineering/smith-chart)
