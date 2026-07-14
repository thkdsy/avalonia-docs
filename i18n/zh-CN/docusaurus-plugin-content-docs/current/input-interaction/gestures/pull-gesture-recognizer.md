---
id: pull-gesture-recognizer
title: Pull
---

这是一个用于跟踪拉动手势的手势识别器。当指针从控件边缘沿 `PullDirection` 属性指定的单一方向拖动时，就会产生 pull 手势。最典型的使用场景是下拉刷新：用户从列表顶部向下拖动，以触发数据重新加载。

与 [`ScrollGestureRecognizer`](/docs/input-interaction/gestures/scroll-gesture-recognizer) 不同，`PullGestureRecognizer` 设计目标是明确的单方向交互，而不是自由平移。它需要更大的初始拖动距离才会激活，只识别一个已配置方向上的移动，也不会应用惯性。这些特性使它特别适合那些必须在用户意图足够明确后才触发的操作。

<div style={{textAlign: 'center', margin: '24px 0'}}>
<svg width="240" height="190" viewBox="0 0 240 190" fill="none" xmlns="http://www.w3.org/2000/svg">
  {/* Trackpad surface */}
  <rect x="20" y="10" width="200" height="140" rx="14"
    fill="currentColor" fillOpacity="0.03"
    stroke="currentColor" strokeWidth="1.5" strokeOpacity="0.12"/>
  <rect x="24" y="14" width="192" height="132" rx="11"
    fill="none"
    stroke="currentColor" strokeWidth="0.5" strokeOpacity="0.06"/>
  {/* Touch point 1: Top to bottom (active 0-22%) */}
  <circle cx="120" r="14"
    fill="currentColor" fillOpacity="0.08"
    stroke="currentColor" strokeWidth="1.5" strokeOpacity="0.35">
    <animate attributeName="cy"
      values="30;30;115;115;30;30"
      keyTimes="0;0.03;0.20;0.22;0.24;1"
      calcMode="spline"
      keySplines="0 0 1 1;0.25 0.1 0.25 1;0 0 1 1;0 0 1 1;0 0 1 1"
      dur="10s" repeatCount="indefinite"/>
    <animate attributeName="opacity"
      values="0;1;1;0;0;0"
      keyTimes="0;0.02;0.20;0.23;0.25;1"
      dur="10s" repeatCount="indefinite"/>
  </circle>
  {/* Touch point 2: Bottom to top (active 25-47%) */}
  <circle cx="120" r="14"
    fill="currentColor" fillOpacity="0.08"
    stroke="currentColor" strokeWidth="1.5" strokeOpacity="0.35">
    <animate attributeName="cy"
      values="130;130;130;40;40;130;130"
      keyTimes="0;0.25;0.28;0.45;0.47;0.49;1"
      calcMode="spline"
      keySplines="0 0 1 1;0 0 1 1;0.25 0.1 0.25 1;0 0 1 1;0 0 1 1;0 0 1 1"
      dur="10s" repeatCount="indefinite"/>
    <animate attributeName="opacity"
      values="0;0;0;1;1;0;0"
      keyTimes="0;0.25;0.27;0.28;0.45;0.48;1"
      dur="10s" repeatCount="indefinite"/>
  </circle>
  {/* Touch point 3: Left to right (active 50-72%) */}
  <circle cy="80" r="14"
    fill="currentColor" fillOpacity="0.08"
    stroke="currentColor" strokeWidth="1.5" strokeOpacity="0.35">
    <animate attributeName="cx"
      values="40;40;40;200;200;40;40"
      keyTimes="0;0.50;0.53;0.70;0.72;0.74;1"
      calcMode="spline"
      keySplines="0 0 1 1;0 0 1 1;0.25 0.1 0.25 1;0 0 1 1;0 0 1 1;0 0 1 1"
      dur="10s" repeatCount="indefinite"/>
    <animate attributeName="opacity"
      values="0;0;0;1;1;0;0"
      keyTimes="0;0.50;0.52;0.53;0.70;0.73;1"
      dur="10s" repeatCount="indefinite"/>
  </circle>
  {/* Touch point 4: Right to left (active 75-97%) */}
  <circle cy="80" r="14"
    fill="currentColor" fillOpacity="0.08"
    stroke="currentColor" strokeWidth="1.5" strokeOpacity="0.35">
    <animate attributeName="cx"
      values="200;200;200;40;40;200;200"
      keyTimes="0;0.75;0.78;0.95;0.97;0.99;1"
      calcMode="spline"
      keySplines="0 0 1 1;0 0 1 1;0.25 0.1 0.25 1;0 0 1 1;0 0 1 1;0 0 1 1"
      dur="10s" repeatCount="indefinite"/>
    <animate attributeName="opacity"
      values="0;0;0;1;1;0;0"
      keyTimes="0;0.75;0.77;0.78;0.95;0.98;1"
      dur="10s" repeatCount="indefinite"/>
  </circle>
  {/* Labels */}
  <text x="120" y="176" textAnchor="middle"
    fill="currentColor" fontSize="13" fontFamily="system-ui, sans-serif">
    <animate attributeName="opacity"
      values="0;0.5;0.5;0;0;0;0;0;0"
      keyTimes="0;0.02;0.20;0.23;0.25;0.50;0.75;0.98;1"
      dur="10s" repeatCount="indefinite"/>
    Top to bottom
  </text>
  <text x="120" y="176" textAnchor="middle"
    fill="currentColor" fontSize="13" fontFamily="system-ui, sans-serif">
    <animate attributeName="opacity"
      values="0;0;0;0.5;0.5;0;0;0;0"
      keyTimes="0;0.23;0.27;0.28;0.45;0.48;0.50;0.98;1"
      dur="10s" repeatCount="indefinite"/>
    Bottom to top
  </text>
  <text x="120" y="176" textAnchor="middle"
    fill="currentColor" fontSize="13" fontFamily="system-ui, sans-serif">
    <animate attributeName="opacity"
      values="0;0;0;0.5;0.5;0;0"
      keyTimes="0;0.48;0.52;0.53;0.70;0.73;1"
      dur="10s" repeatCount="indefinite"/>
    Left to right
  </text>
  <text x="120" y="176" textAnchor="middle"
    fill="currentColor" fontSize="13" fontFamily="system-ui, sans-serif">
    <animate attributeName="opacity"
      values="0;0;0;0.5;0.5;0;0"
      keyTimes="0;0.73;0.77;0.78;0.95;0.98;1"
      dur="10s" repeatCount="indefinite"/>
    Right to left
  </text>
</svg>
</div>

## 使用 PullGestureRecognizer
可以通过控件的 `GestureRecognizers` 属性将 PullGestureRecognizer 附加到控件上。
```xml
<Border Width="500"
        Height="500"
        Margin="5"
        Name="border">
  <Border.GestureRecognizers>
    <PullGestureRecognizer PullDirection="TopToBottom"/>
  </Border.GestureRecognizers>
</Border>
```

```csharp title='C#'
border.GestureRecognizers.Add(new PullGestureRecognizer()
            {
                PullDirection = PullDirection.TopToBottom,
            });
```

当指针沿配置方向移动时，`PullGestureRecognizer` 会持续触发 `InputElement.PullGestureEvent`。当 pull 结束时（例如指针释放或另一个手势开始），它会触发 `InputElement.PullGestureEndedEvent`。

监听 pull 手势的控件通常应在 `PullGestureEndedEvent` 触发时重置其视觉状态，除非拉动距离已经超过触发目标操作的阈值。例如，如果用户在尚未拉到足够距离前就释放，下拉刷新指示器应当回弹到原位。

### PullDirection
该属性定义 pull 的方向，可用值共有 4 个：
* `PullDirection.TopToBottom`：从上边缘开始，向下拖动
* `PullDirection.BottomToTop`：从下边缘开始，向上拖动
* `PullDirection.LeftToRight`：从左边缘开始，向右拖动
* `PullDirection.RightToLeft`：从右边缘开始，向左拖动

## 绑定事件
将 PullGestureRecognizer 添加到控件后，你需要在 code-behind 中绑定这些事件，可以使用内联处理器，也可以绑定到事件函数：
```csharp title='C#'
image.AddHandler(InputElement.PullGestureEvent, (s, e) => { });
image.AddHandler(InputElement.PullGestureEndedEvent, (s, e) => { });
```
```csharp title='C#'
image.AddHandler(InputElement.PullGestureEvent, Image_PullGesture);
image.AddHandler(InputElement.PullGestureEndedEvent, Image_PullGestureEnded);
...
private void Image_PullGesture(object? sender, PullGestureEventArgs e) { }
private void Image_PullGestureEnded(object? sender, PullGestureEndedEventArgs e) { }
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
        <td>PullDirection</td>
        <td>定义拉动手势的方向。</td>
      </tr>
    </tbody>
  </table>


## 更多信息

:::info
在 _GitHub_ 上查看源代码

[`PullGestureRecognizer.cs`](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Base/Input/GestureRecognizers/PullGestureRecognizer.cs)

[`PullGestureEventArgs.cs`](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Base/Input/PullGestureEventArgs.cs)
:::

## 另请参阅

- [手势](/docs/input-interaction/gestures)：手势识别器与内置手势事件概览。
- [Scroll Gesture Recognizer](/docs/input-interaction/gestures/scroll-gesture-recognizer)：用于平移内容的滚动手势。
- [Pinch Gesture Recognizer](/docs/input-interaction/gestures/pinch-gesture-recognizer)：用于缩放交互的 pinch 手势。
