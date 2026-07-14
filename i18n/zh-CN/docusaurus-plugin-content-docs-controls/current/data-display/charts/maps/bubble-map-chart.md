---
id: bubble-map-chart
title: 气泡地图
description: 在地理区域上叠加按比例大小的圆圈来表示数据值，同时显示位置和值的大小。
doc-type: reference
tags:
  - avalonia pro
---

import chartsMapsBubble from '/img/controls/charts/charts-maps-bubble.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

气泡地图使用不同大小的圆圈来表示地理区域上的数据值，以 `ShapeMap` 为基础显示 `ShapeLayer` 并叠加 `BubbleLayer`。它们在同一视图中显示位置和值的大小。

<Image light={chartsMapsBubble} maxWidth={400} position="center" cornerRadius="true" alt="气泡地图，将不同大小的圆圈叠加在地理区域上，表示城市活动水平。" />

## 何时使用
- **事件分布**：映射事件发生的位置和规模（如地震、销售事件）。
- **城市统计**：比较特定城市的人口或活动水平。
- **全球指标**：可视化国家级数据，气泡大小代表数值。

## 代码示例

### XAML
```xml
<ShapeMap xmlns="https://github.com/avaloniaui" Name="BubbleSample" Title="按人口排名的主要城市" Height="400" ShowLegend="True" LegendPosition="Bottom">
                        <ShapeMap.Layers>
                            <ShapeLayer GeoJson="{Binding WorldGeoJson}"
                                                 GeoJsonIdPath="ISO_A2"
                                                 LowBrush="#E3F2FD"
                                                 HighBrush="#E3F2FD"
                                                 StrokeThickness="0.3" />
                            <BubbleLayer LatitudePath="Lat"
                                                  LongitudePath="Lon"
                                                  SizePath="Population"
                                                  LabelPath="City"
                                                  MinBubbleSize="6"
                                                  MaxBubbleSize="45"
                                                  Fill="#B4F44336"
                                                  ShowLabels="True"
                                                  ItemsSource="{Binding CityBubbles}">
                                <BubbleLayer.TooltipTemplate>
                                    <DataTemplate>
                                        <StackPanel Spacing="4">
                                            <TextBlock Text="{Binding City}" FontWeight="Bold"/>
                                            <TextBlock Text="{Binding Population, StringFormat='人口: {0:N1}M'}"/>
                                        </StackPanel>
                                    </DataTemplate>
                                </BubbleLayer.TooltipTemplate>
                            </BubbleLayer>
                        </ShapeMap.Layers>
                    </ShapeMap>
```

### 数据模型 (C#)

确保 GeoJSON 文件已包含在项目中，并在运行时可在指定的相对路径下访问。

```csharp
using System.IO;

public record CityData(string City, double Lat, double Lon, double Population);

public string WorldGeoJson { get; } =
    File.ReadAllText("Resources/ne_110m_world.geojson");

public CityData[] CityBubbles { get; } = new CityData[]
{
    new("东京", 35.7, 139.7, 37.4),
    new("德里", 28.6, 77.2, 32.9),
    new("上海", 31.2, 121.5, 29.2),
    new("圣保罗", -23.5, -46.6, 22.4),
    new("墨西哥城", 19.4, -99.1, 21.9),
    new("开罗", 30.0, 31.2, 21.3),
    new("孟买", 19.1, 72.9, 21.0),
    new("北京", 39.9, 116.4, 20.9),
    new("纽约", 40.7, -74.0, 18.8),
    new("伦敦", 51.5, -0.1, 9.5),
    new("巴黎", 48.9, 2.3, 11.1),
    new("悉尼", -33.9, 151.2, 5.4)
};
```

## 通用属性: `ShapeLayer`

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `GeoJson` | GeoJSON 数据。 | `null` |
| `Source` | GeoJSON 数据来源的 URI。 | `null` |
| `GeoJsonIdPath` | GeoJSON 数据的唯一 ID。 | `null` |
| `ItemsSource` | 包含地图区域的数据集合。 | `null` |
| `RegionPath` | 将数据链接到 GeoJSON 坐标的属性。 | `null` |
| `ValuePath` | 将数据链接到区域值的属性。 | `null` |
| `MinValue` | 颜色标准化的最小值。 | `0.0` |
| `MaxValue` | 颜色标准化的最大值。 | `100.0` |
| `LowBrush` | 表示最低数据值的颜色。 | `#E3F2FD` |
| `HighBrush` | 表示最高数据值的颜色。 | `#1565C0` |
| `Stroke` | 区域轮廓的颜色。 | `null` |
| `StrokeThickness` | 区域轮廓的粗细。 | `0.5` |
| `ShowLabels` | 是否在区域上显示标签。 | `false` |
| `LabelPath` | 标签文本的路径。 | `null` |
| `LabelForeground` | 用于区域标签的画笔。 | `null` |
| `SelectionMode` | 区域选择模式。使用 `None`、`Single`、`SingleDeselect` 或 `Multiple`。 | `None` |
| `SelectionBrush` | 选定区域的颜色。 | `#FFC107` |
| `SelectionStroke` | 选定区域轮廓的颜色。 | `null` |
| `SelectionStrokeThickness` | 选定区域轮廓的粗细。 | `2.0` |
| `SelectedItem` | 当前选定的区域。 | `null` |
| `HoverBrush` | 区域悬停时使用的画笔。 | 白色，30% 不透明度 |

## 通用属性: `BubbleLayer`

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 气泡的数据源。 | `null` |
| `LatitudePath` | 纬度坐标的属性路径。 | `null` |
| `LongitudePath` | 经度坐标的属性路径。 | `null` |
| `SizePath` | 气泡的大小。 | `null` |
| `PointBrushPath` | 可选属性路径，为每个气泡提供画笔或颜色。 | `null` |
| `ShowLabels` | 是否在气泡上显示标签。 | `true` |
| `LabelPath` | 标签的内容。 | `null` |
| `MinBubbleSize` | 最小气泡半径。 | `8.0` |
| `MaxBubbleSize` | 最大气泡半径。 | `40.0` |
| `Fill` | 气泡的颜色。 | `null` |
| `Stroke` | 气泡轮廓的颜色。 | `null` |
