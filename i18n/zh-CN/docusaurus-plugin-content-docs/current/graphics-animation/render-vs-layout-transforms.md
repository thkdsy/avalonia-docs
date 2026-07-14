---
id: render-vs-layout-transforms
title: 渲染变换与布局变换对比
description: Avalonia 中渲染变换与布局变换之间的差异。
doc-type: explanation
---

Avalonia 提供了两种变换控件的方式：渲染变换和布局变换。由于它们应用于渲染管线的不同阶段，因此会产生不同的视觉结果。

## 渲染变换

**渲染变换** 会改变控件的绘制方式，但不会影响布局。控件在布局系统中的位置和大小保持不变，其他控件也不会为了适应变换而移动。

```xml
<StackPanel Spacing="8">
    <Button Content="正常" />
    <Button Content="旋转（渲染）">
        <Button.RenderTransform>
            <RotateTransform Angle="15" />
        </Button.RenderTransform>
    </Button>
    <Button Content="下方按钮" />
</StackPanel>
```

在这个示例中，被旋转的按钮会在视觉上与相邻按钮重叠，因为布局过程并不会考虑这个旋转效果。

### RenderTransformOrigin

这是渲染变换的旋转/缩放原点。在 Avalonia 中，默认值是 `50%,50%`（控件中心）；而在 WPF 中，默认值则是 `0%,0%`（左上角）。

```xml
<!-- 围绕左上角旋转 -->
<Border RenderTransformOrigin="0%,0%">
    <Border.RenderTransform>
        <RotateTransform Angle="45" />
    </Border.RenderTransform>
</Border>

<!-- 围绕中心旋转（默认） -->
<Border>
    <Border.RenderTransform>
        <RotateTransform Angle="45" />
    </Border.RenderTransform>
</Border>
```

### 常见渲染变换类型

| 变换类型 | 说明 | 示例 |
|---|---|---|
| `RotateTransform` | 旋转控件。 | `<RotateTransform Angle="45" />` |
| `ScaleTransform` | 缩放控件。 | `<ScaleTransform ScaleX="1.5" ScaleY="1.5" />` |
| `TranslateTransform` | 在视觉上移动控件。 | `<TranslateTransform X="10" Y="-5" />` |
| `SkewTransform` | 倾斜控件。 | `<SkewTransform AngleX="15" />` |
| `TransformGroup` | 组合多个变换。 | 见下文。 |

### 组合变换

```xml
<Image Source="/assets/photo.jpg" Width="100" Height="100">
    <Image.RenderTransform>
        <TransformGroup>
            <ScaleTransform ScaleX="1.2" ScaleY="1.2" />
            <RotateTransform Angle="10" />
        </TransformGroup>
    </Image.RenderTransform>
</Image>
```

## 布局变换

**布局变换** 会在布局发生之前改变控件的大小和方向。父面板会看到变换后的尺寸，并据此重新安排其他控件的位置，因此可以避免重叠。

要应用布局变换，请使用 `LayoutTransformControl`：

```xml
<StackPanel Spacing="8">
    <Button Content="正常" />
    <LayoutTransformControl>
        <LayoutTransformControl.LayoutTransform>
            <RotateTransform Angle="15" />
        </LayoutTransformControl.LayoutTransform>
        <Button Content="旋转（布局）" />
    </LayoutTransformControl>
    <Button Content="下方按钮（位置正确）" />
</StackPanel>
```

这里“下方按钮”会被放置在旋转后控件完整边界的下方，因此不会发生重叠。

### 常见布局变换使用场景

| 场景 | 为什么使用布局变换 |
|---|---|
| 垂直文本标签 | 旋转后的文本应预留正确的空间。 |
| 缩放内容区域 | 相邻面板应适应缩放后的尺寸。 |
| 旋转表单字段 | 标签和输入控件应围绕旋转元素正确排布。 |

```xml
<!-- 在水平布局中仍能占据正确空间的垂直文本 -->
<StackPanel Orientation="Horizontal" Spacing="8">
    <LayoutTransformControl>
        <LayoutTransformControl.LayoutTransform>
            <RotateTransform Angle="-90" />
        </LayoutTransformControl.LayoutTransform>
        <TextBlock Text="垂直标签" />
    </LayoutTransformControl>
    <Border Background="LightBlue" Width="200" Height="100">
        <TextBlock Text="内容区域" VerticalAlignment="Center"
                   HorizontalAlignment="Center" />
    </Border>
</StackPanel>
```

## 对比

| 特性 | 渲染变换 | 布局变换 |
|---|---|---|
| 影响布局 | 否 | 是 |
| 其他控件是否会调整 | 否 | 是 |
| 性能 | 更快（不会重新布局） | 更慢（会触发布局过程） |
| 是否可动画化 | 是 | 是，但每一帧都会触发布局重算 |
| 应用方式 | `RenderTransform` 属性 | `LayoutTransformControl` |
| 默认原点 | 中心（50%, 50%） | 中心 |
| 是否有重叠风险 | 是 | 否 |

## 何时使用哪一种

**以下情况适合使用渲染变换：**
- 变换是临时性的，或者本身就是动画的一部分（如悬停效果、过渡）
- 性能很重要（动画不应触发布局）
- 与其他元素发生重叠是可以接受的，甚至是有意为之
- 你在制作视觉特效（例如视差、弹跳、抖动）

**以下情况适合使用布局变换：**
- 相邻控件必须尊重变换后的边界范围
- 你需要旋转文本标签，并且它必须正确预留空间
- 变换是永久布局的一部分，而不是动画
- 如果发生重叠会造成视觉错误

## 对变换做动画

渲染变换非常适合做动画，因为它不会触发布局：

```xml
<Border Background="Blue" Width="80" Height="80">
    <Border.Styles>
        <Style Selector="Border:pointerover">
            <Style.Animations>
                <Animation Duration="0:0:0.2">
                    <KeyFrame Cue="100%">
                        <Setter Property="ScaleTransform.ScaleX" Value="1.1" />
                        <Setter Property="ScaleTransform.ScaleY" Value="1.1" />
                    </KeyFrame>
                </Animation>
            </Style.Animations>
        </Style>
    </Border.Styles>
</Border>
```

在对性能敏感的场景中，应避免为布局变换做动画，因为每一帧都会触发完整布局过程。

## 另请参阅

- [变换](/docs/graphics-animation/transforms)：完整的变换参考。
- [动画](/docs/graphics-animation/animations)：关键帧动画与过渡动画。
- [性能](/docs/app-development/performance)：布局性能建议。
