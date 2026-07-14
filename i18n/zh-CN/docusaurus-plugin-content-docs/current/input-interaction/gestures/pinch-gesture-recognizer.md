---
id: pinch-gesture-recognizer
title: Pinch
description: 使用 PinchGestureRecognizer 跟踪双指缩放手势。
doc-type: reference
---

这是一个用于跟踪双指缩放手势的手势识别器。当两个指针接触点彼此靠近或远离时，就会产生 pinch 手势。它非常适合用于实现双指缩放交互的控件。

<div style={{textAlign: 'center', margin: '24px 0'}}>
<svg width="240" height="190" viewBox="0 0 240 190" fill="none" xmlns="http://www.w3.org/2000/svg">
  {/* 触摸板表面 */}
  <rect x="20" y="10" width="200" height="140" rx="14"
    fill="currentColor" fillOpacity="0.03"
    stroke="currentColor" strokeWidth="1.5" strokeOpacity="0.12"/>
  <rect x="24" y="14" width="192" height="132" rx="11"
    fill="none"
    stroke="currentColor" strokeWidth="0.5" strokeOpacity="0.06"/>
  {/* 左侧触点 */}
  <circle cy="80" r="14"
    fill="currentColor" fillOpacity="0.08"
    stroke="currentColor" strokeWidth="1.5" strokeOpacity="0.35">
    <animate attributeName="cx"
      values="108;108;62;62;108;108"
      keyTimes="0;0.06;0.44;0.56;0.94;1"
      calcMode="spline"
      keySplines="0 0 1 1;0.25 0.1 0.25 1;0 0 1 1;0.25 0.1 0.25 1;0 0 1 1"
      dur="5s" repeatCount="indefinite"/>
  </circle>
  {/* 右侧触点 */}
  <circle cy="80" r="14"
    fill="currentColor" fillOpacity="0.08"
    stroke="currentColor" strokeWidth="1.5" strokeOpacity="0.35">
    <animate attributeName="cx"
      values="132;132;178;178;132;132"
      keyTimes="0;0.06;0.44;0.56;0.94;1"
      calcMode="spline"
      keySplines="0 0 1 1;0.25 0.1 0.25 1;0 0 1 1;0.25 0.1 0.25 1;0 0 1 1"
      dur="5s" repeatCount="indefinite"/>
  </circle>
  {/* 标签：放大 */}
  <text x="120" y="176" textAnchor="middle"
    fill="currentColor" fontSize="13" fontFamily="system-ui, sans-serif">
    <animate attributeName="opacity"
      values="0.5;0.5;0;0;0.5"
      keyTimes="0;0.46;0.5;0.96;1"
      dur="5s" repeatCount="indefinite"/>
    放大
  </text>
  {/* 标签：缩小 */}
  <text x="120" y="176" textAnchor="middle"
    fill="currentColor" fontSize="13" fontFamily="system-ui, sans-serif">
    <animate attributeName="opacity"
      values="0;0;0.5;0.5;0"
      keyTimes="0;0.46;0.5;0.96;1"
      dur="5s" repeatCount="indefinite"/>
    缩小
  </text>
</svg>
</div>

## 使用 PinchGestureRecognizer
可以通过控件的 `GestureRecognizers` 属性将 PinchGestureRecognizer 附加到控件上。
```xml
<Image Stretch="UniformToFill"
        Margin="5"
        Name="image"
        Source="/image.jpg">
  <Image.GestureRecognizers>
    <PinchGestureRecognizer/>
  </Image.GestureRecognizers>
</Image>
```

```csharp title='C#'
image.GestureRecognizers.Add(new PinchGestureRecognizer());
```

当 PinchGestureRecognizer 检测到 pinch 手势开始时，会触发 `InputElement.PinchEvent`。当 pinch 结束时，例如指针释放或另一个手势开始时，它会触发 `InputElement.PinchEndedEvent`。
传递给 `InputElement.PinchEvent` 事件处理器的参数中，`Scale` 属性表示自手势开始以来的相对缩放比例。

## 绑定事件
将 PinchGestureRecognizer 添加到控件后，你需要在 code-behind 中绑定这些事件，可以使用内联处理器，也可以绑定到事件函数：
```csharp title='C#'
image.AddHandler(InputElement.PinchEvent, (s, e) => { });
image.AddHandler(InputElement.PinchEndedEvent, (s, e) => { });
```
```csharp title='C#'
image.AddHandler(InputElement.PinchEvent, Image_PinchGesture);
image.AddHandler(InputElement.PinchEndedEvent, Image_PinchGestureEnded);
...
private void Image_PinchGesture(object? sender, PinchGestureEventArgs e) { }
private void Image_PinchGestureEnded(object? sender, PinchGestureEndedEventArgs e) { }
```
如果你的处理器已经完整处理了该手势，可以通过下面的方式将事件标记为已处理：
```csharp title='C#'
e.Handled = true;
```

## 指针类型过滤

`PinchGestureRecognizer` 会响应所有指针类型（鼠标、触摸和笔）。如果你的应用只希望触摸输入触发双指缩放（例如把笔输入保留给绘图），那么你需要创建一个按 `PointerType.Touch` 过滤的[自定义手势识别器](/docs/input-interaction/gestures#custom-gesture-recognizers)。

## 更多信息

:::info
在 _GitHub_ 上查看源代码

[`PinchGestureRecognizer.cs`](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Base/Input/GestureRecognizers/PinchGestureRecognizer.cs)

[`PinchEventArgs.cs`](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Base/Input/PinchEventArgs.cs)
:::

## 另请参阅

- [手势](/docs/input-interaction/gestures)：手势识别器与内置手势事件概览。
- [Scroll Gesture Recognizer](/docs/input-interaction/gestures/scroll-gesture-recognizer)：用于平移内容的滚动手势。
- [Pull Gesture Recognizer](/docs/input-interaction/gestures/pull-gesture-recognizer)：用于下拉刷新的拉动手势。
