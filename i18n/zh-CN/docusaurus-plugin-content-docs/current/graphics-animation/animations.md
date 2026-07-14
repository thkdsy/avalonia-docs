---
id: animations
title: 动画
description: 概览 Avalonia 中的动画类型，包括关键帧、过渡和合成动画。
doc-type: overview
---

import KeyframeDiagram from '/img/concepts/ui-concepts/animations/animation-keyframe.png';

Avalonia 提供三种动画类型：

| 类型 | 说明 | 适用场景 |
|---|---|---|
| [关键帧动画](/docs/graphics-animation/keyframe-animations) | 通过多个关键帧，沿时间轴改变一个或多个属性。 | 由样式选择器触发的复杂多步骤动画。 |
| [控件过渡](/docs/graphics-animation/control-transitions) | 在属性值变化时，对单个属性执行动画。 | 为属性变化提供平滑的视觉反馈（如透明度、颜色、尺寸）。 |
| [合成动画](/docs/graphics-animation/composition-animations) | 由代码驱动，并运行在渲染线程上的动画。 | 对性能敏感、或需要通过 C# 编程控制的动画。 |

此外，[页面过渡](/docs/graphics-animation/page-transitions) 还可以为 `TransitioningContentControl` 和 `Carousel` 这类控件中的内容切换添加动画。

## 关键帧动画

最简单的关键帧动画会在指定时长内改变某个属性值，通常只需定义两个关键帧：起点（0%）和终点（100%）。

<Image light={KeyframeDiagram} alt="Diagram showing a keyframe animation timeline with start and end cue points" position="center" maxWidth={400} cornerRadius="true"/>

关键帧之间的属性值会通过缓动函数进行插值计算。默认使用的是线性插值。

### 快速示例

```xml
<Border Background="Blue" Width="100" Height="100">
    <Border.Styles>
        <Style Selector="Border">
            <Style.Animations>
                <Animation Duration="0:0:1" IterationCount="INFINITE"
                           PlaybackDirection="Alternate">
                    <KeyFrame Cue="0%">
                        <Setter Property="Opacity" Value="1.0" />
                    </KeyFrame>
                    <KeyFrame Cue="100%">
                        <Setter Property="Opacity" Value="0.3" />
                    </KeyFrame>
                </Animation>
            </Style.Animations>
        </Style>
    </Border.Styles>
</Border>
```

这会创建一个持续循环的透明度脉冲动画，在完全不透明和部分透明之间交替变化。

完整语法和更多示例请参阅 [关键帧动画](/docs/graphics-animation/keyframe-animations)。

## 控件过渡

过渡会在属性值发生变化时自动执行动画，因此无需显式编写关键帧，也能提供平滑的视觉反馈：

```xml
<Button Content="Hover me" Background="Blue">
    <Button.Transitions>
        <Transitions>
            <BrushTransition Property="Background" Duration="0:0:0.3" />
            <DoubleTransition Property="Opacity" Duration="0:0:0.2" />
        </Transitions>
    </Button.Transitions>
</Button>
```

关于过渡类型及其配置，请参阅 [控件过渡](/docs/graphics-animation/control-transitions)。

## 合成动画

合成动画提供了一种更底层、由代码驱动的方式，并且运行在渲染线程上。当你需要程序化控制，或者需要渲染线程级别的性能时，可以使用它们：

```csharp
var visual = ElementComposition.GetElementVisual(myControl);
var compositor = visual.Compositor;

var animation = compositor.CreateVector3KeyFrameAnimation();
animation.Duration = TimeSpan.FromMilliseconds(400);
animation.InsertKeyFrame(0f, new Vector3D(-200, 0, 0));
animation.InsertKeyFrame(1f, new Vector3D(0, 0, 0));

visual.StartAnimation("Offset", animation);
```

有关完整 API、隐式动画以及集成模式，请参阅 [合成动画](/docs/graphics-animation/composition-animations)。

## 触发动画

在 XAML 中定义的关键帧动画，通常依赖样式选择器来决定何时触发：

- **无条件选择器**（例如 `Style Selector="Border"`）：控件进入视觉树时动画就会开始。
- **条件选择器**（例如 `Style Selector="Border:pointerover"`）：当选择器匹配时（例如指针悬停在边框上）动画运行，不再匹配时动画停止。

```xml
<Style Selector="Border:pointerover">
    <Style.Animations>
        <Animation Duration="0:0:0.3">
            <KeyFrame Cue="100%">
                <Setter Property="ScaleTransform.ScaleX" Value="1.1" />
                <Setter Property="ScaleTransform.ScaleY" Value="1.1" />
            </KeyFrame>
        </Animation>
    </Style.Animations>
</Style>
```

## 动画设置

关键帧动画支持以下配置项：

| 设置项 | 说明 | 示例 |
|---|---|---|
| `Duration` | 单次动画周期的时长。 | `0:0:1`（1 秒） |
| `Delay` | 动画开始前等待的时间。 | `0:0:0.5` |
| `IterationCount` | 重复次数。若要无限循环，使用 `INFINITE`。 | `3`、`INFINITE` |
| `PlaybackDirection` | 播放方向。 | `Normal`、`Reverse`、`Alternate`、`AlternateReverse` |
| `FillMode` | 动画结束后的行为。 | `Forward`、`Backward`、`Both`、`None` |
| `Easing` | 关键帧之间的插值曲线。 | `CubicEaseInOut` |

有关每个选项的详细说明，请参阅 [动画设置](/docs/graphics-animation/animation-settings)；要查看所有可用缓动类型，请参阅 [缓动函数](/docs/graphics-animation/easing-functions)。

## 另请参阅

- [关键帧动画](/docs/graphics-animation/keyframe-animations)：完整的关键帧动画语法和示例。
- [控件过渡](/docs/graphics-animation/control-transitions)：为属性变化添加动画。
- [合成动画](/docs/graphics-animation/composition-animations)：由代码驱动、运行在渲染线程上的动画。
- [页面过渡](/docs/graphics-animation/page-transitions)：为内容切换添加动画。
