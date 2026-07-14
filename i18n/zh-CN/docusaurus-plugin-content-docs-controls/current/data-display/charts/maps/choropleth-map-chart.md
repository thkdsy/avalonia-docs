---
id: choropleth-map-chart
title: 等值线地图
description: 按统计变量比例对地理区域进行着色，用于可视化跨地区的数据密度、人口统计或市场表现。
doc-type: reference
tags:
  - avalonia pro
---

import chartsMapsChoropleth from '/img/controls/charts/charts-maps.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

等值线地图按统计变量比例对地理区域进行着色。它们适合可视化跨离散区域的数据密度或趋势。

<Image light={chartsMapsChoropleth} maxWidth={400} position="center" cornerRadius="true" alt="等值线地图，以不同颜色强度对地理区域进行着色，表示人口密度。" />

## 何时使用
- **人口统计**：显示人口密度、收入水平或投票模式。
- **市场渗透**：可视化不同地区的销售表现。
- **环境数据**：按区域显示气候数据或资源分布。

## 代码示例

### XAML
```xml
<ChoroplethMap xmlns="https://github.com/avaloniaui" Name="ChoroplethSample" Title="人口密度（包装器）" Height="400">
                        <ChoroplethMap.DataLayer>
                            <ShapeLayer GeoJson="{Binding WorldGeoJson}"
                                                 GeoJsonIdPath="ISO_A2"
                                                 MinValue="0"
                                                 MaxValue="500"
                                                 RegionPath="Code"
                                                 ValuePath="Density"
                                                 LowBrush="#E3F2FD"
                                                 HighBrush="#1565C0"
                                                 Stroke="#90A4AE"
                                                 ItemsSource="{Binding ShapeLayerData}">
                                <ShapeLayer.TooltipTemplate>
                                    <DataTemplate>
                                        <StackPanel Spacing="4">
                                            <TextBlock Text="{Binding Name}" FontWeight="Bold"/>
                                            <TextBlock Text="{Binding Density, StringFormat='密度: {0:N1} 人/km²'}"/>
                                        </StackPanel>
                                    </DataTemplate>
                                </ShapeLayer.TooltipTemplate>
                            </ShapeLayer>
                        </ChoroplethMap.DataLayer>
                    </ChoroplethMap>
```

### 数据模型 (C#)

确保 GeoJSON 文件已包含在项目中，并在运行时可在指定的相对路径下访问。

```csharp
using System.IO;

public record CountryDensityData(string Code, string Name, double Density);

public string WorldGeoJson { get; } =
    File.ReadAllText("Resources/ne_110m_world.geojson");

public CountryDensityData[] ShapeLayerData { get; } = new CountryDensityData[]
{
    new("IN", "印度", 464.0),
    new("BD", "孟加拉国", 1265.0),
    new("JP", "日本", 347.0),
    new("KR", "韩国", 527.0),
    new("NL", "荷兰", 508.0),
    new("BE", "比利时", 376.0),
    new("GB", "英国", 275.0),
    new("DE", "德国", 240.0),
    new("IT", "意大利", 206.0),
    new("FR", "法国", 119.0),
    new("CN", "中国", 153.0),
    new("US", "美国", 36.0),
    new("CA", "加拿大", 4.0),
    new("BR", "巴西", 25.0),
    new("RU", "俄罗斯", 9.0),
    new("AU", "澳大利亚", 3.0)
};
```

## 通用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `DataLayer` | 用于渲染等值线区域的规范 `ShapeLayer`。默认会自动创建一层。 | 自动创建的 `ShapeLayer` |

## 通用属性 (`DataLayer` / `ShapeLayer`)

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 区域数据项目的集合。 | `null` |
| `GeoJson` | GeoJSON 几何数据源。 | `null` |
| `GeoJsonIdPath` | GeoJSON 中用于标识区域的属性名称。 | `ISO_A2` |
| `RegionPath` | `ItemsSource` 中的链接属性名称。 | `null` |
| `ValuePath` | 用于颜色强度的数值属性名称。 | `null` |
| `MinValue` | 用于颜色标准化的最小值。 | `0.0` |
| `MaxValue` | 用于颜色标准化的最大值。 | `100.0` |
| `LowBrush` | 用于数据范围低端的画笔。 | `#E3F2FD` |
| `HighBrush` | 用于数据范围高端的画笔。 | `#1565C0` |
| `Stroke` | 用于区域边界的画笔。 | `null` |
| `TooltipTemplate` | 用于地图工具提示的数据模板。 | `null` |

## 另请参阅

- [形状地图](/controls/data-display/charts/maps/shape-map-chart)
- [气泡地图](/controls/data-display/charts/maps/bubble-map-chart)
