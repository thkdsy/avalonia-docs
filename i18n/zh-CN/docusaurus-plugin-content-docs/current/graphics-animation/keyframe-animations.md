---
id: keyframe-animations
title: 使用关键帧动画
description: 在 XAML 中定义关键帧动画，并沿时间轴对控件属性进行动画处理。
doc-type: how-to
---

import AnimationKeyframeDiagram from '/img/guides/ui-development/graphics/animation-keyframe.png';
import KeyframeFadeScreenshot from '/img/guides/ui-development/graphics/keyframe-fade.gif';
import KeyframeCompositeAnimationScreenshot from '/img/guides/ui-development/graphics/keyframe-composite-animation.gif';
import LinearEasingScreenshot from '/img/guides/ui-development/graphics/linear-easing.gif';
import BounceEaseInScreenshot from '/img/guides/ui-development/graphics/bounce-ease-in.gif';

你可以使用关键帧动画，沿着时间轴改变一个或多个控件属性。关键帧定义在 _Avalonia UI_ 的样式中，并在动画持续时间内设置多个 **cue** 点，用来指定属性在某些时间点上的中间值。

<Image light={AnimationKeyframeDiagram} alt="Diagram showing keyframe animation timeline with cue points" position="center" maxWidth={400} cornerRadius="true"/>

关键帧之间的属性值会按照某个缓动函数的曲线进行插值。默认的缓动函数是线性插值。

动画在被触发后开始运行，并且可以以任意方向运行任意次数。你还可以为动画设置启动延迟，以及重复播放行为。

:::info
如果你熟悉 CSS 中的关键帧动画，那么你会很容易发现它与 _Avalonia UI_ 的实现方式非常相似。
:::

## 示例

关键帧动画是通过样式来定义的。

:::info
如果你想回顾 _Avalonia UI_ 中样式的用法，请参阅 [样式概念](/docs/styling/styles)。
:::

按照以下步骤，用 XAML 定义一个简单的颜色淡入动画：

- 在你选定的层级创建一个样式集合。
- 向集合中添加一个样式，并使用能匹配目标控件的选择器。
- 添加一个 `Setter` 元素，用于定义动画将要改变的属性。本例使用 `<Setter Property="Fill" Value="Red"/>`。
- 添加一个 `Style.Animations` 元素来承载动画。
- 添加一个 `Animation` 元素，并设置它的 `Duration` 属性，格式为 `"Hours:Minutes:Seconds"`。
- 接着定义动画的关键帧。本例在 0% 和 100% 位置设置 cue。
- 在每个关键帧中添加 `Setter` 元素来设置填充不透明度的值。本例会在 0.0 与 1.0 之间进行动画过渡。

最终代码如下：

```xml
<Window xmlns="https://github.com/avaloniaui">
    <Window.Styles>
        <Style Selector="Rectangle.red">
            <Setter Property="Fill" Value="Red"/>
            <Style.Animations>
                <Animation Duration="0:0:3"> 
                    <KeyFrame Cue="0%">
                        <Setter Property="Opacity" Value="0.0"/>
                    </KeyFrame>
                    <KeyFrame Cue="100%">
                        <Setter Property="Opacity" Value="1.0"/>
                    </KeyFrame>
                </Animation>
            </Style.Animations>
        </Style>
    </Window.Styles>

    <Rectangle Classes="red" Width="100" Height="100"/>
</Window>
```

最终动画效果如下：

<Image light={KeyframeFadeScreenshot} alt="Animation showing a red rectangle fading in" position="center" maxWidth={400} cornerRadius="true"/>

只要矩形控件被加载并且能够被该样式匹配到，动画就会立即开始运行。事实上，它甚至也会在预览窗格中运行。

## 同时对两个属性做动画

这个示例演示如何在同一条时间轴上同时对两个属性执行动画。

```xml
<Window.Styles>
    <Style Selector="Rectangle.red">
      <Setter Property="Fill" Value="Red"/>
      <Style.Animations>
        <Animation Duration="0:0:3" IterationCount="4">
          <KeyFrame Cue="0%">
            <Setter Property="Opacity" Value="0.0"/>
            <Setter Property="RotateTransform.Angle" Value="0.0"/>
          </KeyFrame>
          <KeyFrame Cue="100%"> 
            <Setter Property="Opacity" Value="1.0"/>
            <Setter Property="RotateTransform.Angle" Value="90.0"/>
          </KeyFrame>
        </Animation> 
    </Style.Animations>
    </Style>
  </Window.Styles>
```

这个红色矩形会在淡入的同时发生旋转。

<Image light={KeyframeCompositeAnimationScreenshot} alt="Animation showing a red rectangle fading in and rotating simultaneously" position="center" maxWidth={400} cornerRadius="true"/>

## 配置动画

### Delay

你可以通过设置动画元素的 `Delay` 属性，为动画启动增加延迟。例如：

```xml
<Animation Duration="0:0:1"
           Delay="0:0:1"> 
    ...
</Animation>
```

### Repeat

你可以让动画重复指定次数，或者无限循环。若要设置有限次数的重复，请在动画元素上设置 `IterationCount` 属性：

```xml
<Animation IterationCount="5">
    ...
</Animation>
```

如果要让动画无限重复，请使用特殊值 `"INFINITE"`。例如：

```xml
<Animation IterationCount="INFINITE">
    ...
</Animation>
```

### 播放方向

默认情况下，动画会正向播放，并按照缓动函数从左到右的曲线轮廓执行。你可以通过在动画元素上设置 `PlaybackDirection` 属性来改变这一行为：

```xml
<Animation IterationCount="9" PlaybackDirection="AlternateReverse">
    ...
</Animation>
```

下表说明了可用选项：

<table><thead><tr><th width="245">取值</th><th>说明</th></tr></thead><tbody><tr><td><code>Normal</code></td><td>（默认）动画正向播放。</td></tr><tr><td><code>Reverse</code></td><td>动画反向播放。</td></tr><tr><td><code>Alternate</code></td><td>动画先正向播放，再反向播放。</td></tr><tr><td><code>AlternateReverse</code></td><td>动画先反向播放，再正向播放。</td></tr></tbody></table>

### 填充模式

动画的填充模式属性定义了：动画运行结束后，或两次运行之间存在空档时，所设置的属性值应如何保持。例如：

```xml
<Animation IterationCount="9" FillMode="Backward">
    ...
</Animation>
```

下表说明了可用选项：

<table><thead><tr><th width="240">取值</th><th>说明</th></tr></thead><tbody><tr><td><code>None</code></td><td>动画结束后不会保留值；如果动画有延迟，也不会提前应用第一个值。</td></tr><tr><td><code>Forward</code></td><td>最后一个插值结果会保留到目标属性上。</td></tr><tr><td><code>Backward</code></td><td>在动画延迟期间会显示第一个插值值。</td></tr><tr><td><code>Both</code></td><td>同时应用 <code>Forward</code> 与 <code>Backward</code> 的行为。</td></tr></tbody></table>

### 缓动函数

缓动函数定义了动画执行过程中属性值如何随时间变化。

<div>

<Image light={LinearEasingScreenshot} alt="Graph showing linear easing function" position="center" maxWidth={400} cornerRadius="true"/>

<Image light={BounceEaseInScreenshot} alt="Graph showing bounce ease-in easing function" position="center" maxWidth={400} cornerRadius="true"/>

</div>

默认缓动函数是线性的（上图左侧），但你可以通过在 `Easing` 属性中设置所需函数名称来使用其他模式。例如，若要使用 “bounce ease in” 函数（上图右侧）：

```xml
<Animation Duration="0:0:1"
           Delay="0:0:1"
           Easing="BounceEaseIn"> 
    ...
</Animation>
```

:::info
有关 _Avalonia UI_ 缓动函数的完整列表，请参阅[动画设置参考](/docs/graphics-animation/animation-settings)。
:::

你也可以像下面这样添加自己的自定义缓动函数类：

```xml
<Animation Duration="0:0:1"
           Delay="0:0:1">
    <Animation.Easing>
        <local:YourCustomEasingClassHere/>
    </Animation.Easing> 
    ...
</Animation>
```

## 从代码后置中运行动画

在某些场景下，相比 XAML 样式选择器，开发者可能需要对动画生命周期进行更灵活的控制。最简单的方式，是把动画定义在 `Resources` 字典中。

以这种方式定义 `Animation` 时，务必同时指定 `x:Key` 和 `x:SetterTargetType`。前者用于通过键访问动画，后者则帮助编译器创建强类型的 setter。

```xml
<Window xmlns="https://github.com/avaloniaui">
    <Window.Resources>
        <Animation x:Key="ResourceAnimation"
                   x:SetterTargetType="Rectangle"
                   Duration="0:0:3"> 
            <KeyFrame Cue="0%">
                <Setter Property="Opacity" Value="0.0"/>
            </KeyFrame>
            <KeyFrame Cue="100%">
                <Setter Property="Opacity" Value="1.0"/>
            </KeyFrame>
        </Animation>
    </Window.Resources>

    <Rectangle x:Name="Rect" />
</Window>
```

现在，就可以在自定义代码后置处理程序中访问并执行这个动画。

```csharp
var animation = (Animation)this.Resources["ResourceAnimation"];
// 在 Rect 控件上运行动画
await animation.RunAsync(Rect);
```

`RunAsync` 会返回一个任务，该任务会在动画完成时结束。如果动画是无限循环或持续重复的，那么这个任务将不会结束，除非你通过向 `RunAsync` 方法传入 `CancellationToken` 从外部取消它。

:::info
虽然在 XAML 中定义动画通常更简单，但你也完全可以用 C# 代码来完成。你可以直接创建 `Animation` 类型实例，并填充其关键帧集合。
:::

## See also

- [动画设置](/docs/graphics-animation/animation-settings)：持续时间、延迟、迭代次数和播放方向。
- [缓动函数](/docs/graphics-animation/easing-functions)：所有可用的缓动函数。
- [控件过渡](/docs/graphics-animation/control-transitions)：通过过渡对属性变化做动画处理。
