---
id: ternary-chart
title: 三元图
description: 绘制总和为常数的三部分组成数据，适用于混合物、资源分配和分类平衡分析。
doc-type: reference
tags:
  - avalonia pro
---

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

三元图可视化三组分混合物，每个点代表 A、B 和 C 的相对贡献。

## 何时使用

- **混合物分析**：展示混合物、材料或资源分配的成分。
- **三向平衡**：比较总和始终为整体的比例。
- **科学分类**：在三角形决策空间中放置样本。

## 代码示例

### XAML

```xml
<TernaryChart xmlns="https://github.com/avaloniaui" Title="Mixture composition"
                               Height="320"
                               ItemsSource="{Binding TernaryData}"
                               APath="Sand"
                               BPath="Silt"
                               CPath="Clay"
                               ALabel="Sand"
                               BLabel="Silt"
                               CLabel="Clay" />
```

### 数据模型（C#）

```csharp
public record TernaryPoint(double Sand, double Silt, double Clay);

public ObservableCollection<TernaryPoint> TernaryData { get; } = new()
{
    new(60, 25, 15),
    new(35, 45, 20),
    new(20, 30, 50)
};
```

## 常用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 三元点的集合。 | `null` |
| `APath` | 第一个分量值的路径。 | `null` |
| `BPath` | 第二个分量值的路径。 | `null` |
| `CPath` | 第三个分量值的路径。 | `null` |
| `ALabel` | 轴 A 的标签。 | `null` |
| `BLabel` | 轴 B 的标签。 | `null` |
| `CLabel` | 轴 C 的标签。 | `null` |
| `ShowGridLines` | 是否绘制三元网格。 | `true` |
| `DotSize` | 绘制点的半径。 | `5.0` |

## 另请参阅

- [地毯图](/controls/data-display/charts/engineering/carpet-plot-chart)
- [平行坐标图](/controls/data-display/charts/statistical/parallel-coordinates-chart)
