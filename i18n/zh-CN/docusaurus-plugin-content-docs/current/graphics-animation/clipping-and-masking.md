---
id: clipping-and-masking
title: 裁剪与遮罩
description: 在 Avalonia 中使用裁剪和不透明度遮罩来限制可见内容的技术。
doc-type: explanation
---

裁剪会把控件或绘制内容的可见区域限制在某个定义好的范围内。遮罩则通过不透明度渐变来部分隐藏内容。这两种技术都非常适合创建异形控件、圆形头像和各种视觉效果。

## ClipToBounds

最简单的裁剪形式，是把子内容限制在父元素的边界内。你可以在任意控件上设置 `ClipToBounds="True"`：

```xml
<Border Width="100" Height="100" ClipToBounds="True"
        Background="LightGray">
    <!-- 这张图片超出了 Border 边界，但会被裁剪 -->
    <Image Source="/assets/photo.jpg" Width="200" Height="200" />
</Border>
```

默认情况下，`ClipToBounds` 是 `false`，因此内容可以溢出父元素范围。

## Clip 属性

`Clip` 属性接受任意 `Geometry`，可用于定义非矩形裁剪区域。

### 圆形裁剪

你可以通过椭圆裁剪图像，创建一个圆形头像：

```xml
<Image Source="/assets/avatar.jpg" Width="100" Height="100"
       Stretch="UniformToFill">
    <Image.Clip>
        <EllipseGeometry Rect="0,0,100,100" />
    </Image.Clip>
</Image>
```

### 圆角矩形裁剪

```xml
<Image Source="/assets/banner.jpg" Width="300" Height="200"
       Stretch="UniformToFill">
    <Image.Clip>
        <RectangleGeometry Rect="0,0,300,200" RadiusX="16" RadiusY="16" />
    </Image.Clip>
</Image>
```

### 自定义形状裁剪

任意形状可以使用 `PathGeometry` 来实现：

```xml
<Image Source="/assets/photo.jpg" Width="200" Height="200"
       Stretch="UniformToFill">
    <Image.Clip>
        <PathGeometry>
            <PathFigure StartPoint="100,0" IsClosed="True">
                <LineSegment Point="200,75" />
                <LineSegment Point="160,200" />
                <LineSegment Point="40,200" />
                <LineSegment Point="0,75" />
            </PathFigure>
        </PathGeometry>
    </Image.Clip>
</Image>
```

### 使用流式几何语法

紧凑的路径迷你语言同样可以定义裁剪区域：

```xml
<Image Source="/assets/photo.jpg" Width="200" Height="200">
    <Image.Clip>
        <StreamGeometry>M 100,0 L 200,75 160,200 40,200 0,75 Z</StreamGeometry>
    </Image.Clip>
</Image>
```

## 使用 CornerRadius 进行裁剪

`Border` 可以通过 `CornerRadius` 提供内置裁剪能力。圆角边框内部的内容会自动被裁剪：

```xml
<Border CornerRadius="50" Width="100" Height="100" ClipToBounds="True">
    <Image Source="/assets/avatar.jpg" Stretch="UniformToFill" />
</Border>
```

对于圆形图像来说，这种方式通常比使用 `EllipseGeometry` 更简单。

## 不透明度遮罩

可以使用 `OpacityMask` 搭配画刷来实现淡出或部分隐藏内容。遮罩画刷透明的区域会变为不可见，而不透明区域则会保持可见：

### 渐变淡出

```xml
<Image Source="/assets/landscape.jpg" Width="400" Height="300">
    <Image.OpacityMask>
        <LinearGradientBrush StartPoint="0%,0%" EndPoint="0%,100%">
            <GradientStop Color="Black" Offset="0" />
            <GradientStop Color="Black" Offset="0.6" />
            <GradientStop Color="Transparent" Offset="1" />
        </LinearGradientBrush>
    </Image.OpacityMask>
</Image>
```

这会让图像底部逐渐变为透明，形成一种常见的“淡出”效果。

### 水平淡出

```xml
<TextBlock Text="This text fades to the right" FontSize="24">
    <TextBlock.OpacityMask>
        <LinearGradientBrush StartPoint="0%,0%" EndPoint="100%,0%">
            <GradientStop Color="Black" Offset="0" />
            <GradientStop Color="Black" Offset="0.7" />
            <GradientStop Color="Transparent" Offset="1" />
        </LinearGradientBrush>
    </TextBlock.OpacityMask>
</TextBlock>
```

### 径向遮罩

```xml
<Image Source="/assets/photo.jpg" Width="300" Height="300">
    <Image.OpacityMask>
        <RadialGradientBrush>
            <GradientStop Color="Black" Offset="0" />
            <GradientStop Color="Black" Offset="0.5" />
            <GradientStop Color="Transparent" Offset="1" />
        </RadialGradientBrush>
    </Image.OpacityMask>
</Image>
```

这会创建一种边缘逐渐淡出的晕影效果。

### VisualBrush 遮罩

你也可以把 `VisualBrush` 用作不透明度遮罩，让内容按照另一个控件的形状来显示：

```xml
<Image Source="/assets/photo.jpg" Width="300" Height="300">
    <Image.OpacityMask>
        <VisualBrush>
            <VisualBrush.Visual>
                <TextBlock Text="HELLO" FontSize="120" FontWeight="Bold"
                           Foreground="Black" />
            </VisualBrush.Visual>
        </VisualBrush>
    </Image.OpacityMask>
</Image>
```

图像只会在 `TextBlock` 渲染出不透明像素的地方显示，从而形成文字形状的镂空效果。

## 自定义控件中的裁剪

在渲染自定义控件时，你也可以通过代码来应用裁剪：

```csharp
public override void Render(DrawingContext context)
{
    // 压入一个裁剪区域
    using (context.PushClip(new Rect(10, 10, 80, 80)))
    {
        context.FillRectangle(Brushes.Blue, new Rect(0, 0, 100, 100));
        // 只有位于 (10,10,80,80) 范围内的部分会可见
    }
}
```

### 在代码中使用几何裁剪

```csharp
var ellipse = new EllipseGeometry(new Rect(0, 0, 100, 100));
using (context.PushGeometryClip(ellipse))
{
    context.DrawImage(bitmap, new Rect(0, 0, 100, 100));
}
```

## 另请参阅

- [形状与几何图形](/docs/graphics-animation/shapes-and-geometries)：用于裁剪区域的几何类型。
- [画刷](/docs/graphics-animation/brushes)：不透明度遮罩可用的画刷类型。
- [自定义渲染](/docs/graphics-animation/custom-rendering)：DrawingContext 参考。
