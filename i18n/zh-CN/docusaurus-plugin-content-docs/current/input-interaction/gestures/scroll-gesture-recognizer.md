---
id: scroll-gesture-recognizer
title: Scroll
---

这是一个用于跟踪滚动手势并实现内容平移的手势识别器。`ScrollGestureRecognizer` 会检测指针在控件边界内的拖动，并将该移动转换为滚动增量，同时支持惯性效果（即指针释放后内容仍会继续滚动并逐渐减速）。它支持水平滚动、垂直滚动，或同时支持两者。

当控件需要在一个或多个方向上自由平移其内容时，请使用 `ScrollGestureRecognizer`。如果你需要的是从边缘开始、方向明确的单向拖动（例如下拉刷新），则应改用 [`PullGestureRecognizer`](/docs/input-interaction/gestures/pull-gesture-recognizer)。

<div style={{textAlign: 'center', margin: '24px 0'}}>
<svg width="240" height="190" viewBox="0 0 240 190" fill="none" xmlns="http://www.w3.org/2000/svg">
  {/* Trackpad surface */}
  <rect x="20" y="10" width="200" height="140" rx="14"
    fill="currentColor" fillOpacity="0.03"
    stroke="currentColor" strokeWidth="1.5" strokeOpacity="0.12"/>
  <rect x="24" y="14" width="192" height="132" rx="11"
    fill="none"
    stroke="currentColor" strokeWidth="0.5" strokeOpacity="0.06"/>
  {/* Touch point 1: Vertical scroll (active 0-45%) */}
  <circle cx="120" r="14"
    fill="currentColor" fillOpacity="0.08"
    stroke="currentColor" strokeWidth="1.5" strokeOpacity="0.35">
    <animate attributeName="cy"
      values="40;40;120;120;40;40"
      keyTimes="0;0.04;0.40;0.44;0.48;1"
      calcMode="spline"
      keySplines="0 0 1 1;0.25 0.1 0.25 1;0 0 1 1;0 0 1 1;0 0 1 1"
      dur="6s" repeatCount="indefinite"/>
    <animate attributeName="opacity"
      values="0;1;1;0;0;0"
      keyTimes="0;0.03;0.40;0.45;0.48;1"
      dur="6s" repeatCount="indefinite"/>
  </circle>
  {/* Touch point 2: Horizontal scroll (active 50-95%) */}
  <circle cy="80" r="14"
    fill="currentColor" fillOpacity="0.08"
    stroke="currentColor" strokeWidth="1.5" strokeOpacity="0.35">
    <animate attributeName="cx"
      values="50;50;50;190;190;50;50"
      keyTimes="0;0.50;0.54;0.90;0.94;0.98;1"
      calcMode="spline"
      keySplines="0 0 1 1;0 0 1 1;0.25 0.1 0.25 1;0 0 1 1;0 0 1 1;0 0 1 1"
      dur="6s" repeatCount="indefinite"/>
    <animate attributeName="opacity"
      values="0;0;0;1;1;0;0"
      keyTimes="0;0.50;0.52;0.54;0.90;0.95;1"
      dur="6s" repeatCount="indefinite"/>
  </circle>
  {/* Labels */}
  <text x="120" y="176" textAnchor="middle"
    fill="currentColor" fontSize="13" fontFamily="system-ui, sans-serif">
    <animate attributeName="opacity"
      values="0.5;0.5;0;0;0.5"
      keyTimes="0;0.44;0.48;0.96;1"
      dur="6s" repeatCount="indefinite"/>
    Vertical
  </text>
  <text x="120" y="176" textAnchor="middle"
    fill="currentColor" fontSize="13" fontFamily="system-ui, sans-serif">
    <animate attributeName="opacity"
      values="0;0;0.5;0.5;0"
      keyTimes="0;0.48;0.52;0.94;1"
      dur="6s" repeatCount="indefinite"/>
    Horizontal
  </text>
</svg>
</div>

## 使用 ScrollGestureRecognizer
可以通过控件的 `GestureRecognizers` 属性将 ScrollGestureRecognizer 附加到控件上。
```xml
<Image Stretch="UniformToFill"
        Margin="5"
        Name="image"
        Source="/image.jpg">
  <Image.GestureRecognizers>
    <ScrollGestureRecognizer CanHorizontallyScroll="True"
                              CanVerticallyScroll="True"/>
  </Image.GestureRecognizers>
</Image>
```

```csharp title='C#'
image.GestureRecognizers.Add(new ScrollGestureRecognizer()
{
    CanVerticallyScroll = true,
    CanHorizontallyScroll = true,
});
```

当 ScrollGestureRecognizer 检测到滚动手势开始时，会触发 `InputElement.ScrollGestureEvent`。当滚动结束时，例如指针释放或另一个手势开始时，它会触发 `InputElement.ScrollGestureEndedEvent`。

## 绑定事件
将 ScrollGestureRecognizer 添加到控件后，你需要在 code-behind 中绑定这些事件，可以使用内联处理器，也可以绑定到事件函数：
```csharp title='C#'
image.AddHandler(InputElement.ScrollGestureEvent, (s, e) => { });
image.AddHandler(InputElement.ScrollGestureEndedEvent, (s, e) => { });
```
```csharp title='C#'
image.AddHandler(InputElement.ScrollGestureEvent, Image_ScrollGesture);
image.AddHandler(InputElement.ScrollGestureEndedEvent, Image_ScrollGestureEnded);
...
private void Image_ScrollGesture(object? sender, ScrollGestureEventArgs e) { }
private void Image_ScrollGestureEnded(object? sender, ScrollGestureEndedEventArgs e) { }
```
如果你的处理器已经完整处理了该手势，可以通过下面的方式将事件标记为已处理：
```csharp title='C#'
e.Handled = true;
```
## 常用属性

你最常使用的通常会是下面这些属性：

<table>
    <thead>
      <tr>
        <th width="266">Property</th>
        <th>说明</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>CanVerticallyScroll</td>
        <td>定义内容是否可以垂直滚动。</td>
      </tr>
      <tr>
        <td>CanHorizontallyScroll</td>
        <td>定义内容是否可以水平滚动。</td>
      </tr>
    </tbody>
  </table>


## 更多信息

:::info
关于该手势识别器的完整 API 文档，请参阅 [ScrollGestureRecognizer API 参考](/api/avalonia/input/gesturerecognizers/scrollgesturerecognizer)。
:::

:::info
在 _GitHub_ 上查看源代码 [`ScrollGestureRecognizer.cs`](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Base/Input/GestureRecognizers/ScrollGestureRecognizer.cs)
:::

## 另请参阅

- [手势](/docs/input-interaction/gestures)：手势识别器与内置手势事件概览。
- [Pull Gesture Recognizer](/docs/input-interaction/gestures/pull-gesture-recognizer)：用于下拉刷新的拉动手势。
- [Pinch Gesture Recognizer](/docs/input-interaction/gestures/pinch-gesture-recognizer)：用于缩放交互的 pinch 手势。
