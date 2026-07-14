---
id: seat-map-chart
title: 非地理地图（座位图）
description: 使用 ShapeMap 控件和自定义 GeoJSON 实现非地理布局，如座位安排、平面图或交互式场馆布置。
doc-type: reference
tags:
  - avalonia pro
---

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

ShapeMap 控件可以处理非地理坐标系，使其非常适合自定义布局，如飞机座位安排、平面图或剧院布置。

## 何时使用
- **座位预订**：交通或场馆的交互式座位安排。
- **设施管理**：在建筑平面图上可视化数据。
- **交互式 UI**：创建可点击、数据驱动的自定义形状布局。

## 代码示例

### XAML
```xml
<ShapeMap xmlns="https://github.com/avaloniaui" Name="SeatMapSample" Title="飞机座位布局">
    <ShapeMap.Layers>
        <ShapeLayer GeoJson="{Binding SeatMapGeoJson}"
                             GeoJsonIdPath="id"
                             RegionPath="SeatNumber"
                             ValuePath="Class"
                             ItemsSource="{Binding SeatMapData}"
                             SelectionMode="Multiple"
                             SelectedItems="{Binding SelectedSeats}" />
    </ShapeMap.Layers>
</ShapeMap>
```

### 数据模型 (C#)

确保 GeoJSON 文件已包含在项目中，并在运行时可在指定的相对路径下访问。

```csharp
using System.IO;

public record SeatInfo(string SeatNumber, string Class, decimal Price, string Status);

public string SeatMapGeoJson { get; } =
    File.ReadAllText("Resources/seat-map.geojson");

public ObservableCollection<SeatInfo> SeatMapData { get; } = new()
{
    new("1A", "商务舱", 500m, "可预订"),
    new("1B", "商务舱", 500m, "可预订"),
    new("10C", "经济舱", 150m, "可预订"),
    new("10D", "经济舱", 150m, "可预订")
};

public ObservableCollection<SeatInfo> SelectedSeats { get; } = new();
```

## 通用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `GeoJson` | 表示布局的自定义 GeoJSON。 | `null` |
| `RegionPath` | 用于将数据匹配到形状的属性。 | `null` |
| `ValuePath` | 用于根据状态对形状进行颜色编码的属性。 | `null` |
| `SelectionMode` | `None`、`Single`、`SingleDeselect` 或 `Multiple`。 | `None` |
| `SelectedItems` | 绑定到选定的数据项目。 | `null` |
*（注：座位图使用自定义 GeoJSON 来渲染非几何形状）*
