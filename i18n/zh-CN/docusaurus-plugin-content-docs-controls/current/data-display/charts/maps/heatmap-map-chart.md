---
id: heatmap-map-chart
title: 热力图地图
description: 使用颜色渐变可视化跨地理坐标的数据密度，适合在地图上显示活动热点和集中区域。
doc-type: reference
tags:
  - avalonia pro
---

import chartsMapsHeatmap from '/img/controls/charts/charts-maps-gradient.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

热力图地图使用 `ShapeMap` 和 `HeatmapLayer` 可视化跨地理坐标的数据密度。它们适合显示活动集中的热点区域。

<Image light={chartsMapsHeatmap} maxWidth={400} position="center" cornerRadius="true" alt="地理热力图，使用颜色渐变显示跨区域的数据密度热点和集中区域。" />

## 何时使用
- **用户活动**：可视化移动应用用户地理上最活跃的位置。
- **事件报告**：映射犯罪、交通事故或服务中断的热点。
- **环境密度**：显示物种或污染的浓度。

## 代码示例

### XAML
```xml
<ShapeMap xmlns="https://github.com/avaloniaui" Name="HeatmapSample" Title="全球地震活动" Height="400" ShowLegend="False">
                        <ShapeMap.Layers>
                            <ShapeLayer GeoJson="{Binding WorldGeoJson}"
                                                 GeoJsonIdPath="ISO_A2"
                                                 LowBrush="#F5F5F5"
                                                 HighBrush="#F5F5F5"
                                                 Stroke="#E0E0E0"
                                                 StrokeThickness="0.3" />
                            <HeatmapLayer LatitudePath="Lat"
                                                   LongitudePath="Lon"
                                                   IntensityPath="Magnitude"
                                                   MaxIntensity="9.0"
                                                   Radius="50"
                                                   LowBrush="#4000FF00"
                                                   MediumBrush="#CCFFFF00"
                                                   HighBrush="#FFFF0000"
                                                   ItemsSource="{Binding EarthquakeData}">
                                <HeatmapLayer.TooltipTemplate>
                                    <DataTemplate>
                                        <StackPanel Spacing="4">
                                            <TextBlock Text="地震" FontWeight="Bold"/>
                                            <TextBlock Text="{Binding Magnitude, StringFormat='震级: {0:N1}'}"/>
                                        </StackPanel>
                                    </DataTemplate>
                                </HeatmapLayer.TooltipTemplate>
                            </HeatmapLayer>
                        </ShapeMap.Layers>
                    </ShapeMap>
```

### 数据模型 (C#)

确保 GeoJSON 文件已包含在项目中，并在运行时可在指定的相对路径下访问。

```csharp
using System.IO;

public record EarthquakeItem(double Lat, double Lon, double Magnitude);

public string WorldGeoJson { get; } =
    File.ReadAllText("Resources/ne_110m_world.geojson");

public EarthquakeItem[] EarthquakeData { get; } = new EarthquakeItem[]
{
    new(38.3, 142.4, 9.1),
    new(35.0, 135.8, 6.9),
    new(34.4, 135.3, 6.1),
    new(3.3, 95.9, 9.1),
    new(-0.8, 99.8, 7.6),
    new(-7.5, 110.4, 6.3),
    new(-36.1, -72.9, 8.8),
    new(-33.4, -70.6, 6.5),
    new(34.2, -118.4, 6.7),
    new(37.9, -122.3, 6.9),
    new(36.2, -120.2, 5.8),
    new(61.3, -149.9, 7.1),
    new(57.8, -152.4, 7.9),
    new(28.2, 84.7, 7.8),
    new(37.2, 37.0, 7.8),
    new(38.0, 38.5, 7.5),
    new(-41.5, 174.8, 6.3),
    new(-42.7, 173.0, 7.8),
    new(42.4, 13.4, 6.2),
    new(15.5, 120.8, 7.7),
    new(19.4, -99.4, 7.1),
    new(33.4, 46.0, 7.3)
};
```

## 通用属性 (`HeatmapLayer`)

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 要渲染的地理点集合。 | `null` |
| `LatitudePath` | 纬度值的属性名称。 | `Latitude` |
| `LongitudePath` | 经度值的属性名称。 | `Longitude` |
| `IntensityPath` | 强度值的属性名称。 | `Intensity` |
| `Radius` | 每个热点的基本半径（像素）。 | `40.0` |
| `MaxIntensity` | 用于标准化的最大强度。 | `100.0` |
| `LowBrush` | 低强度值使用的画笔。 | `#0000FF00` |
| `MediumBrush` | 中强度值使用的画笔。 | `#FFFF00` |
| `HighBrush` | 高强度值使用的画笔。 | `#FF0000` |
