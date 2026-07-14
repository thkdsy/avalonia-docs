---
id: shapes-and-geometries
title: 形状与几何图形
description: 在 Avalonia 中用于绘制 2D 矢量图形的形状控件和几何类型。
doc-type: reference
---

Avalonia 提供了一组用于绘制常见 2D 矢量图形的形状控件，以及一套几何系统，用于定义可应用于路径、裁剪和命中测试的复杂轮廓。

## 形状控件

形状控件是可以直接放入 XAML 布局中的可视元素。它们会参与布局，也可以接收指针事件。

### Rectangle

```xml
<Rectangle Width="100" Height="60"
           Fill="SteelBlue" Stroke="DarkBlue" StrokeThickness="2" />
```

| 属性 | 说明 |
|---|---|
| `RadiusX`, `RadiusY` | 圆角矩形的圆角半径。 |

```xml
<Rectangle Width="100" Height="60" Fill="Coral"
           RadiusX="10" RadiusY="10" />
```

### Ellipse

```xml
<Ellipse Width="100" Height="80"
         Fill="MediumPurple" Stroke="Indigo" StrokeThickness="2" />
```

当 `Width` 与 `Height` 相等时，`Ellipse` 就是一个圆形。

### Line

```xml
<Line StartPoint="0,0" EndPoint="200,100"
      Stroke="Red" StrokeThickness="2" />
```

### Polyline

绘制一组相连的线段，但不会自动闭合图形：

```xml
<Polyline Points="0,0 50,50 100,0 150,50 200,0"
          Stroke="Green" StrokeThickness="2" />
```

### Polygon

与 `Polyline` 类似，但图形会自动闭合：

```xml
<Polygon Points="50,0 100,100 0,100"
         Fill="Gold" Stroke="DarkGoldenRod" StrokeThickness="1" />
```

`Polyline` 和 `Polygon` 都支持 `FillRule` 属性，用于控制自相交图形的填充方式：

- `EvenOdd`（默认）：对重叠区域交替填充，从而形成镂空效果。
- `NonZero`：无论是否重叠，都会填充所有封闭区域。

```xml
<Polygon Points="50,0 61,35 98,35 68,57 79,91 50,70 21,91 32,57 2,35 39,35"
         Fill="Orange" FillRule="EvenOdd" />
```

### Path

这是最灵活的形状控件。它使用 `Geometry` 来定义轮廓：

```xml
<!-- 使用迷你语言 -->
<Path Data="M 10,10 L 100,10 L 100,100 Z"
      Fill="LightBlue" Stroke="Navy" StrokeThickness="1" />

<!-- 使用 PathGeometry -->
<Path Fill="Orange">
    <Path.Data>
        <PathGeometry>
            <PathFigure StartPoint="0,0" IsClosed="True">
                <LineSegment Point="100,0" />
                <LineSegment Point="100,100" />
            </PathFigure>
        </PathGeometry>
    </Path.Data>
</Path>
```

## 常见形状属性

所有形状都继承自 `Shape`，并共享以下属性：

| 属性 | 说明 |
|---|---|
| `Fill` | 用于绘制内部区域的画刷。 |
| `Stroke` | 用于绘制轮廓的画刷。 |
| `StrokeThickness` | 以设备无关像素表示的轮廓粗细。 |
| `StrokeDashArray` | 定义虚线模式的一组 `double` 值。 |
| `StrokeDashOffset` | 绘制虚线时在虚线模式中的起始偏移。 |
| `StrokeLineCap` | 线段端点样式：`Flat`、`Round` 或 `Square`。 |
| `StrokeJoin` | 转角处的连接样式：`Miter`、`Bevel` 或 `Round`。 |
| `StrokeMiterLimit` | 控制斜接转角何时改为斜切的比值上限。当斜接长度除以描边粗细超过该值时，连接会从尖角斜接变为斜切。默认值为 `10`。仅在 `StrokeJoin` 为 `Miter` 时生效。 |
| `Stretch` | 形状填充其分配空间的方式：`None`、`Fill`、`Uniform`、`UniformToFill`。 |

### 虚线

```xml
<Line StartPoint="0,0" EndPoint="200,0"
      Stroke="Black" StrokeThickness="2"
      StrokeDashArray="5,3" />

<Rectangle Width="150" Height="80" Fill="Transparent"
           Stroke="Gray" StrokeThickness="1"
           StrokeDashArray="4,2,1,2" />
```

## 几何类型

几何对象用于从数学上描述 2D 形状。它们比形状控件更轻量，通常用于 `Path.Data`、`Clip` 和 `OpacityMask`。

### RectangleGeometry

```xml
<Path Fill="LightCoral">
    <Path.Data>
        <RectangleGeometry Rect="0,0,100,60" />
    </Path.Data>
</Path>
```

### EllipseGeometry

```xml
<Path Fill="LightGreen">
    <Path.Data>
        <EllipseGeometry Center="50,40" RadiusX="50" RadiusY="40" />
    </Path.Data>
</Path>
```

### LineGeometry

```xml
<Path Stroke="Red" StrokeThickness="2">
    <Path.Data>
        <LineGeometry StartPoint="0,0" EndPoint="100,50" />
    </Path.Data>
</Path>
```

### PathGeometry

这是最灵活的几何类型，由 figure 和 segment 组成：

```xml
<PathGeometry>
    <PathFigure StartPoint="10,50" IsClosed="True" IsFilled="True">
        <LineSegment Point="100,50" />
        <ArcSegment Point="100,150" Size="50,50"
                    SweepDirection="Clockwise" />
        <LineSegment Point="10,150" />
    </PathFigure>
</PathGeometry>
```

### 段类型

| 段类型 | 说明 |
|---|---|
| `LineSegment` | 绘制一条到指定点的直线。 |
| `ArcSegment` | 绘制一段椭圆弧。 |
| `BezierSegment` | 绘制三次贝塞尔曲线（两个控制点）。 |
| `QuadraticBezierSegment` | 绘制二次贝塞尔曲线（一个控制点）。 |
| `PolyLineSegment` | 绘制一系列相连的直线。 |
| `PolyBezierSegment` | 绘制一系列相连的三次贝塞尔曲线。每组三个点分别表示第一个控制点、第二个控制点和终点。 |

### CombinedGeometry

使用集合运算组合两个几何对象：

```xml
<Path Fill="CornflowerBlue">
    <Path.Data>
        <CombinedGeometry GeometryCombineMode="Exclude">
            <CombinedGeometry.Geometry1>
                <EllipseGeometry Center="50,50" RadiusX="50" RadiusY="50" />
            </CombinedGeometry.Geometry1>
            <CombinedGeometry.Geometry2>
                <EllipseGeometry Center="80,50" RadiusX="50" RadiusY="50" />
            </CombinedGeometry.Geometry2>
        </CombinedGeometry>
    </Path.Data>
</Path>
```

| CombineMode | 结果 |
|---|---|
| `Union` | 任一几何覆盖到的区域。 |
| `Intersect` | 两个几何共同覆盖的区域。 |
| `Exclude` | 在第一个几何中、但不在第二个几何中的区域。 |
| `Xor` | 只被其中一个几何覆盖、而不是同时被两者覆盖的区域。 |

### GeometryGroup

将多个几何对象合并为单个几何对象：

```xml
<Path Fill="Salmon" Stroke="DarkRed" StrokeThickness="1">
    <Path.Data>
        <GeometryGroup FillRule="EvenOdd">
            <EllipseGeometry Center="50,50" RadiusX="50" RadiusY="50" />
            <EllipseGeometry Center="50,50" RadiusX="25" RadiusY="25" />
        </GeometryGroup>
    </Path.Data>
</Path>
```

`FillRule` 决定重叠区域如何填充：
- `EvenOdd`：对重叠区域交替填充（形成空洞）。
- `NonZero`：填充所有封闭区域。

## Path 迷你语言

`Path` 的 `Data` 属性接受 SVG 风格的路径字符串数据。这种紧凑语法非常适合描述复杂形状。

### 命令

| 命令 | 参数 | 说明 |
|---|---|---|
| `M` / `m` | `x,y` | 移动到某点（绝对 / 相对） |
| `L` / `l` | `x,y` | 绘制到某点的直线 |
| `H` / `h` | `x` | 水平线 |
| `V` / `v` | `y` | 垂直线 |
| `C` / `c` | `x1,y1 x2,y2 x,y` | 三次贝塞尔曲线 |
| `S` / `s` | `x2,y2 x,y` | 平滑三次贝塞尔曲线 |
| `Q` / `q` | `x1,y1 x,y` | 二次贝塞尔曲线 |
| `T` / `t` | `x,y` | 平滑二次贝塞尔曲线 |
| `A` / `a` | `rx,ry rotation large-arc sweep x,y` | 椭圆弧 |
| `Z` / `z` | | 闭合路径 |

大写命令使用绝对坐标，小写命令使用相对坐标。

### 示例

```xml
<!-- 三角形 -->
<Path Data="M 50,0 L 100,100 L 0,100 Z" Fill="Gold" />

<!-- 圆角矩形路径 -->
<Path Data="M 10,0 H 90 A 10,10 0 0 1 100,10 V 50 A 10,10 0 0 1 90,60 H 10 A 10,10 0 0 1 0,50 V 10 A 10,10 0 0 1 10,0 Z"
      Fill="LightSkyBlue" />

<!-- 心形 -->
<Path Data="M 50,30 A 20,20 0 0 1 90,30 A 20,20 0 0 1 50,80 A 20,20 0 0 1 10,30 A 20,20 0 0 1 50,30 Z"
      Fill="Red" />

<!-- Star -->
<Path Data="M 50,0 L 61,35 L 98,35 L 68,57 L 79,91 L 50,70 L 21,91 L 32,57 L 2,35 L 39,35 Z"
      Fill="Orange" />
```

## 在代码中构建几何图形

你可以使用 `StreamGeometryContext` 以编程方式构建几何图形。调用 `StreamGeometry.Open()` 获取上下文后，再使用其绘制方法定义 figure：

```csharp
var geometry = new StreamGeometry();

using (var ctx = geometry.Open())
{
    ctx.BeginFigure(new Point(10, 50), isFilled: true);
    ctx.LineTo(new Point(100, 50));
    ctx.ArcTo(
        new Point(100, 150),
        new Size(50, 50),
        rotationAngle: 0,
        isLargeArc: false,
        SweepDirection.Clockwise);
    ctx.LineTo(new Point(10, 150));
    ctx.EndFigure(isClosed: true);
}
```

### StreamGeometryContext 方法

| 方法 | 说明 |
|---|---|
| `BeginFigure(Point, bool isFilled = true)` | 在指定点开始新的 figure。将 `isFilled` 设为 `false` 可创建开放且不填充的路径。 |
| `LineTo(Point, bool isStroked = true)` | 绘制一条到指定点的直线。 |
| `ArcTo(Point, Size, double, bool, SweepDirection, bool isStroked = true)` | 绘制一段到指定点的椭圆弧。 |
| `CubicBezierTo(Point, Point, Point, bool isStroked = true)` | 使用两个控制点和一个终点绘制三次贝塞尔曲线。 |
| `QuadraticBezierTo(Point, Point, bool isStroked = true)` | 使用一个控制点和一个终点绘制二次贝塞尔曲线。 |
| `EndFigure(bool isClosed)` | 结束当前 figure。将 `isClosed` 设为 `true` 可闭合图形。 |

在任意 segment 上将 `isStroked` 设为 `false`，即可在保留该 segment 形状的同时不绘制它的轮廓。这对于创建部分描边的路径很有用。

```csharp
using (var ctx = geometry.Open())
{
    ctx.BeginFigure(new Point(0, 0), isFilled: false);
    ctx.LineTo(new Point(50, 0));              // 会描边
    ctx.LineTo(new Point(100, 0), isStroked: false); // 间隔（不描边）
    ctx.LineTo(new Point(150, 0));             // 会描边
    ctx.EndFigure(isClosed: false);
}
```

## 几何方法

所有 `Geometry` 对象都提供了用于命中测试和变换的方法：

| 方法 | 说明 |
|---|---|
| `FillContains(Point)` | 如果点位于几何对象填充区域内，则返回 `true`。 |
| `StrokeContains(Pen, Point)` | 如果点位于使用指定画笔描边后的几何轮廓上，则返回 `true`。 |
| `GetWidenedGeometry(Pen)` | 返回一个新的几何对象，表示使用指定画笔描边当前几何后所覆盖的区域。适合用于创建轮廓形状。 |
| `GetFlattenedPathGeometry()` | 返回一个简化后的 `PathGeometry`，其中曲线会被近似为线段。 |

```csharp
var ellipse = new EllipseGeometry { Center = new Point(50, 50), RadiusX = 40, RadiusY = 40 };
var pen = new Pen(Brushes.Black, 10);

// 获取描边后椭圆轮廓所对应的几何图形
var outlined = ellipse.GetWidenedGeometry(pen);
```

## 将几何对象作为资源使用

你可以把几何对象定义为资源，以便在整个应用中复用：

```xml
<Application.Resources>
    <StreamGeometry x:Key="CheckmarkIcon">M 4,8.5 L 8,12.5 L 16,4</StreamGeometry>
    <StreamGeometry x:Key="CloseIcon">M 4,4 L 16,16 M 16,4 L 4,16</StreamGeometry>
</Application.Resources>

<Path Data="{StaticResource CheckmarkIcon}" Stroke="Green" StrokeThickness="2" />
```

`StreamGeometry` 是一种轻量、不可变、且经过性能优化的几何类型。它非常适合用于图标路径和其他静态形状。

## 另请参阅

- [绘制图形](/docs/graphics-animation/drawing-graphics)：Avalonia 图形系统概览。
- [画刷](/docs/graphics-animation/brushes)：填充与描边画刷。
- [效果](/docs/graphics-animation/effects)：盒阴影、裁剪和不透明度遮罩。
- [添加图标](/docs/graphics-animation/adding-icons)：使用图标字体和矢量图标。
