---
id: kagi-chart
title: Kagi 图
description: 与时间无关的图表，使用垂直线跟踪价格变动，仅在价格超过设定的反转量时改变方向。
doc-type: reference
tags:
  - avalonia pro
---

import chartsFinancialKagi from '/img/controls/charts/charts-financial-kagi.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

Kagi 图是一种与时间无关的图表，使用垂直线跟踪价格变动。它们仅在价格达到特定的反转量时改变方向（和线条粗细）。

<Image light={chartsFinancialKagi} maxWidth={400} position="center" cornerRadius="true" alt="Kagi 图，使用粗线和细线组成垂直线条，仅在价格按设定量反转时改变方向。" />

## 何时使用
- **纯粹的价格行为**：关注价格变化，忽略时间或成交量。
- **突破识别**：使用"阳"（粗）线和"阴"（细）线发现反转。
- **趋势跟踪**：过滤掉未达到反转阈值的小幅波动。

## 代码示例

### XAML
```xml
<KagiChart xmlns="https://github.com/avaloniaui" Name="KagiChartSample" Title="Trend Reversal" Height="300"
                                        ReversalAmount="4"
                                        ItemsSource="{Binding KagiData}"
                                        ValuePath="Value" />
```

### 数据模型（C#）
```csharp
using System;

public record KagiPoint(double Value);

public ObservableCollection<KagiPoint> KagiData { get; } = new(CreateKagiData());

private static IEnumerable<KagiPoint> CreateKagiData()
{
    var price = 100.0;
    var random = new Random(123);

    for (var i = 0; i < 100; i++)
    {
        price += (random.NextDouble() - 0.5) * 4;
        yield return new KagiPoint(price);
    }
}
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 原始价格点的集合。 | `null` |
| `ValuePath` | 表示价格的属性。 | `null` |
| `ReversalAmount`| 翻转方向所需的最小价格变动。 | `1.0` |
| `YangBrush` | 阳线段的画笔（上涨超过前高）。 | `Green` |
| `YinBrush` | 阴线段的画笔（下跌超过前低）。 | `Red` |
| `StrokeThickness` | 基础线条粗细。 | `2.0` |
