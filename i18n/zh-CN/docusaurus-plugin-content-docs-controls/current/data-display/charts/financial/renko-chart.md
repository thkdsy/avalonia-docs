---
id: renko-chart
title: Renko 图
description: 使用固定大小的砖块表示价格变动，过滤时间和小幅波动以澄清趋势及支撑/阻力位。
doc-type: reference
tags:
  - avalonia pro
---

import chartsFinancialRenko from '/img/controls/charts/charts-financial-renko.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

Renko 图由"砖块"组成，每个砖块代表一个固定的价格变动。只有当价格变动达到指定的砖块大小时，才会添加新的砖块，从而过滤时间和小幅波动。

<Image light={chartsFinancialRenko} maxWidth={400} position="center" cornerRadius="true" alt="Renko 图，显示固定大小的绿色和红色砖块，代表超过设定阈值的价格变动。" />

## 何时使用
- **支撑和阻力**：识别砖块频繁反转的清晰水平位。
- **趋势确认**：发现持续的看涨/看跌砖块序列。
- **清晰可视化**：将复杂嘈杂的价格数据简化为统一的块状图。

## 代码示例

### XAML
```xml
<RenkoChart xmlns="https://github.com/avaloniaui" Name="RenkoChartSample" Title="Price Movement" Height="300" BrickSize="5"
                                         ItemsSource="{Binding RenkoData}"
                                         ValuePath="Value" />
```

### 数据模型（C#）
```csharp
public record RenkoPoint(double Value);

public ObservableCollection<RenkoPoint> RenkoData { get; } = new()
{
    new(100), new(105), new(103), new(108), new(115),
    new(112), new(118), new(120), new(115), new(122),
    new(118), new(114), new(110), new(105), new(100)
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 价格数据的集合。 | `null` |
| `BrickSize` | 新砖块所需的价格变动。 | `10.0` |
| `ValuePath` | 价格值的属性名称。 | `null` |
| `UpBrush` | 上涨趋势的颜色。 | `Green` |
| `DownBrush` | 下跌趋势的颜色。 | `Red` |
