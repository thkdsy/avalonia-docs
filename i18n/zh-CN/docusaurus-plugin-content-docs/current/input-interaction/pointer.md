---
id: pointer
title: 指针设备
---

import PointerPressedSampleScreenshot from '/img/concepts/ui-concepts/user-input/pointer-pressed.gif';

Avalonia 基于一种称为“指针设备”的抽象来处理输入。这种抽象可以表示多种设备，包括但不限于鼠标、触摸板和触控笔。

控件最常见的职责之一，就是检测并响应用户输入。Avalonia 输入系统同时使用[直接事件和路由事件](/docs/input-interaction/routed-events)来支持文本输入、焦点管理和鼠标定位。

应用通常会有复杂的输入需求。Avalonia 提供了[命令系统](/docs/input-interaction/adding-interactivity)，可以将用户输入动作与响应该动作的代码分离开来。

实现了 `ICommandSource` 的控件具有 `HotKey` 属性。此外，控件还提供了一些事件，允许你订阅指针移动、点击和滚轮操作：

| 事件 | 说明 |
|---|---|
| `PointerEntered` | 当指针移动到控件边界内时触发。 |
| `PointerExited` | 当指针离开控件边界时触发。 |
| `PointerMoved` | 当指针在控件边界内移动时触发。 |
| `PointerPressed` | 当指针按钮在控件上方按下时触发。 |
| `PointerReleased` | 当此前按下的指针按钮在控件上方释放时触发。 |
| `PointerWheelChanged` | 当在控件上使用鼠标滚轮或触摸板滚动时触发。 |
| `Tapped` | 当指针在控件上按下并释放后触发。 |
| `DoubleTapped` | 当两次点击发生在同一位置且间隔在平台双击阈值内时触发。 |
| `Holding` | 当指针被按住并持续达到 `PlatformSettings.HoldWaitDuration` 定义的时长时触发。 |

例如，你可以像下面这样订阅某个控件上的指针按下事件：

```csharp title='C#'
private void PointerPressedHandler (object sender, PointerPressedEventArgs args)
{
    var point = args.GetCurrentPoint(sender as Control);
    var x = point.Position.X;
    var y = point.Position.Y;
    var msg = $"相对于发送者，指针按下位置为 {x}, {y}。";
    if (point.Properties.IsLeftButtonPressed)
    {
        msg += " 左键已按下。";
    }
    if (point.Properties.IsRightButtonPressed)
    {
        msg += " 右键已按下。";
    }
    results.Text = msg ;
}
```

```xml title='XAML'
<StackPanel Margin="20" Background="AliceBlue" 
            PointerPressed="PointerPressedHandler" >
  <TextBlock x:Name="results" Margin="5">就绪...</TextBlock>
</StackPanel>
```

<Image light={PointerPressedSampleScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

## 指针类型

Avalonia 通过 `PointerPoint.Pointer.Type` 属性区分不同的输入设备类型：

| 类型 | 说明 |
|---|---|
| `Mouse` | 标准鼠标或触摸板输入。 |
| `Touch` | 触摸屏输入。 |
| `Pen` | 触控笔或手写笔输入（如数位板、主动笔）。 |

### 触控笔与手写笔属性

当指针类型为 `Pen` 时，`PointerPointProperties` 上会提供额外属性：

| 属性 | 类型 | 说明 |
|---|---|---|
| `Pressure` | `float` | 压力值，范围从 0（无压力）到 1（最大压力）。 |
| `XTilt` | `float` | 触控笔沿 X 轴的倾斜角度。 |
| `YTilt` | `float` | 触控笔沿 Y 轴的倾斜角度。 |
| `Twist` | `float` | 触控笔绕自身轴顺时针旋转的角度。 |
| `IsEraser` | `bool` | 当橡皮擦笔尖生效时为 `true`。 |
| `IsBarrelButtonPressed` | `bool` | 当笔杆按钮被按住时为 `true`。 |

```csharp
private void OnPointerMoved(object? sender, PointerEventArgs e)
{
    var point = e.GetCurrentPoint(this);

    if (point.Pointer.Type == PointerType.Pen)
    {
        var pressure = point.Properties.Pressure;
        var isEraser = point.Properties.IsEraser;
        // 根据压力和橡皮状态调整画笔大小或工具
    }
}
```

所有支持笔输入的平台都会提供这些属性（Windows、macOS，以及基于 X11 的 Linux）。

## 指针位置

在上面的示例中，指针坐标（`x` 和 `y`）是相对于发送者控件原点（左上角）计算的，这里就是相对于 `StackPanel`。如果你希望得到相对于所属窗口的坐标，可以像下面这样使用 `GetCurrentPoint` 方法：

```csharp
var point = args.GetCurrentPoint(this);
```

## 点击手势事件

控件还提供了特殊的手势事件：`Tapped`、`DoubleTapped` 和 `Holding`。`Tapped` 会在指针在控件上按下再释放后触发；`DoubleTapped` 会在同一位置连续两次按下后触发。

`Holding` 会在指针按住持续一段设定时间后触发。这个时长由 `TopLevel` 的 `PlatformSettings` 中的 `HoldWaitDuration` 属性定义。可以通过设置控件上的 `InputElement.IsHoldingEnabled` 附加属性来启用 `Holding`。当按住时间达到阈值时，控件的 `HoldingEvent` 会被触发，并将参数中的 `HoldingState` 设为 `HoldingState.Started`。当指针释放时，该事件会再次触发，此时状态为 `HoldingState.Completed`。如果在 `Holding` 已开始后又发起了新的手势，或按下了第二个指针，则 `Holding` 手势会被取消，并触发 `HoldingEvent`，状态为 `HoldingState.Canceled`。你也可以通过设置控件上的 `InputElement.IsHoldWithMouseEnabled` 附加属性，允许鼠标触发 `Holding`。

:::info
注意：两次点击之间允许的最大距离，以及它们之间的最大时间间隔，都取决于目标平台，并且在触摸设备上通常会更宽松。
:::


## 指针捕获

指针捕获会将后续所有指针事件都定向发送到某个特定控件，即使指针已经移动到了控件边界之外。这对于拖拽操作和类似滑块的交互来说至关重要。

```csharp
protected override void OnPointerPressed(PointerPressedEventArgs e)
{
    base.OnPointerPressed(e);
    e.Pointer.Capture(this);
}

protected override void OnPointerReleased(PointerReleasedEventArgs e)
{
    base.OnPointerReleased(e);
    e.Pointer.Capture(null); // 释放捕获
}
```

整个应用中同一时刻只能有一个元素持有指针捕获。如果另一个控件捕获了指针，先前控件的捕获就会被释放，并收到一个 `PointerCaptureLost` 事件。你应处理这个事件，以清理所有进行中的交互状态：

```csharp
protected override void OnPointerCaptureLost(PointerCaptureLostEventArgs e)
{
    base.OnPointerCaptureLost(e);
    _isDragging = false;
}
```

## 光标管理

### 在控件上设置光标

当指针位于控件上方时，可以使用 `Cursor` 属性改变光标样式：

```xml
<Border Cursor="Hand" Background="LightGray" Padding="20">
    <TextBlock Text="点我" />
</Border>

<Border Cursor="SizeWestEast" Background="LightBlue" Padding="20">
    <TextBlock Text="水平调整大小" />
</Border>
```

### 常见光标类型

| 光标 | 说明 |
|---|---|
| `Arrow` | 默认箭头指针。 |
| `Hand` | 手形指针（表示可点击元素）。 |
| `IBeam` | 文本编辑光标。 |
| `Cross` | 十字准星。 |
| `SizeWestEast` | 水平调整大小。 |
| `SizeNorthSouth` | 垂直调整大小。 |
| `SizeAll` | 任意方向移动/拖拽。 |
| `Wait` | 忙碌指示（沙漏/旋转图标）。 |
| `AppStarting` | 带小沙漏的箭头。 |
| `No` | 不允许（带斜线的圆圈）。 |
| `None` | 隐藏光标。 |

### 在代码中设置光标

```csharp
myControl.Cursor = new Cursor(StandardCursorType.Hand);
```

### 在拖拽操作期间改变光标

```csharp
private void OnPointerMoved(object? sender, PointerEventArgs e)
{
    if (_isDragging)
    {
        Cursor = new Cursor(StandardCursorType.SizeAll);
    }
}

private void OnPointerReleased(object? sender, PointerReleasedEventArgs e)
{
    _isDragging = false;
    Cursor = Cursor.Default;
}
```

## 更多信息

有关指针和点击事件的完整 API 文档，请参阅 [PointerEventArgs API 参考](/api/avalonia/input/pointereventargs)。

## 另请参阅

- [手势](/docs/input-interaction/gestures)：构建在指针事件之上的更高层手势事件。
- [拖放](/docs/input-interaction/drag-and-drop)：使用指针事件实现拖放操作。
- [路由事件](/docs/input-interaction/routed-events)：事件如何在元素树中传播。
