---
id: parliament-chart
title: 议会图
description: 在半圆形布局中显示席位分布，适用于立法机构构成和比例代表制。
doc-type: reference
tags:
  - avalonia pro
---

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

议会图在半圆中排列席位，使政党代表可以同时作为总数和空间平衡来解读。

## 何时使用

- **立法机构构成**：显示各政党或集团席位的分布。
- **董事会代表**：可视化委员会或理事会成员组成。
- **比例分配**：以熟悉的半圆形布局呈现基于席位的结果。

## 代码示例

### XAML

```xml
<ParliamentChart xmlns="https://github.com/avaloniaui" Title="议会席位"
                                  Height="320"
                                  TotalSeats="120"
                                  Parties="{Binding Parties}" />
```

### 数据模型 (C#)

```csharp
public ObservableCollection<ParliamentParty> Parties { get; } = new()
{
    new() { Name = "联盟党", Seats = 46, Color = Colors.SteelBlue },
    new() { Name = "联合党", Seats = 38, Color = Colors.OrangeRed },
    new() { Name = "绿党", Seats = 22, Color = Colors.ForestGreen },
    new() { Name = "独立党", Seats = 14, Color = Colors.Gray }
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `TotalSeats` | 要绘制的总席位数。 | `100` |
| `Rows` | 同心席位行数。 | `4` |
| `InnerRadiusFactor` | 半圆的内半径因子。 | `0.4` |
| `SeatGap` | 席位之间的间隙。 | `2.0` |
| `StartAngle` | 半圆左边缘的起始角度（度）。 | `180.0` |
| `EndAngle` | 半圆右边缘的结束角度（度）。 | `0.0` |
| `Parties` | 定义席位分配的 `ParliamentParty` 项集合。 | `null` |

## 常用属性（`ParliamentParty`）

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Name` | 政党或集团名称。 | `string.Empty` |
| `Seats` | 分配给该政党的席位数。 | `0` |
| `Color` | 用于绘制该政党席位的颜色。 | `Gray` |

## 另请参阅

- [饼图](/controls/data-display/charts/circular/pie-chart)
- [马赛克图](/controls/data-display/charts/comparison/mekko-chart)
