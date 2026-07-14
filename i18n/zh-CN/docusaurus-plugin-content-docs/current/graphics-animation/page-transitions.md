---
id: page-transitions
title: 设置页面过渡
description: 在 Avalonia 中使用 CrossFade、PageSlide、CompositePageTransition 或自定义过渡来配置视图切换动画。
doc-type: how-to
---

import CustomPageTransitionScreenshot from '/img/guides/ui-development/graphics/custom-page-transition.webp';

页面过渡用于在两个视图之间添加动画。通常你会将它们与 `TransitioningContentControl` 或 `Carousel` 这类控件一起使用。Avalonia 提供了两种内置过渡，并允许你组合或创建自定义过渡。

要指定过渡效果，请在宿主控件上设置 `PageTransition` 属性：

```xml title='XAML'
<TransitioningContentControl PageTransition="{StaticResource MyTransition}"
                              Content="{Binding CurrentPage}" />
```

## [`CrossFade`](/api/avalonia/animation/crossfade)

`CrossFade` 通过动画化不透明度，让当前视图淡出、新视图淡入。当你需要一种轻柔、没有方向感、适用于各种内容的默认过渡时，它是非常合适的选择。

```xml title='XAML'
<CrossFade Duration="0:00:00.500" />
```

```csharp title='C#'
var transition = new CrossFade(TimeSpan.FromMilliseconds(500));
```

:::tip
`CrossFade` 很适合标签页式导航，因为这类导航本身通常没有明确的前进/后退方向。如果你的视图高度差异很大，交叉淡入淡出还能避免滑动过渡带来的突兀跳动感。
:::

## [`PageSlide`](/api/avalonia/animation/pageslide)

`PageSlide` 会让旧视图滑出、新视图滑入。`Orientation` 属性用于控制滑动轴向（默认是水平）。这种过渡能表达明确的导航方向，因此非常适合向导式流程或顺序页面。

```xml title='XAML'
<PageSlide Duration="0:00:00.500" Orientation="Vertical" />
```

```csharp title='C#'
var transition = new PageSlide(TimeSpan.FromMilliseconds(500),
                                PageSlide.SlideAxis.Vertical);
```

:::tip
传给过渡的 `forward` 参数会决定滑动方向。当你进行反向导航时（例如点击“返回”按钮），应将该参数设为 `false`，这样滑动方向就会自动反转。
:::

## [`CompositePageTransition`](/api/avalonia/animation/compositepagetransition)

`CompositePageTransition` 可以将两个或更多过渡组合成单一效果。每个子过渡会并行运行。下面的示例会让视图沿对角线滑动（同时叠加水平与垂直滑动），并同时执行交叉淡入淡出：

```xml title='XAML'
<CompositePageTransition>
    <CrossFade Duration="0:00:00.500" />
    <PageSlide Duration="0:00:00.500" Orientation="Horizontal" />
    <PageSlide Duration="0:00:00.500" Orientation="Vertical" />
</CompositePageTransition>
```

```csharp title='C#'
var transition = new CompositePageTransition();
transition.PageTransitions.Add(new CrossFade(TimeSpan.FromMilliseconds(500)));
transition.PageTransitions.Add(new PageSlide(TimeSpan.FromMilliseconds(500),
    PageSlide.SlideAxis.Horizontal));
transition.PageTransitions.Add(new PageSlide(TimeSpan.FromMilliseconds(500),
    PageSlide.SlideAxis.Vertical));
```

:::note
请让所有子过渡的 `Duration` 保持一致，这样它们才能同时开始并同时结束。若持续时间不一致，可能会出现某个过渡先结束、导致视觉异常的情况。
:::

## 选择合适的过渡效果

| 过渡类型 | 最适合 | 说明 |
|---|---|---|
| `CrossFade` | 标签页、设置面板、原位切换的内容 | 轻柔、无方向感 |
| `PageSlide`（水平） | 向导步骤、前进/后退导航 | 能表达顺序流程 |
| `PageSlide`（垂直） | 展开区域、逐层深入导航 | 能暗示层级关系 |
| `CompositePageTransition` | 更丰富的分层效果 | 可组合以上任意效果 |

## 自定义页面过渡

当内置过渡不符合你的设计需求时，可以通过实现 `IPageTransition` 接口来创建自定义过渡。这个接口只有一个方法：

```csharp title='C#'
public Task Start(Visual? from, Visual? to, bool forward,
                  CancellationToken cancellationToken)
{
    // 将旧视图（from）过渡到新视图（to）
}
```

`from` 和 `to` 参数都可能为 `null`（例如控件首次加载时，并不存在要退出的旧视图）。`forward` 参数表示导航方向，因此你可以据此反转动画。务必正确处理 `cancellationToken`，以便当用户在过渡尚未完成时再次导航，Avalonia 能够取消当前过渡。

下面的示例会先将旧视图沿垂直方向收缩，再展开新视图：

```csharp title='C#'
using Avalonia.VisualTree;

public class CustomTransition : IPageTransition
{
    public CustomTransition() { }

    public CustomTransition(TimeSpan duration)
    {
        Duration = duration;
    }

    public TimeSpan Duration { get; set; }

    public async Task Start(Visual from, Visual to, bool forward,
                            CancellationToken cancellationToken)
    {
        if (cancellationToken.IsCancellationRequested)
        {
            return;
        }

        var tasks = new List<Task>();
        var scaleYProperty = ScaleTransform.ScaleYProperty;

        if (from != null)
        {
            var animation = new Animation
            {
                FillMode = FillMode.Forward,
                Children =
                {
                    new KeyFrame
                    {
                        Setters = { new Setter { Property = scaleYProperty, Value = 1d } },
                        Cue = new Cue(0d)
                    },
                    new KeyFrame
                    {
                        Setters = { new Setter { Property = scaleYProperty, Value = 0d } },
                        Cue = new Cue(1d)
                    }
                },
                Duration = Duration
            };
            tasks.Add(animation.RunAsync(from, cancellationToken));
        }

        if (to != null)
        {
            to.IsVisible = true;
            var animation = new Animation
            {
                FillMode = FillMode.Forward,
                Children =
                {
                    new KeyFrame
                    {
                        Setters = { new Setter { Property = scaleYProperty, Value = 0d } },
                        Cue = new Cue(0d)
                    },
                    new KeyFrame
                    {
                        Setters = { new Setter { Property = scaleYProperty, Value = 1d } },
                        Cue = new Cue(1d)
                    }
                },
                Duration = Duration
            };
            tasks.Add(animation.RunAsync(to, cancellationToken));
        }

        await Task.WhenAll(tasks);

        if (from != null && !cancellationToken.IsCancellationRequested)
        {
            from.IsVisible = false;
        }
    }
}
```

<Image light={CustomPageTransitionScreenshot} alt="演示自定义页面过渡：视图沿垂直方向收缩与展开的动画" position="center" maxWidth={400} cornerRadius="true"/>

## 另请参阅

- [动画](/docs/graphics-animation/animations)：Avalonia 中动画类型的概览。
- [关键帧动画](/docs/graphics-animation/keyframe-animations)：多步骤关键帧动画。
- [控件过渡](/docs/graphics-animation/control-transitions)：为属性变化添加动画。
- [在视图之间导航](/docs/how-to/navigation-how-to)：切换视图的常见模式。
