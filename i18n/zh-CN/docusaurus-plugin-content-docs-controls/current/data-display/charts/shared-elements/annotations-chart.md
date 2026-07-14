---
id: annotations-chart
title: 标注
description: 向图表添加上下文线条、带状区域、形状和文本，用于突出显示阈值、里程碑或关注区域。
doc-type: reference
tags:
  - avalonia pro
---

import chartsFeaturesAnnotation from '/img/controls/charts/charts-annotations.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

标注允许您使用线条、带状区域、形状和自定义文本为图表添加上下文信息。它们对于突出显示阈值、里程碑或特定关注区域非常有用。

<Image light={chartsFeaturesAnnotation} maxWidth={400} position="center" cornerRadius="true" alt="带标注叠加层的图表，包括水平阈值线、阴影舒适区域和自定义文本标签。" />

## 使用时机

- **阈值**：在性能图表上显示"目标"或"限制"线。
- **里程碑**：在时间线上标记特定的关注日期。
- **区域高亮**：跨一组值着色"危险区"或"舒适区"。

## 代码示例

### XAML

```xml
<CartesianChart xmlns="https://github.com/avaloniaui" Name="ShapesChart" Height="250">
                        <CartesianChart.HorizontalAxis>
                            <NumericalAxis Title="X" Minimum="0" Maximum="10" />
                        </CartesianChart.HorizontalAxis>
                        <CartesianChart.VerticalAxis>
                            <NumericalAxis Title="Y" Minimum="0" Maximum="100" />
                        </CartesianChart.VerticalAxis>
                        <CartesianChart.Series>
                            <ScatterSeries Title="Points" ItemsSource="{Binding ShapeData}"
                                                    CategoryPath="X" ValuePath="Y" MarkerSize="8" />
                        </CartesianChart.Series>
                        <CartesianChart.Annotations>
                            <!-- 高亮区域 -->
                            <RectangleAnnotation X="0.6" Y="60" Width="0.2" Height="20"
                                                          Stroke="Blue" Label="A 区">
                                <RectangleAnnotation.Fill>
                                    <SolidColorBrush Color="#280000FF" />
                                </RectangleAnnotation.Fill>
                            </RectangleAnnotation>
                            <!-- 关注圆 -->
                            <EllipseAnnotation X="0.3" Y="30" RadiusX="0.05" RadiusY="5"
                                                        Stroke="Orange" Label="聚类">
                                <EllipseAnnotation.Fill>
                                    <SolidColorBrush Color="#28FFA500" />
                                </EllipseAnnotation.Fill>
                            </EllipseAnnotation>
                            <!-- 趋势箭头 -->
                            <ArrowLineAnnotation X1="0.3" Y1="35" X2="0.7" Y2="65"
                                                          Stroke="Purple" StrokeThickness="2" ShowEndArrow="True" Label="增长" />
                        </CartesianChart.Annotations>
                    </CartesianChart>
```

### 数据模型 (C#)

```csharp
public record Point(double X, double Y);

public ObservableCollection<Point> ShapeData { get; } = new()
{
    new(3, 30),
    new(7, 70)
};
```

## 坐标系统

标注坐标在轴空间中指定。在分类轴上，水平值使用基于零的分类槽索引。在连续水平轴上，水平值使用轴值域，例如数值或 `DateTime` 刻度。垂直值使用垂直轴值域。

形状大小也以轴单位度量。在对数轴或带有刻度中断的轴上，相同的数据增量可能根据其原点映射到不同的像素大小。自定义标注渲染器应使用 `CartesianAnnotationRenderContext.DataXToPixel` 和 `DataYToPixel` 获取位置，使用 `DeltaXToPixelsAt(origin, value)` 或 `DeltaYToPixelsAt(origin, value)` 获取原点相关的大小。

## 常用属性（LineAnnotation）

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Value` | 线条放置位置的轴空间值。水平线使用垂直轴值；垂直线使用水平轴值。 | `0` |
| `Orientation` | `Horizontal` 或 `Vertical`。 | `Horizontal` |
| `Stroke` | 标注线条的颜色。 | `Gray` |
| `StrokeThickness` | 标注线条的宽度。 | `1.0` |
| `DashStyle` | 标注线条使用的虚线样式。 | `null` |
| `Foreground` | 标签文本使用的画刷。当为 `null` 时，标注会回退使用 `Stroke`（在支持的情况下）。 | `null` |
| `FontSize` | 标注标签使用的字号。 | `12.0` |
| `Label` | 在线条旁边显示的文本。 | `null` |

## 常用属性（`BandAnnotation`）

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `FromValue` | 带状区域的起始轴空间值。水平带使用垂直轴值；垂直带使用水平轴值。 | `0` |
| `ToValue` | 带状区域的结束轴空间值。 | `0` |
| `Orientation` | `Horizontal` 或 `Vertical`。 | `Horizontal` |
| `Fill` | 用于填充阴影区域的画刷。 | `null` |
| `Foreground` | 带状标签文本使用的画刷。当为 `null` 时，标注会回退使用 `Stroke`（在支持的情况下）。 | `null` |
| `FontSize` | 带状标签使用的字号。 | `12.0` |
| `Label` | 在带状区域内显示的文本。 | `null` |

## 常用属性（TextAnnotation）

放置在图表区域特定坐标处的文本标注。

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `X` | 水平轴值。分类轴使用基于零的分类槽索引。 | `0` |
| `Y` | 垂直轴值。 | `0` |
| `Text` | 标注显示的文本。 | `null` |
| `Foreground` | 文本填充使用的画刷。 | `null` |
| `FontSize` | 文本使用的字号。 | `12.0` |
| `Stroke` | 可选的文本轮廓画刷。仅当显式设置 `Stroke` 时才绘制文本轮廓。 | `Gray` |
| `StrokeThickness` | 可选的文本轮廓粗细。仅当显式设置 `Stroke` 时才应用。 | `1.0` |
| `Opacity` | 标注的不透明度。 | `1.0` |

## 常用属性（`RectangleAnnotation`）

放置在图表区域特定坐标处的矩形标注。

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `X` | 左侧水平轴值。分类轴使用基于零的分类槽索引。 | `0` |
| `Y` | 下部垂直轴值。 | `0` |
| `Width` | 水平轴单位的宽度。 | `0.5` |
| `Height` | 垂直轴单位的高度。 | `10.0` |
| `Fill` | 用于填充矩形的画刷。 | `null` |
| `CornerRadius` | 圆角矩形的圆角半径。 | `0` |
| `Label` | 在矩形中心显示的文本。 | `null` |
| `Stroke` | 矩形边框的颜色。 | `Gray` |
| `StrokeThickness` | 矩形边框的宽度。 | `1` |
| `Foreground` | 矩形标签文本使用的画刷。 | `null` |
| `FontSize` | 矩形标签使用的字号。 | `12.0` |
| `Opacity` | 标注的不透明度。 | `1.0` |

### XAML

```xml
<CartesianChart xmlns="https://github.com/avaloniaui" Height="250">
    <CartesianChart.Annotations>
        <RectangleAnnotation X="0.2" Y="50" Width="0.3" Height="20"
                                    Stroke="DarkGreen" StrokeThickness="2"
                                    CornerRadius="4" Label="目标区域">
            <RectangleAnnotation.Fill>
                <SolidColorBrush Color="#3200FF00" />
            </RectangleAnnotation.Fill>
        </RectangleAnnotation>
    </CartesianChart.Annotations>
</CartesianChart>
```

## 常用属性（`EllipseAnnotation`）

放置在图表区域特定坐标处的椭圆标注。

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `X` | 中心水平轴值。分类轴使用基于零的分类槽索引。 | `0` |
| `Y` | 中心垂直轴值。 | `0` |
| `RadiusX` | 水平半径（轴单位）。 | `0.25` |
| `RadiusY` | 垂直半径（轴单位）。 | `10.0` |
| `Fill` | 用于填充椭圆的画刷。 | `null` |
| `Label` | 在椭圆中心显示的文本。 | `null` |
| `Stroke` | 椭圆边框的颜色。 | `Gray` |
| `StrokeThickness` | 椭圆边框的宽度。 | `1` |
| `Foreground` | 椭圆标签文本使用的画刷。 | `null` |
| `FontSize` | 椭圆标签使用的字号。 | `12.0` |
| `Opacity` | 标注的不透明度。 | `1.0` |

### XAML

```xml
<CartesianChart xmlns="https://github.com/avaloniaui" Height="250">
    <CartesianChart.Annotations>
        <EllipseAnnotation X="0.5" Y="60" RadiusX="0.15" RadiusY="15"
                                  Stroke="Purple" StrokeThickness="2"
                                  Label="异常">
            <EllipseAnnotation.Fill>
                <SolidColorBrush Color="#32800080" />
            </EllipseAnnotation.Fill>
        </EllipseAnnotation>
    </CartesianChart.Annotations>
</CartesianChart>
```

## 常用属性（`ArrowLineAnnotation`）

可选在任一端或两端带箭头的线条标注，用于指示方向或在两个数据点之间引起注意。

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `X1` | 起始水平轴值。分类轴使用基于零的分类槽索引。 | `0` |
| `Y1` | 起始垂直轴值。 | `0` |
| `X2` | 结束水平轴值。 | `0` |
| `Y2` | 结束垂直轴值。 | `0` |
| `ShowStartArrow` | 是否在起始端显示箭头。 | `false` |
| `ShowEndArrow` | 是否在结束端显示箭头。 | `true` |
| `ArrowSize` | 箭头的大小（像素）。 | `8.0` |
| `Label` | 在线条中点显示的文本。 | `null` |
| `Stroke` | 箭头线条的颜色。 | `Gray` |
| `StrokeThickness` | 箭头线条的宽度。 | `1` |
| `Foreground` | 箭头标签文本使用的画刷。 | `null` |
| `FontSize` | 箭头标签使用的字号。 | `12.0` |
| `Opacity` | 标注的不透明度。 | `1.0` |

### XAML

```xml
<CartesianChart xmlns="https://github.com/avaloniaui" Height="250">
    <CartesianChart.Annotations>
        <ArrowLineAnnotation X1="0.1" Y1="30" X2="0.6" Y2="80"
                                    Stroke="OrangeRed" StrokeThickness="2"
                                    ShowEndArrow="True" ArrowSize="10"
                                    Label="上升趋势" />
    </CartesianChart.Annotations>
</CartesianChart>
```
