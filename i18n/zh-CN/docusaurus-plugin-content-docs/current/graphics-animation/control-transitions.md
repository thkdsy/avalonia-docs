---
id: control-transitions
title: 设置控件过渡
description: 为 Avalonia 控件配置在属性变化时播放的过渡动画。
doc-type: how-to
---

Avalonia 中的过渡同样深受 CSS 动画启发。它会侦听目标属性值的变化，并根据配置参数对变化过程执行动画。你可以通过 [`Transitions`](/api/avalonia/animation/transitions) 属性在任意 `Control` 上定义过渡：

## 基本用法

```xml
<Window xmlns="https://github.com/avaloniaui">
    <Window.Styles>
        <Style Selector="Rectangle.red">
            <Setter Property="Height" Value="100"/>
            <Setter Property="Width" Value="100"/>
            <Setter Property="Fill" Value="Red"/>
            <Setter Property="Opacity" Value="0.5"/>
        </Style>
        <Style Selector="Rectangle.red:pointerover">
            <Setter Property="Opacity" Value="1"/>
        </Style>
    </Window.Styles>

    <Rectangle Classes="red">
        <Rectangle.Transitions>
            <Transitions>
                <DoubleTransition Property="Opacity" Duration="0:0:0.2"/>
            </Transitions>
        </Rectangle.Transitions>
    </Rectangle>

</Window>
```

上面的示例会侦听 `Rectangle` 的 `Opacity` 属性变化，并在值发生改变时，于 0.2 秒内将旧值平滑过渡到新值。

你也可以在任意样式中定义过渡：将 `Transitions` 作为 `Setter` 的目标属性，并把过渡项放在 `Transitions` 对象中，例如：

```xml
<Window xmlns="https://github.com/avaloniaui">
    <Window.Styles>
        <Style Selector="Rectangle.red">
            <Setter Property="Height" Value="100"/>
            <Setter Property="Width" Value="100"/>
            <Setter Property="Fill" Value="Red"/>
            <Setter Property="Opacity" Value="0.5"/>
            <Setter Property="Transitions">
                <Transitions>
                    <DoubleTransition Property="Opacity" Duration="0:0:0.2"/>
                </Transitions>
            </Setter>
        </Style>
        <Style Selector="Rectangle.red:pointerover">
            <Setter Property="Opacity" Value="1"/>
        </Style>
    </Window.Styles>

    <Rectangle Classes="red"/>

</Window>
```

每个过渡都有 `Property`、`Delay`、`Duration`，以及可选的 `Easing` 属性。

`Property` 表示过渡要侦听并执行动画的目标属性。

`Delay` 表示在将过渡应用到目标之前需要等待的时间。

`Duration` 表示过渡动画持续的时间。

缓动函数与[关键帧动画](/docs/graphics-animation/keyframe-animations#easing-function)中描述的缓动函数相同。

必须根据被动画化属性的类型选择正确的过渡类型：

| 过渡类型 | 属性类型 |
|---|---|
| `BoolTransition` | `bool` |
| `BoxShadowsTransition` | `BoxShadows` |
| `BrushTransition` | `IBrush` |
| `ColorTransition` | `Color` |
| `CornerRadiusTransition` | `CornerRadius` |
| `DoubleTransition` | `double` |
| `FloatTransition` | `float` |
| `IntegerTransition` | `int` |
| `PointTransition` | `Point` |
| `SizeTransition` | `Size` |
| `ThicknessTransition` | `Thickness` |
| `TransformOperationsTransition` | `ITransform` |
| `VectorTransition` | `Vector` |

## 为渲染变换添加过渡

使用类 CSS 语法应用到控件上的渲染变换可以添加过渡。下面的示例展示了一个 `Border`，当指针悬停其上时会旋转 45 度：

```xml title='XAML'
<Border Width="100" Height="100" Background="Red">
    <Border.Styles>
        <Style Selector="Border">
            <Setter Property="RenderTransform" Value="rotate(0)"/>
        </Style>
        <Style Selector="Border:pointerover">
            <Setter Property="RenderTransform" Value="rotate(45deg)"/>
        </Style>
    </Border.Styles>
    <Border.Transitions>
        <Transitions>
            <TransformOperationsTransition Property="RenderTransform" Duration="0:0:1"/>
        </Transitions>
    </Border.Transitions>
</Border>
```

```csharp title='C#'
new Border
{
    Width = 100,
    Height = 100,
    Background = Brushes.Red,
    Styles =
    {
        new Style(x => x.OfType<Border>())
        {
            Setters =
            {
                new Setter(
                    Border.RenderTransformProperty,
                    TransformOperations.Parse("rotate(0)"))
            },
        },
        new Style(x => x.OfType<Border>().Class(":pointerover"))
        {
            Setters =
            {
                new Setter(
                    Border.RenderTransformProperty,
                    TransformOperations.Parse("rotate(45deg)"))
            },
        },
    },
    Transitions = new Transitions
    {
        new TransformOperationsTransition
        {
            Property = Border.RenderTransformProperty,
            Duration = TimeSpan.FromSeconds(1),
        }
    }
};
```

可用的变换操作如下：

| 变换 | 示例 | 可接受单位 |
| ------------ | ----------------------------------------- | ---------------------------- |
| `translate`  | `translate(10px)`, `translate(0px, 10px)` | `px`                         |
| `translateX` | `translateX(10px)`                        | `px`                         |
| `translateY` | `translateY(10px)`                        | `px`                         |
| `scale`      | `scale(10)`, `scale(0, 10)`               |                              |
| `scaleX`     | `scaleX(10)`                              |                              |
| `scaleY`     | `scaleY(10)`                              |                              |
| `skew`       | `skew(90deg)`, `skew(0, 90deg)`           | `deg`, `grad`, `rad`, `turn` |
| `skewX`      | `skewX(90deg)`                            | `deg`, `grad`, `rad`, `turn` |
| `skewY`      | `skewY(90deg)`                            | `deg`, `grad`, `rad`, `turn` |
| `rotate`     | `rotate(90deg)`                           | `deg`, `grad`, `rad`, `turn` |
| `matrix`     | `matrix(1,2,3,4,5,6)`                     |                              |

:::info
Avalonia 也支持 WPF 风格的渲染变换，例如 `RotateTransform` 和 `ScaleTransform`。但这些变换不能应用过渡动画；如果你希望对渲染变换添加过渡，请始终使用类 CSS 的格式。
:::

## 另请参阅

- [关键帧动画](/docs/graphics-animation/keyframe-animations)：多步骤关键帧动画。
- [动画设置](/docs/graphics-animation/animation-settings)：持续时间、延迟、迭代次数和播放方向。
- [缓动函数](/docs/graphics-animation/easing-functions)：所有可用的缓动函数。
