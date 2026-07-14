---
id: easing-functions
title: 缓动函数
description: Avalonia 中可用于控制动画时间曲线的缓动函数。
doc-type: reference
---

缓动函数用于控制动画过程中的变化速率，让运动看起来更自然。与恒定速度的线性动画不同，缓动函数可以让数值加速、减速或产生弹跳效果，从而创建更生动的过渡。

## 使用缓动函数

可以在关键帧动画中的 `KeyFrame` 上指定缓动函数：

```xml
<Border Background="Blue" Width="50" Height="50">
    <Border.Styles>
        <Style Selector="Border">
            <Style.Animations>
                <Animation Duration="0:0:1" IterationCount="Infinite" PlaybackDirection="Alternate">
                    <KeyFrame Cue="0%" KeySpline="0.1,0.9,0.2,1.0">
                        <Setter Property="TranslateTransform.X" Value="0" />
                    </KeyFrame>
                    <KeyFrame Cue="100%">
                        <Setter Property="TranslateTransform.X" Value="300" />
                    </KeyFrame>
                </Animation>
            </Style.Animations>
        </Style>
    </Border.Styles>
</Border>
```

也可以在整个 `Animation` 上设置缓动函数：

```xml
<Animation Duration="0:0:0.5" Easing="CubicEaseOut">
    <KeyFrame Cue="0%">
        <Setter Property="Opacity" Value="0" />
    </KeyFrame>
    <KeyFrame Cue="100%">
        <Setter Property="Opacity" Value="1" />
    </KeyFrame>
</Animation>
```

缓动函数同样可用于 `Transitions`，以控制属性变化时的动画：

```xml
<Border.Transitions>
    <DoubleTransition Property="Opacity" Duration="0:0:0.3" Easing="QuadraticEaseOut" />
</Border.Transitions>
```

## 内置缓动函数

Avalonia 提供了一套完整的缓动函数。每个函数通常都有三种变体：

- **EaseIn**：开始较慢，随后逐渐加速
- **EaseOut**：开始较快，随后逐渐减速
- **EaseInOut**：开始较慢，中段加速，结束前再减速

### Linear

| 名称 | 行为 |
|---|---|
| `LinearEasing` | 恒定速度，不加速也不减速。 |

### Sine

基于正弦曲线的缓动，可产生柔和、平滑的运动效果。

| 名称 | 变体效果 |
|---|---|
| `SineEaseIn` | 起步慢，随后加速 |
| `SineEaseOut` | 起步快，随后减速 |
| `SineEaseInOut` | 开始和结束都较慢 |

### Quadratic

基于二次曲线（t^2）的缓动，比正弦缓动更明显一些。

| 名称 | 变体效果 |
|---|---|
| `QuadraticEaseIn` | 开始较慢 |
| `QuadraticEaseOut` | 开始较快 |
| `QuadraticEaseInOut` | 开始和结束都较慢 |

### Cubic

基于三次曲线（t^3）的缓动，比二次缓动更明显。

| 名称 | 变体效果 |
|---|---|
| `CubicEaseIn` | 开始较慢 |
| [`CubicEaseOut`](/api/avalonia/animation/easings/cubiceaseout) | 开始较快 |
| `CubicEaseInOut` | 开始和结束都较慢 |

### Quartic

基于 t^4 的缓动，加速效果比三次缓动更强。

| 名称 | 变体效果 |
|---|---|
| `QuarticEaseIn` | 开始较慢 |
| `QuarticEaseOut` | 开始较快 |
| `QuarticEaseInOut` | 开始和结束都较慢 |

### Quintic

基于 t^5 的缓动，是多项式缓动中最强烈的一类。

| 名称 | 变体效果 |
|---|---|
| `QuinticEaseIn` | 开始较慢 |
| `QuinticEaseOut` | 开始较快 |
| `QuinticEaseInOut` | 开始和结束都较慢 |

### Exponential

基于指数曲线的缓动，会产生非常明显的加速效果。

| 名称 | 变体效果 |
|---|---|
| `ExponentialEaseIn` | 起步很慢，随后急剧加速 |
| `ExponentialEaseOut` | 急剧减速，结束非常缓慢 |
| `ExponentialEaseInOut` | 加速和减速都很明显 |

### Circular

基于圆形曲线的缓动，可产生较自然的运动感觉。

| 名称 | 变体效果 |
|---|---|
| `CircularEaseIn` | 开始较慢 |
| `CircularEaseOut` | 开始较快 |
| `CircularEaseInOut` | 开始和结束都较慢 |

### Back

先超过目标值再回到目标位置，产生“回拉”效果。

| 名称 | 变体效果 |
|---|---|
| `BackEaseIn` | 先向后拉，再向前加速 |
| [`BackEaseOut`](/api/avalonia/animation/easings/backeaseout) | 超过目标后再回落稳定 |
| `BackEaseInOut` | 回拉、超调，然后稳定 |

### Bounce

模拟在边界处弹跳的效果。

| 名称 | 变体效果 |
|---|---|
| `BounceEaseIn` | 开始阶段弹跳 |
| `BounceEaseOut` | 结束阶段弹跳 |
| `BounceEaseInOut` | 开始和结束都弹跳 |

### Elastic

模拟带振荡的弹簧或橡皮筋效果。

| 名称 | 变体效果 |
|---|---|
| `ElasticEaseIn` | 开始阶段振荡 |
| [`ElasticEaseOut`](/api/avalonia/animation/easings/elasticeaseout) | 结束阶段振荡 |
| `ElasticEaseInOut` | 两端都振荡 |

## 选择缓动函数

各类缓动函数的常见使用场景：

| 场景 | 推荐缓动 |
|---|---|
| 淡入/淡出 | `QuadraticEaseOut` 或 `CubicEaseOut` |
| 面板滑入 | `CubicEaseOut` |
| 面板滑出 | `CubicEaseIn` |
| 按钮按压反馈 | `QuadraticEaseInOut` |
| 展开/折叠 | `CubicEaseInOut` |
| 通知弹出 | `BackEaseOut` |
| 活泼的弹跳效果 | `BounceEaseOut` |
| 类弹簧运动 | `ElasticEaseOut` 或 `SpringEasing` |
| 轻微悬停效果 | `SineEaseOut` |

## SplineEasing（自定义三次贝塞尔曲线）

如果内置函数都不符合需求，可以使用 `SplineEasing`，通过四个控制点来定义一条三次贝塞尔曲线：

```xml
<Animation Duration="0:0:0.5">
    <Animation.Easing>
        <SplineEasing X1="0.25" Y1="0.1" X2="0.25" Y2="1.0" />
    </Animation.Easing>
    <KeyFrame Cue="0%">
        <Setter Property="Opacity" Value="0" />
    </KeyFrame>
    <KeyFrame Cue="100%">
        <Setter Property="Opacity" Value="1" />
    </KeyFrame>
</Animation>
```

你也可以使用 `KeySpline` 简写并以逗号分隔值的方式，内联指定样条缓动：

```xml
<KeyFrame Cue="0%" KeySpline="0.25,0.1,0.25,1.0">
    <Setter Property="TranslateTransform.X" Value="0" />
</KeyFrame>
```

这四个值（`X1`、`Y1`、`X2`、`Y2`）定义了从 (0,0) 到 (1,1) 的三次贝塞尔曲线的两个控制点。它们与 CSS 的 `cubic-bezier()` 参数一致。常见预设如下：

| 曲线 | X1 | Y1 | X2 | Y2 | 对应 |
|---|---|---|---|---|---|
| ease | 0.25 | 0.1 | 0.25 | 1.0 | CSS `ease` |
| ease-in | 0.42 | 0 | 1.0 | 1.0 | CSS `ease-in` |
| ease-out | 0 | 0 | 0.58 | 1.0 | CSS `ease-out` |
| ease-in-out | 0.42 | 0 | 0.58 | 1.0 | CSS `ease-in-out` |

## SpringEasing

`SpringEasing` 用于模拟基于物理的弹簧运动。与内置的弹性缓动不同，弹簧缓动允许你控制弹簧的物理参数：

```xml
<Animation Duration="0:0:1">
    <Animation.Easing>
        <SpringEasing Mass="1" Stiffness="100" Damping="10" InitialVelocity="0" />
    </Animation.Easing>
    <KeyFrame Cue="0%">
        <Setter Property="TranslateTransform.Y" Value="-50" />
    </KeyFrame>
    <KeyFrame Cue="100%">
        <Setter Property="TranslateTransform.Y" Value="0" />
    </KeyFrame>
</Animation>
```

### 弹簧参数

| 参数 | 说明 | 增大后的效果 |
|---|---|---|
| `Mass` | 弹簧上物体的质量 | 运动更慢、更沉重 |
| `Stiffness` | 弹簧的刚度 | 振荡更快、响应更利落 |
| `Damping` | 让弹簧减速的摩擦/阻尼 | 振荡更少、更快稳定 |
| `InitialVelocity` | 运动的初始速度 | 初始运动更明显 |

较低的阻尼会产生更“弹”的运动效果。较高的阻尼则会产生过阻尼运动，数值会趋近目标而几乎不发生振荡。

## 自定义缓动函数

你可以通过继承 `Easing` 并重写 `Ease` 方法来创建自定义缓动函数：

```csharp
using Avalonia.Animation.Easings;

public class StepEasing : Easing
{
    public int Steps { get; set; } = 4;

    public override double Ease(double progress)
    {
        return Math.Floor(progress * Steps) / Steps;
    }
}
```

在 XAML 中通过引用命名空间来使用自定义缓动：

```xml
<Animation Duration="0:0:1">
    <Animation.Easing>
        <local:StepEasing Steps="8" />
    </Animation.Easing>
    <!-- keyframes -->
</Animation>
```

`Ease` 方法接收一个从 0.0 到 1.0 的 `progress` 值，表示线性的时间进度，并返回一个修改后的值（通常同样位于 0.0 到 1.0 之间，但像 `BackEaseOut` 和 `ElasticEaseOut` 这样的效果允许超出该范围）。

## 另请参阅

- [关键帧动画](/docs/graphics-animation/keyframe-animations)：在动画中使用关键帧和缓动。
- [控件过渡](/docs/graphics-animation/control-transitions)：属性变化时自动执行的过渡动画。
- [动画设置](/docs/graphics-animation/animation-settings)：持续时间、延迟、迭代次数和播放方向。
