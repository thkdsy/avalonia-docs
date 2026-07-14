---
id: effects
title: 效果
description: Avalonia 控件中的视觉效果，包括盒阴影、裁剪和不透明度遮罩。
doc-type: explanation
---

Avalonia 支持多种视觉效果，可为控件增加层次感和视觉吸引力。主要效果类型包括盒阴影、裁剪和不透明度遮罩。

## 盒阴影

[`Border`](/api/avalonia/controls/border) 和 `ContentPresenter` 上的 [`BoxShadow`](/api/avalonia/media/boxshadow) 属性可为元素添加投影或内嵌阴影。它的语法遵循 CSS `box-shadow` 的约定。

### 基本语法

```xml
<Border BoxShadow="5 5 10 0 #80000000" CornerRadius="8"
        Background="White" Padding="20">
    <TextBlock Text="Shadow" />
</Border>
```

阴影参数按顺序分别为：`offsetX offsetY blur spread color`。

| 参数 | 说明 |
|---|---|
| `offsetX` | 水平偏移。正值会让阴影向右移动。 |
| `offsetY` | 垂直偏移。正值会让阴影向下移动。 |
| `blur` | 模糊半径。值越大，阴影越柔和。必须是非负值。 |
| `spread` | 扩散半径。正值会扩大阴影，负值会收缩阴影。 |
| `color` | 阴影颜色。支持十六进制值（`#80000000`）、命名颜色（`Gray`）以及颜色函数（`rgba(0,0,0,0.5)`、`hsla(0,0%,0%,0.3)`）。 |

### 阴影中的颜色函数

带逗号的颜色函数（例如 `rgba()` 和 `hsla()`）都得到完整支持，并且可用于多重阴影定义：

```xml
<Border BoxShadow="0 4 8 0 rgba(0,0,0,0.3), 0 2 4 0 rgba(0,0,0,0.1)"
        CornerRadius="8" Background="White" Padding="20">
    <TextBlock Text="RGBA shadows" />
</Border>
```

### 内嵌阴影

在阴影定义前加上 `inset`，即可在元素内部绘制阴影：

```xml
<Border BoxShadow="inset 0 2 4 0 #40000000" CornerRadius="8"
        Background="#F0F0F0" Padding="20">
    <TextBlock Text="Inset shadow" />
</Border>
```

### 多重阴影

多个阴影定义之间用逗号分隔：

```xml
<Border BoxShadow="0 2 4 0 #20000000, 0 8 16 0 #10000000"
        CornerRadius="12" Background="White" Padding="24">
    <TextBlock Text="Layered shadows" />
</Border>
```

### 常见阴影模式

```xml
<!-- 轻微抬升 -->
<Border BoxShadow="0 1 3 0 #20000000" />

<!-- 中等抬升 -->
<Border BoxShadow="0 4 6 -1 #20000000, 0 2 4 -2 #20000000" />

<!-- 高抬升 -->
<Border BoxShadow="0 10 15 -3 #20000000, 0 4 6 -4 #20000000" />

<!-- 发光效果 -->
<Border BoxShadow="0 0 20 5 #4060A0FF" />

<!-- 内嵌按压效果 -->
<Border BoxShadow="inset 0 2 4 0 #40000000" />
```

### 在代码中设置盒阴影

```csharp
myBorder.BoxShadow = BoxShadows.Parse("0 4 8 0 #40000000");
```

## BlurEffect

任意 `Visual` 上的 [`Effect`](/api/avalonia/media/effect) 属性都可以接收效果对象。`BlurEffect` 会对整个元素应用高斯模糊：

```xml
<Border Background="SteelBlue" Padding="20" CornerRadius="8">
    <Border.Effect>
        <BlurEffect Radius="10" />
    </Border.Effect>
    <TextBlock Text="Blurred content" Foreground="White" />
</Border>
```

| 属性 | 说明 |
|---|---|
| `Radius` | 以像素为单位的模糊半径。值越大，模糊越明显。默认值为 5。 |

## DropShadowEffect

`DropShadowEffect` 通过 `Effect` 属性在整个视觉元素后方添加阴影。这与 `BoxShadow` 不同，后者只适用于 `Border` 元素。

```xml
<TextBlock Text="Shadow Text" FontSize="24">
    <TextBlock.Effect>
        <DropShadowEffect OffsetX="3" OffsetY="3"
                          BlurRadius="5" Color="Black" Opacity="0.5" />
    </TextBlock.Effect>
</TextBlock>
```

| 属性 | 说明 |
|---|---|
| `OffsetX` | 以像素为单位的水平阴影偏移。默认约为 3.5。 |
| `OffsetY` | 以像素为单位的垂直阴影偏移。默认约为 3.5。 |
| `BlurRadius` | 阴影模糊半径。默认值为 5。 |
| `Color` | 阴影颜色。默认值为 `Black`。 |
| `Opacity` | 阴影不透明度，范围从 0.0 到 1.0。默认值为 1.0。 |

### DropShadowDirectionEffect

这是另一种效果，它使用方向和深度，而不是显式偏移量：

```xml
<Border Background="White" Padding="20" CornerRadius="8">
    <Border.Effect>
        <DropShadowDirectionEffect ShadowDepth="5" Direction="315"
                                    BlurRadius="10" Color="Black" Opacity="0.3" />
    </Border.Effect>
    <TextBlock Text="Directional shadow" />
</Border>
```

| 属性 | 说明 |
|---|---|
| `ShadowDepth` | 阴影与元素之间的距离。默认值为 5。 |
| `Direction` | 以角度（0-360）表示的阴影方向。默认值为 315（右下）。 |
| `BlurRadius` | 阴影模糊半径。默认值为 5。 |
| `Color` | 阴影颜色。默认值为 `Black`。 |
| `Opacity` | 阴影不透明度。默认值为 1.0。 |

:::info
`Effect`（如 BlurEffect、DropShadowEffect）可应用到任意可视元素，包括文本和图像。`BoxShadow` 只适用于 `Border` 和 `ContentPresenter` 控件，但在绘制矩形阴影时性能更好。
:::

## 裁剪

任意控件上的 `ClipToBounds` 属性都可以裁剪超出元素边界的子内容。

```xml
<Border Width="100" Height="100" ClipToBounds="True" CornerRadius="50">
    <Image Source="avares://MyApp/Assets/photo.png"
           Stretch="UniformToFill" />
</Border>
```

这会通过圆角边框裁剪出一个圆形图像。

### Clip 属性

若要实现自定义裁剪形状，请配合 `Geometry` 使用 `Clip` 属性：

```xml
<Image Source="avares://MyApp/Assets/photo.png" Width="200" Height="200">
    <Image.Clip>
        <EllipseGeometry Rect="0,0,200,200" />
    </Image.Clip>
</Image>
```

你可以使用任意几何类型来进行裁剪：

```xml
<Image Source="avares://MyApp/Assets/photo.png" Width="200" Height="200">
    <Image.Clip>
        <PathGeometry>
            <PathFigure StartPoint="100,0" IsClosed="True">
                <LineSegment Point="200,80" />
                <LineSegment Point="160,200" />
                <LineSegment Point="40,200" />
                <LineSegment Point="0,80" />
            </PathFigure>
        </PathGeometry>
    </Image.Clip>
</Image>
```

## OpacityMask

`OpacityMask` 属性使用画刷控制逐像素透明度。只会使用遮罩画刷的 alpha 通道。黑色区域会完全可见，透明区域会被隐藏。

```xml
<!-- 从上到下淡出 -->
<Image Source="avares://MyApp/Assets/photo.png" Width="200" Height="200">
    <Image.OpacityMask>
        <LinearGradientBrush StartPoint="0%,0%" EndPoint="0%,100%">
            <GradientStop Color="Black" Offset="0" />
            <GradientStop Color="Black" Offset="0.5" />
            <GradientStop Color="Transparent" Offset="1" />
        </LinearGradientBrush>
    </Image.OpacityMask>
</Image>
```

### 径向淡出

```xml
<Border Width="200" Height="200" Background="SteelBlue">
    <Border.OpacityMask>
        <RadialGradientBrush>
            <GradientStop Color="Black" Offset="0" />
            <GradientStop Color="Transparent" Offset="1" />
        </RadialGradientBrush>
    </Border.OpacityMask>
</Border>
```

## Opacity

任意 `Visual` 上的 `Opacity` 属性会控制该元素及其所有子元素的整体透明度：

```xml
<Border Opacity="0.5" Background="Red" Padding="20">
    <TextBlock Text="Semi-transparent" />
</Border>
```

它的取值范围是 `0.0`（完全透明）到 `1.0`（完全不透明）。与 `OpacityMask` 不同，它会均匀地作用于整个元素。

### IsVisible 与 Opacity 的区别

| 方式 | 对布局的影响 | 交互 |
|---|---|---|
| `IsVisible="False"` | 元素会从布局中移除。 | 无法接收输入。 |
| `Opacity="0"` | 元素仍然占据空间。 | 仍然可以接收指针和键盘输入。 |

## 为效果添加动画

盒阴影和透明度都可以通过过渡来实现动画：

```xml
<Border Background="White" CornerRadius="8" Padding="20"
        BoxShadow="0 2 4 0 #20000000">
    <Border.Transitions>
        <Transitions>
            <BoxShadowsTransition Property="BoxShadow" Duration="0:0:0.2" />
            <DoubleTransition Property="Opacity" Duration="0:0:0.2" />
        </Transitions>
    </Border.Transitions>
    <Border.Styles>
        <Style Selector="Border:pointerover">
            <Setter Property="BoxShadow" Value="0 8 16 0 #30000000" />
        </Style>
    </Border.Styles>
    <TextBlock Text="Hover for shadow" />
</Border>
```

## 另请参阅

- [画刷](/docs/graphics-animation/brushes)：包括渐变和图像画刷在内的所有画刷类型。
- [绘制图形](/docs/graphics-animation/drawing-graphics)：形状、几何图形和路径数据。
- [变换](/docs/graphics-animation/transforms)：旋转、缩放、倾斜和平移元素。
- [控件过渡](/docs/graphics-animation/control-transitions)：为属性变化添加动画。
