---
id: swipe-gesture-recognizer
title: Swipe
doc-type: reference
description: Swipe 手势识别器可检测用户沿单一方向快速拖动指针的行为，并提供逐事件速度数据以支持过渡效果。
---

这是一个用于跟踪 swipe 手势的识别器，适合离散式翻页交互。`SwipeGestureRecognizer` 会检测用户是否沿单一方向快速拖动指针，并提供逐事件的速度数据，从而支持对速度敏感的过渡效果，例如轮播页切换。与 `ScrollGestureRecognizer` 不同，它不包含惯性或连续滚动物理效果。

当控件需要对明确的方向性甩动做出响应时（例如在轮播图中切换页面），请使用 `SwipeGestureRecognizer`。如果你需要的是带惯性的连续平移，请改用 [`ScrollGestureRecognizer`](/docs/input-interaction/gestures/scroll-gesture-recognizer)。

## 使用 SwipeGestureRecognizer

可以通过控件的 `GestureRecognizers` 属性将 `SwipeGestureRecognizer` 附加到控件上。
```xml
<Border Name="swipeArea" Background="Transparent" Height="300">
    <Border.GestureRecognizers>
        <SwipeGestureRecognizer CanHorizontallySwipe="True" />
    </Border.GestureRecognizers>
    <TextBlock Text="向左或向右滑动"
               HorizontalAlignment="Center"
               VerticalAlignment="Center" />
</Border>
```

```csharp title='C#'
swipeArea.GestureRecognizers.Add(new SwipeGestureRecognizer
{
    CanHorizontallySwipe = true,
    CanVerticallySwipe = false
});
```

在 swipe 过程中，`SwipeGestureRecognizer` 会随着指针移动持续触发 `InputElement.SwipeGestureEvent`。当 swipe 结束时，例如指针释放或另一个手势开始时，它会触发 `InputElement.SwipeGestureEndedEvent`。

## 绑定事件

将 `SwipeGestureRecognizer` 添加到控件后，你需要在 code-behind 中绑定这些事件，可以使用内联处理器，也可以绑定到事件函数：

```csharp title='C#'
swipeArea.AddHandler(InputElement.SwipeGestureEvent, (s, e) => { });
swipeArea.AddHandler(InputElement.SwipeGestureEndedEvent, (s, e) => { });
```

```csharp title='C#'
swipeArea.AddHandler(InputElement.SwipeGestureEvent, OnSwipeGesture);
swipeArea.AddHandler(InputElement.SwipeGestureEndedEvent, OnSwipeGestureEnded);
...
private void OnSwipeGesture(object? sender, SwipeGestureEventArgs e) { }
private void OnSwipeGestureEnded(object? sender, SwipeGestureEndedEventArgs e) { }
```

如果你的处理器已经完整处理了该手势，可以通过下面的方式将事件标记为已处理：

```csharp title='C#'
e.Handled = true;
```

## 事件参数

`SwipeGestureEventArgs` 会在手势进行过程中触发：

| 属性 | 类型 | 说明 |
|---|---|---|
| `Id` | `int` | 当前手势序列的唯一标识符。 |
| `Delta` | `Vector` | 自上一次事件以来的像素位移增量。 |
| `Velocity` | `Vector` | 当前 swipe 速度，单位为像素/秒。 |
| `SwipeDirection` | `SwipeDirection` | 主导滑动方向：`Left`、`Right`、`Up` 或 `Down`。 |

`SwipeGestureEndedEventArgs` 会在指针释放时触发：

| 属性 | 类型 | 说明 |
|---|---|---|
| `Id` | `int` | 当前手势序列的唯一标识符。 |
| `Velocity` | `Vector` | 指针释放瞬间的 swipe 速度。 |

## 属性

你最常使用的通常会是下面这些属性：

| 属性 | 类型 | 说明 | 默认值 |
|---|---|---|---|
| `CanHorizontallySwipe` | `bool` | 启用水平（左/右）swipe 跟踪。 | `false` |
| `CanVerticallySwipe` | `bool` | 启用垂直（上/下）swipe 跟踪。 | `false` |
| `Threshold` | `double` | 在识别 swipe 之前，指针至少需要移动的像素距离。若设为 0，则使用平台默认阈值。 | 0 |
| `IsMouseEnabled` | `bool` | 为 `true` 时，除了触摸和笔输入外，鼠标指针事件也会触发 swipe 手势。 | `false` |
| `IsEnabled` | `bool` | 完全启用或禁用该识别器。 | `true` |

## 示例

### 检测水平滑动以切换页面

```csharp title='C#'
int currentPage = 0;
int totalPages = 5;

swipeArea.AddHandler(InputElement.SwipeGestureEndedEvent, (s, e) =>
{
    if (Math.Abs(e.Velocity.X) > 200) // 甩动速度足够快
    {
        if (e.Velocity.X < 0 && currentPage < totalPages - 1)
            currentPage++;
        else if (e.Velocity.X > 0 && currentPage > 0)
            currentPage--;
    }
});
```

### 垂直滑动检测

```xml
<Border Name="verticalSwipeArea" Background="Transparent">
    <Border.GestureRecognizers>
        <SwipeGestureRecognizer CanVerticallySwipe="True" />
    </Border.GestureRecognizers>
</Border>
```

### 启用鼠标支持

默认情况下，只有触摸和笔输入会触发 swipe 手势。若要在桌面场景中支持鼠标，请启用鼠标支持：

```xml
<Border.GestureRecognizers>
    <SwipeGestureRecognizer CanHorizontallySwipe="True"
                            IsMouseEnabled="True" />
</Border.GestureRecognizers>
```

## 另请参阅

- [API 参考](/api/avalonia/input/gesturerecognizers/swipegesturerecognizer)
- [源代码](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Base/Input/GestureRecognizers/SwipeGestureRecognizer.cs)
- [手势](/docs/input-interaction/gestures)：手势识别器与内置手势事件概览。
- [Scroll](/docs/input-interaction/gestures/scroll-gesture-recognizer)：用于带惯性连续平移的滚动手势。
- [Pull](/docs/input-interaction/gestures/pull-gesture-recognizer)：用于下拉刷新的拉动手势。
- [Pinch](/docs/input-interaction/gestures/pinch-gesture-recognizer)：用于缩放交互的 pinch 手势。
