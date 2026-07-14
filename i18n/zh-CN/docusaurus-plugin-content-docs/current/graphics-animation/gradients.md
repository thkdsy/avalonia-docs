---
id: gradients
title: 渐变
description: 如何在 Avalonia UI 中使用 LinearGradientBrush、RadialGradientBrush 和 ConicGradientBrush 创建颜色过渡效果。
doc-type: reference
---

渐变画刷可以在两种或多种颜色之间创建平滑过渡。凡是接受画刷的地方都可以使用它，包括 `Background`、`Foreground`、`BorderBrush`、`Fill` 和 `Stroke`。Avalonia 提供三种渐变画刷类型：

| 画刷 | 说明 |
|---|---|
| [`LinearGradientBrush`](/api/avalonia/media/lineargradientbrush) | 沿直线方向过渡颜色。 |
| [`RadialGradientBrush`](/api/avalonia/media/radialgradientbrush) | 以中心点为起点，沿椭圆向外过渡颜色。 |
| [`ConicGradientBrush`](/api/avalonia/media/conicgradientbrush) | 围绕中心点以环绕方式过渡颜色。 |

所有渐变画刷都共享 `GradientStops` 集合和 `SpreadMethod` 属性。下面的章节会分别介绍每种画刷类型，随后再说明适用于它们的共同概念。

## 线性渐变画刷

`LinearGradientBrush` 会沿一条由起点和终点定义的直线混合颜色。

### 基本语法

```xml
<LinearGradientBrush StartPoint="0%,0%" EndPoint="100%,0%">
    <GradientStop Color="#FF6B6B" Offset="0.0"/>
    <GradientStop Color="#4ECDC4" Offset="1.0"/>
</LinearGradientBrush>
```

### `StartPoint` 与 `EndPoint`

这两个属性用于控制渐变方向。你可以用百分比指定数值（例如 `"0%,50%"`），也可以用相对于边界框的小数值（例如 `"0,0.5"`）。两种写法是等价的。

常见方向模式：

| 方向 | `StartPoint` | `EndPoint` |
|---|---|---|
| 水平（从左到右） | `0%,50%` | `100%,50%` |
| 垂直（从上到下） | `50%,0%` | `50%,100%` |
| 对角线（左上到右下） | `0%,0%` | `100%,100%` |

### 水平渐变

```xml
<LinearGradientBrush StartPoint="0%,50%" EndPoint="100%,50%">
    <GradientStop Color="#FF6B6B" Offset="0.0"/>
    <GradientStop Color="#4ECDC4" Offset="1.0"/>
</LinearGradientBrush>
```

### 垂直渐变

```xml
<LinearGradientBrush StartPoint="50%,0%" EndPoint="50%,100%">
    <GradientStop Color="#A8E6CF" Offset="0.0"/>
    <GradientStop Color="#3D84A8" Offset="1.0"/>
</LinearGradientBrush>
```

### 多色渐变

添加更多 [`GradientStop`](/api/avalonia/media/gradientstop) 元素，即可创建经过多种颜色的渐变。通过调整 `Offset` 值的分布，可以控制各颜色出现的位置：

```xml
<LinearGradientBrush StartPoint="0%,50%" EndPoint="100%,50%">
    <GradientStop Color="#FF6B6B" Offset="0.0"/>
    <GradientStop Color="#FF8E53" Offset="0.3"/>
    <GradientStop Color="#FF5E3A" Offset="0.6"/>
    <GradientStop Color="#4ECDC4" Offset="1.0"/>
</LinearGradientBrush>
```

### 常见使用场景

#### 按钮背景

```xml
<Button>
    <Button.Background>
        <LinearGradientBrush StartPoint="0%,0%" EndPoint="0%,100%">
            <GradientStop Color="#4CAF50" Offset="0.0"/>
            <GradientStop Color="#45A049" Offset="1.0"/>
        </LinearGradientBrush>
    </Button.Background>
</Button>
```

#### 面板背景

```xml
<Border CornerRadius="8">
    <Border.Background>
        <LinearGradientBrush StartPoint="0%,0%" EndPoint="100%,100%">
            <GradientStop Color="#FF9A9E" Offset="0.0"/>
            <GradientStop Color="#FAD0C4" Offset="0.5"/>
            <GradientStop Color="#FFD1FF" Offset="1.0"/>
        </LinearGradientBrush>
    </Border.Background>
</Border>
```

## 径向渐变画刷

`RadialGradientBrush` 会从中心点开始，以椭圆方式向外混合颜色。

### Basic syntax

```xml
<RadialGradientBrush GradientOrigin="50%,50%" Center="50%,50%" RadiusX="50%" RadiusY="50%">
    <GradientStop Color="Yellow" Offset="0.0"/>
    <GradientStop Color="Red" Offset="1.0"/>
</RadialGradientBrush>
```

### 关键属性

| 属性 | 说明 |
|---|---|
| `Center` | 最外层椭圆的中心点，以边界框百分比表示。默认值为 `50%,50%`。 |
| `GradientOrigin` | 渐变开始的点（最内层颜色位置）。默认与 `Center` 相同。 |
| `RadiusX`, `RadiusY` | 椭圆的水平和垂直半径。默认值为 `50%`。 |

当你将 `GradientOrigin` 设置为与 `Center` 不同的值时，渐变会呈现偏心效果，这对于模拟光照效果很有用：

```xml
<RadialGradientBrush GradientOrigin="30%,30%" Center="50%,50%">
    <GradientStop Color="White" Offset="0.0"/>
    <GradientStop Color="#3D84A8" Offset="1.0"/>
</RadialGradientBrush>
```

### 椭圆渐变

通过给 `RadiusX` 和 `RadiusY` 指定不同的值，你可以创建非圆形的渐变：

```xml
<RadialGradientBrush Center="50%,50%" RadiusX="80%" RadiusY="40%">
    <GradientStop Color="#A8E6CF" Offset="0.0"/>
    <GradientStop Color="#3D84A8" Offset="1.0"/>
</RadialGradientBrush>
```

## 圆锥渐变画刷

`ConicGradientBrush` 会围绕中心点环绕过渡颜色，效果类似色轮表盘。

### Basic syntax

```xml
<ConicGradientBrush Center="50%,50%" Angle="0">
    <GradientStop Color="Red" Offset="0.0"/>
    <GradientStop Color="Yellow" Offset="0.25"/>
    <GradientStop Color="Green" Offset="0.5"/>
    <GradientStop Color="Blue" Offset="0.75"/>
    <GradientStop Color="Red" Offset="1.0"/>
</ConicGradientBrush>
```

### 关键属性

| 属性 | 说明 |
|---|---|
| `Center` | 环绕渐变的中心点。默认值为 `50%,50%`。 |
| `Angle` | 起始角度，以度为单位，从顶部开始顺时针计算。默认值为 `0`。 |

若要创建无缝环绕渐变，请将最后一个 `GradientStop` 的颜色设置为与第一个相同。

## 共享的渐变概念

### `GradientStop` 元素

每个渐变画刷都包含一个或多个 `GradientStop` 元素。每个 stop 定义一个 `Color` 和一个 `Offset`：

| 属性 | 说明 |
|---|---|
| `Color` | 任意合法颜色值（十六进制、命名颜色、`rgb()`、`hsl()` 等）。 |
| `Offset` | 从 `0.0` 到 `1.0` 的数值，表示在渐变中的位置。 |

如果省略 `Offset`，Avalonia 会均匀分布这些 stop。当两个 stop 使用相同 offset 时，你会得到清晰的颜色边界，而不是平滑过渡。

### `SpreadMethod`

`SpreadMethod` 用于控制：当渐变没有填满整个区域时，会发生什么（例如某个 `LinearGradientBrush` 的起点和终点都位于边界框内部）。

| 值 | 行为 |
|---|---|
| `Pad`（默认） | 用两端颜色向外延伸，填满剩余空间。 |
| `Reflect` | 渐变反向后继续重复。 |
| `Repeat` | 渐变从头开始重复。 |

```xml
<LinearGradientBrush StartPoint="0%,50%" EndPoint="50%,50%" SpreadMethod="Reflect">
    <GradientStop Color="#08AEEA" Offset="0.0"/>
    <GradientStop Color="#2AF598" Offset="1.0"/>
</LinearGradientBrush>
```

### `Opacity`

所有渐变画刷都继承自 `Brush` 的 `Opacity` 属性。你可以将其设置为 `0.0`（完全透明）到 `1.0`（完全不透明）之间的值，让整个渐变半透明：

```xml
<LinearGradientBrush StartPoint="0%,50%" EndPoint="100%,50%" Opacity="0.5">
    <GradientStop Color="#FF6B6B" Offset="0.0"/>
    <GradientStop Color="#4ECDC4" Offset="1.0"/>
</LinearGradientBrush>
```

如果你需要对不同颜色分别设置透明度，请直接在 `Color` 值中使用 alpha 通道（例如 `#80FF6B6B`）。

### 在代码后置中创建渐变

如果你需要动态生成渐变，也可以在 C# 中构建渐变画刷：

```csharp
var brush = new LinearGradientBrush
{
    StartPoint = new RelativePoint(0, 0.5, RelativeUnit.Relative),
    EndPoint = new RelativePoint(1, 0.5, RelativeUnit.Relative),
    GradientStops =
    {
        new GradientStop(Color.Parse("#FF6B6B"), 0.0),
        new GradientStop(Color.Parse("#4ECDC4"), 1.0)
    }
};

myBorder.Background = brush;
```

同样的模式也适用于 `RadialGradientBrush` 和 `ConicGradientBrush`。

## 完整示例

<XamlPreview>

```xml
<StackPanel Spacing="20" Margin="20" xmlns="https://github.com/avaloniaui">
    <!-- 水平渐变 -->
    <Border Height="100" CornerRadius="8">
        <Border.Background>
            <LinearGradientBrush StartPoint="0%,50%" EndPoint="100%,50%">
                <GradientStop Color="#FF6B6B" Offset="0.0"/>
                <GradientStop Color="#FF8E53" Offset="0.3"/>
                <GradientStop Color="#FF5E3A" Offset="0.6"/>
                <GradientStop Color="#4ECDC4" Offset="1.0"/>
            </LinearGradientBrush>
        </Border.Background>
        <TextBlock Text="水平"
                   HorizontalAlignment="Center"
                   VerticalAlignment="Center"
                   Foreground="White"/>
    </Border>

    <!-- 垂直渐变 -->
    <Border Height="100" CornerRadius="8">
        <Border.Background>
            <LinearGradientBrush StartPoint="50%,0%" EndPoint="50%,100%">
                <GradientStop Color="#A8E6CF" Offset="0.0"/>
                <GradientStop Color="#3D84A8" Offset="0.5"/>
                <GradientStop Color="#46CDCF" Offset="1.0"/>
            </LinearGradientBrush>
        </Border.Background>
        <TextBlock Text="垂直"
                   HorizontalAlignment="Center"
                   VerticalAlignment="Center"
                   Foreground="White"/>
    </Border>

    <!-- 径向渐变 -->
    <Border Height="100" CornerRadius="8">
        <Border.Background>
            <RadialGradientBrush GradientOrigin="30%,30%">
                <GradientStop Color="White" Offset="0.0"/>
                <GradientStop Color="#3D84A8" Offset="1.0"/>
            </RadialGradientBrush>
        </Border.Background>
        <TextBlock Text="径向"
                   HorizontalAlignment="Center"
                   VerticalAlignment="Center"
                   Foreground="White"/>
    </Border>

    <!-- 圆锥渐变 -->
    <Border Height="100" CornerRadius="8">
        <Border.Background>
            <ConicGradientBrush Center="50%,50%">
                <GradientStop Color="#FF6B6B" Offset="0.0"/>
                <GradientStop Color="#4ECDC4" Offset="0.5"/>
                <GradientStop Color="#FF6B6B" Offset="1.0"/>
            </ConicGradientBrush>
        </Border.Background>
        <TextBlock Text="圆锥"
                   HorizontalAlignment="Center"
                   VerticalAlignment="Center"
                   Foreground="White"/>
    </Border>
</StackPanel>
```

</XamlPreview>

## 另请参阅

- [画刷](/docs/graphics-animation/brushes)：所有画刷类型概览，包括 `SolidColorBrush` 和平铺画刷。
- [效果](/docs/graphics-animation/effects)：盒阴影、裁剪和不透明度遮罩。
- [形状与几何图形](/docs/graphics-animation/shapes-and-geometries)：可用渐变画刷填充的形状绘制。
- [自定义渲染](/docs/graphics-animation/custom-rendering)：使用 `DrawingContext` 直接进行底层绘制，可直接使用渐变画刷。
