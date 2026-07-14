---
id: brushes
title: 画刷
description: Avalonia 中用于绘制表面的画刷类型，包括纯色、渐变和平铺画刷。
doc-type: reference
---

画刷定义了 Avalonia 中表面的绘制方式。任何接受 `Brush` 的属性（例如 `Background`、`Foreground`、`BorderBrush`、`Fill` 和 `Stroke`）都可以使用这里介绍的任意画刷类型。

## SolidColorBrush

使用单一颜色填充区域。这是最常见的画刷类型，当你在画刷属性上直接设置颜色字符串时，通常会隐式使用它。

```xml
<Border Background="SteelBlue" />

<!-- 显式写法 -->
<Border>
    <Border.Background>
        <SolidColorBrush Color="#4682B4" Opacity="0.8" />
    </Border.Background>
</Border>
```

颜色可以按以下格式指定：

| 格式 | 示例 | 说明 |
|---|---|---|
| 格式 | 示例 | 说明 |
| 命名颜色 | `Red`, `SteelBlue` | 任意标准 CSS/WPF 颜色名称。 |
| `#RRGGBB` | `#4682B4` | 十六进制 RGB。 |
| `#AARRGGBB` | `#804682B4` | 十六进制 ARGB（含 alpha）。 |
| `#RGB` | `#F00` | 简写十六进制 RGB。 |
| `rgb()` | `rgb(70, 130, 180)` | CSS RGB 函数，取值范围 0-255。 |
| `rgba()` | `rgba(70, 130, 180, 0.8)` | 带 alpha 的 CSS RGB（0.0-1.0）。 |
| `hsl()` | `hsl(207, 44%, 49%)` | CSS HSL（色相、饱和度、亮度）。 |
| `hsla()` | `hsla(207, 44%, 49%, 0.8)` | 带 alpha 的 CSS HSL。 |
| `hsv()` | `hsv(207, 61%, 71%)` | HSV（色相、饱和度、明度）。 |
| `hsva()` | `hsva(207, 61%, 71%, 0.8)` | 带 alpha 的 HSV。 |

这些格式可用于任何期望画刷或颜色的场景，包括 XAML 属性、样式，以及代码中的 `Brush.Parse()` / `Color.Parse()`。

```xml
<!-- 这些写法都是等价的 -->
<Border Background="SteelBlue" />
<Border Background="#4682B4" />
<Border Background="rgb(70, 130, 180)" />
<Border Background="hsl(207, 44%, 49%)" />
```

### 在代码中创建

```csharp
var brush = new SolidColorBrush(Colors.SteelBlue);
var brush2 = new SolidColorBrush(Color.Parse("#4682B4"));
var brush3 = Brush.Parse("rgb(70, 130, 180)");
var brush4 = Brush.Parse("hsl(207, 44%, 49%)");
myBorder.Background = brush;
```

`Brush.Parse()` 接受上面列出的所有颜色格式，包括命名颜色、十六进制值和 CSS 颜色函数。

## LinearGradientBrush

使用沿一条线在不同颜色之间过渡的渐变来填充区域。若想查看更聚焦的线性渐变指南，请参阅 [渐变](/docs/graphics-animation/gradients)。

```xml
<Border Height="80" CornerRadius="8">
    <Border.Background>
        <LinearGradientBrush StartPoint="0%,50%" EndPoint="100%,50%">
            <GradientStop Color="#6366F1" Offset="0" />
            <GradientStop Color="#EC4899" Offset="1" />
        </LinearGradientBrush>
    </Border.Background>
</Border>
```

### 关键属性

| 属性 | 说明 |
|---|---|
| `StartPoint` | 渐变线的起点，可使用相对坐标（`50%,0%`）或绝对坐标。 |
| `EndPoint` | 渐变线的终点。 |
| [`GradientStops`](/api/avalonia/media/gradientstops) | 由 `GradientStop` 对象组成的集合，用于定义颜色和位置。 |
| `SpreadMethod` | 渐变在定义区域之外的填充方式：`Pad`（默认）、`Reflect` 或 `Repeat`。 |
| `Opacity` | 画刷整体不透明度（0.0 到 1.0）。 |

### 渐变方向

```xml
<!-- 水平（从左到右） -->
<LinearGradientBrush StartPoint="0%,50%" EndPoint="100%,50%">

<!-- 垂直（从上到下） -->
<LinearGradientBrush StartPoint="50%,0%" EndPoint="50%,100%">

<!-- 对角线 -->
<LinearGradientBrush StartPoint="0%,0%" EndPoint="100%,100%">
```

## RadialGradientBrush

使用从中心点向外辐射的渐变来填充区域。

```xml
<Ellipse Width="150" Height="150">
    <Ellipse.Fill>
        <RadialGradientBrush GradientOrigin="30%,30%">
            <GradientStop Color="White" Offset="0" />
            <GradientStop Color="#3B82F6" Offset="0.6" />
            <GradientStop Color="#1E3A8A" Offset="1" />
        </RadialGradientBrush>
    </Ellipse.Fill>
</Ellipse>
```

### 关键属性

| 属性 | 说明 |
|---|---|
| `Center` | 最外层圆的中心。默认值为 `50%,50%`。 |
| `GradientOrigin` | 渐变原点（焦点）。相对中心的偏移可以产生聚光灯效果。 |
| `RadiusX`, `RadiusY` | 最外层渐变圆的水平和垂直半径。默认值为 `50%`。 |
| `GradientStops` | 沿半径方向分布的颜色和位置。 |
| `SpreadMethod` | `Pad`、`Reflect` 或 `Repeat`。 |

## ConicGradientBrush

使用围绕中心点旋转扫过的渐变来填充区域，颜色会随着旋转方向逐步过渡。

```xml
<Ellipse Width="150" Height="150">
    <Ellipse.Fill>
        <ConicGradientBrush Center="50%,50%" Angle="0">
            <GradientStop Color="#EF4444" Offset="0" />
            <GradientStop Color="#F59E0B" Offset="0.25" />
            <GradientStop Color="#22C55E" Offset="0.5" />
            <GradientStop Color="#3B82F6" Offset="0.75" />
            <GradientStop Color="#EF4444" Offset="1" />
        </ConicGradientBrush>
    </Ellipse.Fill>
</Ellipse>
```

### 关键属性

| 属性 | 说明 |
|---|---|
| `Center` | 圆锥渐变的中心点。默认值为 `50%,50%`。 |
| `Angle` | 起始角度（度）。默认值为 `0`。 |
| `GradientStops` | 沿旋转方向分布的颜色和位置。 |

## ImageBrush

使用图像绘制区域。

```xml
<Border Width="200" Height="200">
    <Border.Background>
        <ImageBrush Source="avares://MyApp/Assets/texture.png"
                    Stretch="UniformToFill"
                    TileMode="Tile"
                    SourceRect="0,0,64,64" />
    </Border.Background>
</Border>
```

### 关键属性

| 属性 | 说明 |
|---|---|
| `Source` | 图像源。支持 `avares://` URI 和文件路径。 |
| `Stretch` | 图像填充区域的方式：`None`、`Fill`、`Uniform`（默认）、`UniformToFill`。 |
| `TileMode` | 图像平铺方式：`None`（默认）、`Tile`、`FlipX`、`FlipY`、`FlipXY`。 |
| `AlignmentX` | 图像在平铺单元内的水平对齐方式：`Left`、`Center`（默认）、`Right`。 |
| `AlignmentY` | 图像在平铺单元内的垂直对齐方式：`Top`、`Center`（默认）、`Bottom`。 |
| `SourceRect` | 要使用的源图像矩形区域。 |
| `DestinationRect` | 目标区域中的目标矩形。 |
| `Opacity` | 画刷整体不透明度。 |
| `BitmapInterpolationMode` | 插值质量：`Default`、`LowQuality`、`MediumQuality`、`HighQuality`。 |

### 平铺示例

```xml
<Border Width="300" Height="200">
    <Border.Background>
        <ImageBrush Source="avares://MyApp/Assets/pattern.png"
                    TileMode="Tile"
                    DestinationRect="0,0,32,32" />
    </Border.Background>
</Border>
```

## VisualBrush

使用另一个可视元素的渲染结果来绘制区域。

```xml
<Border Width="200" Height="200" BorderBrush="Gray" BorderThickness="1">
    <Border.Background>
        <VisualBrush Stretch="Uniform" TileMode="Tile"
                     DestinationRect="0,0,50,50">
            <VisualBrush.Visual>
                <TextBlock Text="Avalonia" FontSize="14" Foreground="LightGray"
                           RenderTransform="rotate(45deg)" />
            </VisualBrush.Visual>
        </VisualBrush>
    </Border.Background>
</Border>
```

### 关键属性

| 属性 | 说明 |
|---|---|
| `Visual` | 作为画刷内容进行渲染的可视元素。 |
| `Stretch` | 可视内容填充区域的方式。 |
| `TileMode` | 重复该可视内容时使用的平铺模式。 |
| `SourceRect` | 要使用的可视区域部分。 |
| `DestinationRect` | 每个平铺单元对应的目标矩形。 |

:::info
`VisualBrush` 可以捕获任意控件的视觉外观。这对于创建倒影效果、水印和预览缩略图非常有用。
:::

## 画刷通用属性

所有画刷类型都共享以下属性：

| 属性 | 说明 |
|---|---|
| `Opacity` | 0.0（完全透明）到 1.0（完全不透明）之间的值。 |
| `Transform` | 应用于画刷坐标的变换。 |
| `TransformOrigin` | 画刷变换的原点。 |

## OpacityMask

任何控件的 `OpacityMask` 属性都接受一个用于控制逐像素透明度的画刷。遮罩画刷的 alpha 通道会决定控件中每个像素的不透明度。

```xml
<Image Source="avares://MyApp/Assets/photo.png" Width="200" Height="200">
    <Image.OpacityMask>
        <LinearGradientBrush StartPoint="0%,0%" EndPoint="0%,100%">
            <GradientStop Color="Black" Offset="0" />
            <GradientStop Color="Transparent" Offset="1" />
        </LinearGradientBrush>
    </Image.OpacityMask>
</Image>
```

在这个示例中，图像会从顶部完全可见逐渐过渡到底部透明。遮罩中的颜色值本身并不重要，真正起作用的是 alpha 通道。

## 将画刷作为资源使用

你可以把画刷定义为资源，以便在整个应用中复用：

```xml
<Application.Resources>
    <SolidColorBrush x:Key="PrimaryBrush" Color="#6366F1" />
    <LinearGradientBrush x:Key="AccentGradient" StartPoint="0%,0%" EndPoint="100%,100%">
        <GradientStop Color="#6366F1" Offset="0" />
        <GradientStop Color="#EC4899" Offset="1" />
    </LinearGradientBrush>
</Application.Resources>
```

```xml
<Button Background="{StaticResource PrimaryBrush}" />
<Border Background="{DynamicResource AccentGradient}" />
```

## 在代码中使用画刷

```csharp
// SolidColorBrush
var solid = new SolidColorBrush(Colors.IndianRed);

// LinearGradientBrush
var linear = new LinearGradientBrush
{
    StartPoint = new RelativePoint(0, 0.5, RelativeUnit.Relative),
    EndPoint = new RelativePoint(1, 0.5, RelativeUnit.Relative),
    GradientStops =
    {
        new GradientStop(Colors.Blue, 0),
        new GradientStop(Colors.Red, 1)
    }
};

// RadialGradientBrush
var radial = new RadialGradientBrush
{
    GradientStops =
    {
        new GradientStop(Colors.White, 0),
        new GradientStop(Colors.Black, 1)
    }
};

myBorder.Background = linear;
```

## 另请参阅

- [渐变](/docs/graphics-animation/gradients)：关于线性渐变的专题指南。
- [图形绘制](/docs/graphics-animation/drawing-graphics)：形状与几何图形。
- [图像插值](/docs/graphics-animation/image-interpolation)：位图渲染质量设置。
