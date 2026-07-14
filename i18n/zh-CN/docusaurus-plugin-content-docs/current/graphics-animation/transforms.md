---
id: transforms
title: 变换
description: 用于修改位置、大小、旋转和倾斜的渲染变换与布局变换。
doc-type: explanation
---

变换可以修改可视元素的位置、大小、旋转或倾斜，而不必直接改动其内容本身。Avalonia 支持渲染变换（在布局之后应用）和布局变换（通过 [`LayoutTransformControl`](/api/avalonia/controls/layouttransformcontrol) 在布局期间应用）。

## RenderTransform

每个 `Visual` 元素都提供 `RenderTransform` 属性。它会在布局计算完成后再应用变换，因此不会影响相邻控件的大小或位置。

```xml
<Button Content="Rotated" RenderTransformOrigin="50%,50%">
    <Button.RenderTransform>
        <RotateTransform Angle="15" />
    </Button.RenderTransform>
</Button>
```

### RenderTransformOrigin

`RenderTransformOrigin` 属性用于定义变换围绕哪个点进行，使用相对坐标表示。默认值为 `50%,50%`（元素中心）。

```xml
<Image Source="avares://MyApp/Assets/logo.png"
       RenderTransformOrigin="50%,50%">
    <Image.RenderTransform>
        <ScaleTransform ScaleX="1.5" ScaleY="1.5" />
    </Image.RenderTransform>
</Image>
```

## 变换类型

### RotateTransform

按指定角度（以度为单位）旋转元素。

| 属性 | 说明 |
|---|---|
| `Angle` | 旋转角度，以度为单位。正值表示顺时针旋转。 |
| `CenterX`, `CenterY` | 在 `RenderTransformOrigin` 基础上额外偏移的旋转中心，单位为设备无关像素。默认值为 0。 |

```xml
<Border Width="100" Height="100" Background="SteelBlue"
        RenderTransformOrigin="50%,50%">
    <Border.RenderTransform>
        <RotateTransform Angle="45" />
    </Border.RenderTransform>
</Border>
```

### ScaleTransform

对元素进行水平、垂直或同时双向缩放。

| 属性 | 说明 |
|---|---|
| `ScaleX` | 水平缩放系数。1.0 表示原尺寸，2.0 表示双倍，0.5 表示一半。 |
| `ScaleY` | 垂直缩放系数。 |

```xml
<!-- 宽度加倍，高度保持不变 -->
<TextBlock Text="拉伸" RenderTransformOrigin="50%,50%">
    <TextBlock.RenderTransform>
        <ScaleTransform ScaleX="2" ScaleY="1" />
    </TextBlock.RenderTransform>
</TextBlock>

<!-- 水平镜像 -->
<Image Source="avares://MyApp/Assets/arrow.png">
    <Image.RenderTransform>
        <ScaleTransform ScaleX="-1" ScaleY="1" />
    </Image.RenderTransform>
</Image>
```

### SkewTransform

沿 X 轴或 Y 轴对元素执行错切。

| 属性 | 说明 |
|---|---|
| `AngleX` | 水平错切角度（度）。 |
| `AngleY` | 垂直错切角度（度）。 |

```xml
<Border Width="100" Height="60" Background="Orange"
        RenderTransformOrigin="50%,50%">
    <Border.RenderTransform>
        <SkewTransform AngleX="20" />
    </Border.RenderTransform>
</Border>
```

### TranslateTransform

按指定偏移量移动元素，而不影响布局。

| 属性 | 说明 |
|---|---|
| `X` | 水平偏移量，单位为设备无关像素。 |
| `Y` | 垂直偏移量，单位为设备无关像素。 |

```xml
<TextBlock Text="已偏移" RenderTransformOrigin="50%,50%">
    <TextBlock.RenderTransform>
        <TranslateTransform X="20" Y="-10" />
    </TextBlock.RenderTransform>
</TextBlock>
```

### MatrixTransform

应用由 3x2 矩阵定义的任意二维仿射变换。

| 属性 | 说明 |
|---|---|
| `Matrix` | 形如 `m11,m12,m21,m22,offsetX,offsetY` 的字符串。 |

```xml
<Border Width="80" Height="80" Background="Purple">
    <Border.RenderTransform>
        <MatrixTransform Matrix="1,0,0.5,1,0,0" />
    </Border.RenderTransform>
</Border>
```

单位矩阵为 `1,0,0,1,0,0`（不进行任何变换）。

### TransformGroup

将多个变换组合为一个整体变换。各变换会按照出现顺序依次应用。

```xml
<Border Width="100" Height="60" Background="Teal"
        RenderTransformOrigin="50%,50%">
    <Border.RenderTransform>
        <TransformGroup>
            <ScaleTransform ScaleX="1.5" ScaleY="1.5" />
            <RotateTransform Angle="30" />
        </TransformGroup>
    </Border.RenderTransform>
</Border>
```

:::info
`TransformGroup` 中变换的顺序非常重要。先缩放再旋转，与先旋转再缩放，结果会不同。
:::

## 简写语法

Avalonia 为 `RenderTransform` 提供了类似 CSS 的简写语法：

```xml
<Border RenderTransform="rotate(45deg)" />
<Border RenderTransform="scale(2, 1)" />
<Border RenderTransform="translate(10px, 20px)" />
<Border RenderTransform="skew(15deg, 0deg)" />
```

多个变换可以串联：

```xml
<Border RenderTransform="scale(1.5) rotate(30deg)" />
```

## LayoutTransformControl

`RenderTransform` 不会影响布局计算，这意味着相邻控件不会感知该变换。如果你需要让变换参与布局过程（例如旋转侧边栏后让相邻内容自动调整），请将元素包裹在 `LayoutTransformControl` 中：

```xml
<LayoutTransformControl>
    <LayoutTransformControl.LayoutTransform>
        <RotateTransform Angle="90" />
    </LayoutTransformControl.LayoutTransform>
    <TextBlock Text="竖排文本" />
</LayoutTransformControl>
```

`LayoutTransformControl` 会在应用变换后再测量和排列其子元素，因此父面板会为变换后的尺寸分配空间。

## 对变换做动画

变换通常会通过关键帧动画或过渡效果来驱动。你可以对变换属性做动画，实现平滑旋转、缩放效果或位移。

```xml
<Border Width="80" Height="80" Background="Coral"
        RenderTransformOrigin="50%,50%">
    <Border.RenderTransform>
        <RotateTransform x:Name="MyRotation" Angle="0" />
    </Border.RenderTransform>
    <Border.Styles>
        <Style Selector="Border:pointerover">
            <Style.Animations>
                <Animation Duration="0:0:0.3">
                    <KeyFrame Cue="100%">
                        <Setter Property="RotateTransform.Angle" Value="90" />
                    </KeyFrame>
                </Animation>
            </Style.Animations>
        </Style>
    </Border.Styles>
</Border>
```

有关动画的更多信息，请参阅 [关键帧动画](/docs/graphics-animation/keyframe-animations) 和 [控件过渡](/docs/graphics-animation/control-transitions)。

## 在代码中使用变换

```csharp
var rotateTransform = new RotateTransform(45);
myBorder.RenderTransform = rotateTransform;
myBorder.RenderTransformOrigin = RelativePoint.Center;

// TransformGroup
var group = new TransformGroup();
group.Children.Add(new ScaleTransform(2, 2));
group.Children.Add(new RotateTransform(30));
myBorder.RenderTransform = group;

// 为变换属性设置值
rotateTransform.Angle = 90;
```

## 另请参阅

- [关键帧动画](/docs/graphics-animation/keyframe-animations)：让变换随时间变化。
- [控件过渡](/docs/graphics-animation/control-transitions)：在属性值变化时应用过渡效果。
- [图形绘制](/docs/graphics-animation/drawing-graphics)：形状与几何图形。
