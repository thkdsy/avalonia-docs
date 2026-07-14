---
id: gestures
title: 手势
description: 内置手势事件、手势识别器，以及针对触摸、笔和鼠标输入的自定义手势处理。
doc-type: overview
---

Avalonia 使用统一的指针事件系统。鼠标、触摸和手写笔输入都会通过同一套 `PointerPressed`、`PointerMoved` 和 `PointerReleased` 事件流转，而不是为每种设备分别设计独立事件类型。指针事件描述的是硬件层面的动作：例如某个按钮被按下，或者某根手指发生了移动。

手势是在指针事件之上构建的更高层抽象，用来表达用户的*意图*：例如点击、双指缩放或滚动。

Avalonia 提供两类手势：

**内置手势事件** 用于覆盖最常见的交互：

| 事件 | 说明 |
|---|---|
| `Tapped` | 指针在控件上按下并释放。 |
| `DoubleTapped` | 在平台允许的双击时间和距离阈值内，两次点击发生在同一位置。 |
| `Holding` | 指针按下后保持不动一段时间。必须为每个控件显式启用 `InputElement.IsHoldingEnabled`。 |

**手势识别器** 用于检测更复杂的多指或方向性模式。你可以把它们附加到控件的 `GestureRecognizers` 集合中，它们会监听控件的指针事件来识别特定模式：

| 识别器 | 说明 |
|---|---|
| [`PinchGestureRecognizer`](/docs/input-interaction/gestures/pinch-gesture-recognizer) | 两个指针彼此靠近或远离移动，用于双指缩放。 |
| [`PullGestureRecognizer`](/docs/input-interaction/gestures/pull-gesture-recognizer) | 指针从控件边缘沿特定方向拖动，用于下拉刷新。 |
| [`ScrollGestureRecognizer`](/docs/input-interaction/gestures/scroll-gesture-recognizer) | 指针拖动以水平、垂直或双向滚动内容。 |
| [`SwipeGestureRecognizer`](/docs/input-interaction/gestures/swipe-gesture-recognizer) | 快速的方向性指针拖动，用于离散翻页等交互，并提供速度数据以支持速度敏感的过渡。 |

## 附加手势识别器

手势识别器可以在 XAML 或 code-behind 中添加到控件上：

```xml
<Image Stretch="UniformToFill" Name="image" Source="/image.jpg">
  <Image.GestureRecognizers>
    <PinchGestureRecognizer />
  </Image.GestureRecognizers>
</Image>
```

```csharp title='C#'
image.GestureRecognizers.Add(new PinchGestureRecognizer());
```

附加完成后，识别器会监控该控件的指针事件，并在检测到匹配模式时触发对应的手势事件。每个识别器通常都会触发一个开始事件（例如 `InputElement.PinchEvent`）和一个结束事件（例如 `InputElement.PinchEndedEvent`）。

## 订阅手势事件

手势识别器触发的事件属于路由事件。请使用 `AddHandler` 进行订阅：

```csharp title='C#'
image.AddHandler(InputElement.PinchEvent, (sender, args) =>
{
    var scale = args.Scale;
    // 处理缩放手势
});
```

如果你的处理器已经完整处理了该手势，请将其标记为已处理，以阻止它继续冒泡：

```csharp title='C#'
args.Handled = true;
```

## Holding 手势

与 `Tapped` 和 `DoubleTapped` 不同，`Holding` 手势必须通过设置 `InputElement.IsHoldingEnabled` 附加属性为每个控件单独启用：

```xml
<Border InputElement.IsHoldingEnabled="True" Holding="OnHolding" />
```

按住时长由 `TopLevel` 上的 `PlatformSettings.HoldWaitDuration` 定义。当达到这个时长后，会触发一次 `Holding` 事件，且 `HoldingState` 为 `Started`。当指针释放时，它会再次触发，此时状态为 `Completed`。如果在按住期间开始了新的手势，或按下了第二个指针，则会触发状态为 `Canceled` 的 `Holding`。

如果你希望鼠标指针也能触发 `Holding`（而不仅仅是触摸），请设置 `InputElement.IsHoldWithMouseEnabled`：

```xml
<Border InputElement.IsHoldingEnabled="True"
        InputElement.IsHoldWithMouseEnabled="True"
        Holding="OnHolding" />
```

## 组合多个手势识别器

你可以为同一个控件附加多个手势识别器。例如，为了同时支持图像的双指缩放和拖动平移：

```xml
<Image Name="image" Source="/image.jpg">
  <Image.GestureRecognizers>
    <PinchGestureRecognizer />
    <ScrollGestureRecognizer CanHorizontallyScroll="True"
                              CanVerticallyScroll="True" />
  </Image.GestureRecognizers>
</Image>
```

当附加了多个识别器时，它们会各自独立地监控控件的指针事件。但同一时刻只能有一个识别器处于活动状态：当某个识别器捕获到一个手势后，它会阻止其他识别器激活，直到该手势结束。

## 指针类型过滤

内置手势识别器会处理所有指针类型（鼠标、触摸和笔）。在某些需要为不同输入设备分配不同行为的应用中，这可能会成为问题。例如，绘图应用可能希望用笔来绘制，而用触摸来进行平移和缩放。

如果需要区分输入设备，请检查指针事件参数中的 `PointerType`：

```csharp title='C#'
private void OnPointerPressed(object? sender, PointerPressedEventArgs e)
{
    var pointerType = e.Pointer.Type;

    if (pointerType == PointerType.Pen)
    {
        // 处理绘图
    }
    else if (pointerType == PointerType.Touch)
    {
        // 处理平移/缩放导航
    }
}
```

由于内置识别器不会根据指针类型进行过滤，因此凡是需要针对不同设备执行不同手势处理的场景（例如保留触摸用于平移/缩放，而用笔进行绘图），都需要自定义手势识别器。

## 自定义手势识别器

若要创建自定义手势识别器，可以继承 `GestureRecognizer` 并重写其指针跟踪方法。这样你就能完全控制：要捕获哪些指针事件、如何识别手势，以及最终触发哪些事件。

```csharp title='C#'
public class TouchOnlyPinchRecognizer : GestureRecognizer
{
    protected override void PointerPressed(PointerPressedEventArgs e)
    {
        if (e.Pointer.Type != PointerType.Touch)
            return;

        // 跟踪触摸接触点以检测双指缩放
    }

    protected override void PointerMoved(PointerEventArgs e)
    {
        // 根据已跟踪的接触点计算缩放比例
    }

    protected override void PointerReleased(PointerReleasedEventArgs e)
    {
        // 结束手势跟踪
    }
}
```

自定义识别器的附加方式与内置识别器完全相同：

```xml
<Image Name="image" Source="/image.jpg">
  <Image.GestureRecognizers>
    <local:TouchOnlyPinchRecognizer />
  </Image.GestureRecognizers>
</Image>
```

如需参考实现，请查看 GitHub 上的[内置手势识别器源码](https://github.com/AvaloniaUI/Avalonia/tree/master/src/Avalonia.Base/Input/GestureRecognizers)。

## 另请参阅

- [指针事件](/docs/input-interaction/pointer)：手势构建所依赖的底层指针事件。
- [路由事件](/docs/input-interaction/routed-events)：事件如何在元素树中传播。
