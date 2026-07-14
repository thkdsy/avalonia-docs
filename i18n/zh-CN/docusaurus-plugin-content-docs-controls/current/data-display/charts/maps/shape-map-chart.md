---
id: shape-map-chart
title: 形状地图
description: 从 GeoJSON 渲染任意地理或自定义形状，作为专业地图和交互式自定义区域可视化的基础。
doc-type: reference
tags:
  - avalonia pro
---

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

形状地图控件允许进行任意地理或自定义形状的可视化。它作为专业地图的基础，允许开发者定义自定义区域和交互。

## 何时使用
- **自定义区域**：可视化标准地图集未覆盖的区域（例如，特定邮政区域）。
- **物理布局**：将数据映射到示意图上（如硬件板或工厂车间）。
- **交互式图表**：创建高性能的交互式形状系统。

## 代码示例

### XAML

```xml
<ShapeMap xmlns="https://github.com/avaloniaui" Title="区域分析" Height="400">
    <ShapeMap.Layers>
        <ShapeLayer GeoJson="{Binding AreaGeoJson}"
                             ItemsSource="{Binding AreaData}"
                             RegionPath="AreaId" />
    </ShapeMap.Layers>
</ShapeMap>
```

### 数据模型 (C#)

确保 GeoJSON 文件已包含在项目中，并在运行时可在指定的相对路径下访问。

```csharp
using System.IO;

public record AreaInfo(string AreaId, string Status);

public string AreaGeoJson { get; } =
    File.ReadAllText("Resources/custom-areas.geojson");

public string WorldGeoJson { get; } =
    File.ReadAllText("Resources/ne_110m_world.geojson");

public ObservableCollection<AreaInfo> AreaData { get; } = new()
{
    new("A1", "活跃"), new("A2", "维护中"), new("B1", "活跃")
};

public record OfficeLocation(double Lat, double Lon, string Name);

public ObservableCollection<OfficeLocation> Offices { get; } = new()
{
    new(40.7128, -74.0060, "纽约"),
    new(51.5074, -0.1278, "伦敦"),
    new(35.6762, 139.6503, "东京")
};

public record Route(double FromLat, double FromLon, double ToLat, double ToLon, double Passengers);

public ObservableCollection<Route> Routes { get; } = new()
{
    new(40.7128, -74.0060, 51.5074, -0.1278, 95),
    new(34.0522, -118.2437, 35.6762, 139.6503, 85)
};

public record Segment(string Category, double Amount);

public record RegionalPoint(double Lat, double Lon, ObservableCollection<Segment> Segments);

public ObservableCollection<RegionalPoint> RegionalData { get; } = new()
{
    new(40.7128, -74.0060, new ObservableCollection<Segment>
    {
        new("科技", 45),
        new("金融", 30),
        new("零售", 25)
    }),
    new(51.5074, -0.1278, new ObservableCollection<Segment>
    {
        new("科技", 30),
        new("金融", 50),
        new("零售", 20)
    })
};
```

## 通用属性 (ShapeMap)

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Layers` | 按顺序渲染的 `MapLayer` 实例集合。 | 空集合 |

## 通用属性 (ShapeLayer)

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `GeoJson` | 形状的几何数据。 | `null` |
| `Source` | 用于加载 GeoJSON 数据的 URI 源。 | `null` |
| `GeoJsonIdPath` | GeoJSON 中用作区域标识符的属性名称。 | `null` |
| `ItemsSource` | 表示形状的数据项目。 | `null` |
| `RegionPath` | 将数据匹配到形状 ID 的关键属性。 | `null` |
| `ValuePath` | 绑定到每个形状的数值的属性名称。 | `null` |
| `MinValue` | 用于标准化的最小值。 | `0.0` |
| `MaxValue` | 用于标准化的最大值。 | `100.0` |
| `LowBrush` | 用于最低数据值的画笔。 | `#E3F2FD` |
| `HighBrush` | 用于最高数据值的画笔。 | `#1565C0` |
| `Stroke` | 用于形状轮廓的画笔。 | `null` |
| `StrokeThickness` | 形状轮廓的粗细。 | `0.5` |
| `ShowLabels` | 是否为形状绘制标签。 | `false` |
| `LabelPath` | 用于形状标签的属性名称。 | `null` |
| `LabelForeground` | 用于形状标签的画笔。 | `null` |
| `IsSelectionEnabled` | 是否启用形状选择。 | `true` |
| `SelectedItem` | 单选场景中当前选定的项目。 | `null` |
| `SelectedItems` | 多选场景中选定项目的集合。 | `null` |
| `SelectionMode` | 图层的选择行为。 | `None` |
| `SelectionBrush` | 用于选定形状的画笔。 | `#FFC107` |
| `SelectionStroke` | 用于选定形状的轮廓画笔。 | `null` |
| `SelectionStrokeThickness` | 用于选定形状的轮廓粗细。 | `2.0` |
| `HoverBrush` | 形状悬停时应用的画笔。 | 白色 @ 30% 不透明度 |
| `ColorMappings` | 形状的可选显式颜色映射规则。 | 空集合 |
| `Legend` | 与图层关联的可选图例。 | `null` |

## 通用属性 (MapLegend)

`MapLegend` 显示由地图图层生成的图例项，例如显式的 `ShapeLayer.ColorMappings`。

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Source` | 用于生成图例项的地图图层。 | `null` |
| `Orientation` | 图例条目的布局方向，`Horizontal` 或 `Vertical`。 | `Vertical` |
| `ItemTemplate` | 用于渲染每个图例项的可选模板。 | `null` |
| `Items` | 从源图层生成的只读 `AvaloniaList<LegendItem>`。 | 空集合 |

## 地图图层

除了 `ShapeLayer`，形状地图还支持几种可添加到 `Layers` 集合中的特殊图层类型。每个图层在地图上渲染不同类型的覆盖层。

`ShapeMap.Layers` 可以绑定到 `IList<MapLayer>`。如果列表实现了 `INotifyCollectionChanged`，则运行时添加或删除图层将更新渲染、命中测试和生成的图例。

图层集合属性（如 `MarkerLayer.Markers`、`VectorLayer.Arcs` 和 `HeatmapLayer.ItemsSource`）在其集合源实现 `INotifyCollectionChanged` 时也会更新地图。用新列表替换图层集合属性会将该图层重新绑定到新源。

### MarkerLayer

在地理坐标处渲染单个点标记。标记可以手动定义或绑定到数据源。

#### 通用属性 (`MarkerLayer`)

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Markers` | 手动定义的 `MapMarker` 对象集合。 | 空集合 |
| `ItemsSource` | 用于自动生成标记的数据源。 | `null` |
| `LatitudePath` | 数据源项目中纬度值的属性路径。 | `null` |
| `LongitudePath` | 数据源项目中经度值的属性路径。 | `null` |
| `LabelPath` | 数据源项目中标签值的属性路径。 | `null` |
| `MarkerSize` | 每个标记的默认大小（像素）。 | `10.0` |
| `MarkerType` | 默认标记形状：`Circle`、`Diamond`、`Triangle`、`Rectangle` 或 `Pin`。 | `Circle` |
| `Fill` | 标记的默认填充画笔。 | `Red` |
| `Stroke` | 标记的默认描边画笔。 | `White` |
| `IsVisible` | 图层是否可见。 | `true` |
| `Opacity` | 图层的不透明度。 | `1.0` |
| `TooltipTemplate` | 用于工具提示的数据模板。 | `null` |

#### XAML

```xml
<ShapeMap xmlns="https://github.com/avaloniaui" Title="办公室位置" Height="400">
    <ShapeMap.Layers>
        <ShapeLayer GeoJson="{Binding WorldGeoJson}" />
        <MarkerLayer ItemsSource="{Binding Offices}"
                              LatitudePath="Lat" LongitudePath="Lon"
                              LabelPath="Name" MarkerSize="12"
                              MarkerType="Pin" Fill="DodgerBlue" />
    </ShapeMap.Layers>
</ShapeMap>
```

### LineLayer

在地理点对之间渲染连接线。线条粗细可根据数据值变化。线条可以绘制为直线或曲线。

#### 通用属性 (`LineLayer`)

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 包含连接项的数据源。 | `null` |
| `FromLatitudePath` | 源纬度的属性路径。 | `null` |
| `FromLongitudePath` | 源经度的属性路径。 | `null` |
| `ToLatitudePath` | 目标纬度的属性路径。 | `null` |
| `ToLongitudePath` | 目标经度的属性路径。 | `null` |
| `ValuePath` | 线条值的属性路径（影响粗细）。 | `null` |
| `Stroke` | 用于连接线的画笔。 | `#2196F3` |
| `MinLineThickness` | 最小线条粗细。 | `1.0` |
| `MaxLineThickness` | 最大线条粗细。 | `6.0` |
| `IsCurved` | 是否绘制曲线（贝塞尔曲线）而非直线。 | `true` |
| `ShowEndpoints` | 是否在端点显示圆圈。 | `true` |
| `IsVisible` | 图层是否可见。 | `true` |
| `Opacity` | 图层的不透明度。 | `1.0` |
| `TooltipTemplate` | 用于工具提示的数据模板。 | `null` |

#### XAML

```xml
<ShapeMap xmlns="https://github.com/avaloniaui" Title="飞行路线" Height="400">
    <ShapeMap.Layers>
        <ShapeLayer GeoJson="{Binding WorldGeoJson}" />
        <LineLayer ItemsSource="{Binding Routes}"
                            FromLatitudePath="FromLat" FromLongitudePath="FromLon"
                            ToLatitudePath="ToLat" ToLongitudePath="ToLon"
                            ValuePath="Passengers" IsCurved="True"
                            MinLineThickness="1" MaxLineThickness="5" />
    </ShapeMap.Layers>
</ShapeMap>
```

### VectorLayer

使用地理坐标在地图上渲染几何形状（线、弧、圆、多边形和多段线）。形状通过图层集合定义。

#### 通用属性 (`VectorLayer`)

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Lines` | `MapLine` 对象集合。 | 空集合 |
| `Arcs` | `MapArc` 对象集合。 | 空集合 |
| `Circles` | `MapCircle` 对象集合。 | 空集合 |
| `Polygons` | `MapPolygon` 对象集合。 | 空集合 |
| `Polylines` | `MapPolyline` 对象集合。 | 空集合 |
| `IsLineAnimationEnabled` | 是否动画显示线条绘制。 | `true` |
| `IsVisible` | 图层是否可见。 | `true` |
| `Opacity` | 图层的不透明度。 | `1.0` |
| `TooltipTemplate` | 用于工具提示的数据模板。 | `null` |

#### XAML

```xml
<ShapeMap xmlns="https://github.com/avaloniaui" Title="领土边界" Height="400">
    <ShapeMap.Layers>
        <ShapeLayer GeoJson="{Binding WorldGeoJson}" />
        <VectorLayer IsLineAnimationEnabled="True">
            <VectorLayer.Circles>
                <MapCircle Latitude="48.8566" Longitude="2.3522"
                                    Radius="20" StrokeThickness="2">
                    <MapCircle.Fill>
                        <SolidColorBrush Color="#402196F3" />
                    </MapCircle.Fill>
                </MapCircle>
            </VectorLayer.Circles>
            <VectorLayer.Lines>
                <MapLine FromLatitude="40.7128" FromLongitude="-74.006"
                                  ToLatitude="51.5074" ToLongitude="-0.1278"
                                  StrokeThickness="2" />
            </VectorLayer.Lines>
        </VectorLayer>
    </ShapeMap.Layers>
</ShapeMap>
```

### PieChartMapLayer

在特定地理坐标处绘制饼图。每个饼图可视化该位置值的细分。

#### 通用属性 (`PieChartMapLayer`)

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 包含地理坐标和值段的数据源。 | `null` |
| `LatitudePath` | 纬度坐标的属性路径。 | `null` |
| `LongitudePath` | 经度坐标的属性路径。 | `null` |
| `ValuesPath` | 每个数据项内段对象集合的属性路径。 | `null` |
| `ValuePath` | 每个段对象中数值的属性路径。 | `null` |
| `LabelPath` | 每个段对象中标签的属性路径。 | `null` |
| `PieSize` | 饼图的直径（像素）。 | `30.0` |
| `Palette` | 用于饼图扇区的调色板。 | 内置 8 色调色板 |
| `IsVisible` | 图层是否可见。 | `true` |
| `Opacity` | 图层的不透明度。 | `1.0` |
| `TooltipTemplate` | 用于工具提示的数据模板。 | `null` |

#### XAML

```xml
<ShapeMap xmlns="https://github.com/avaloniaui" Title="区域销售细分" Height="400">
    <ShapeMap.Layers>
        <ShapeLayer GeoJson="{Binding WorldGeoJson}" />
        <PieChartMapLayer ItemsSource="{Binding RegionalData}"
                                   LatitudePath="Lat" LongitudePath="Lon"
                                   ValuesPath="Segments" ValuePath="Amount"
                                   LabelPath="Category" PieSize="40" />
    </ShapeMap.Layers>
</ShapeMap>
```

## 另请参阅

- [等值线地图](/controls/data-display/charts/maps/choropleth-map-chart)
- [气泡地图](/controls/data-display/charts/maps/bubble-map-chart)
- [热力图](/controls/data-display/charts/maps/heatmap-map-chart)
