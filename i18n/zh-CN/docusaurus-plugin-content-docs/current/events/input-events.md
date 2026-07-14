---
id: input-events
title: 输入事件
description: 处理 Avalonia 控件中的指针、键盘和手势输入事件。
doc-type: reference
---

Avalonia 提供了一整套输入事件，用于处理指针（鼠标/触摸/笔）、键盘以及手势交互。大多数输入事件都使用组合式的 `Tunnel | Bubble` 路由策略，使父元素有机会在输入到达目标之前先行拦截。

## 指针事件

指针事件将鼠标、触摸和手写笔输入抽象成统一模型。默认情况下，它们会沿视觉树向上冒泡。

| 事件 | 触发时机 |
|---|---|
| `PointerEntered` | 指针进入控件边界时。 |
| `PointerExited` | 指针离开控件边界时。 |
| `PointerMoved` | 指针在控件内部移动时。 |
| `PointerPressed` | 指针按钮在控件上被按下时。 |
| `PointerReleased` | 指针按钮在控件上被释放时。 |
| `PointerCaptureLost` | 控件失去指针捕获时。 |
| `PointerWheelChanged` | 鼠标滚轮或触控板在控件上滚动时。 |

### 处理指针事件

```csharp
protected override void OnPointerPressed(PointerPressedEventArgs e)
{
    base.OnPointerPressed(e);

    var point = e.GetPosition(this); // Position relative to this control
    var properties = e.GetCurrentPoint(this).Properties;

    if (properties.IsLeftButtonPressed)
    {
        // 左键在 (point.X, point.Y) 位置被按下
    }
}
```

### `PointerEventArgs` 上的关键属性

| 属性 / 方法 | 说明 |
|---|---|
| `GetPosition(Visual)` | 返回相对于指定视觉对象的指针位置。 |
| `GetCurrentPoint(Visual)` | 返回包含位置和按钮状态的 `PointerPoint`。 |
| `Pointer` | 当前的 `Pointer` 实例，可用于捕获操作。 |
| `KeyModifiers` | 表示当前是否按住了 Shift、Control、Alt 或 Meta 键。 |

### 指针捕获

当你捕获指针后，后续所有指针事件都会被定向发送到执行捕获的控件，直到捕获被释放为止：

```csharp
protected override void OnPointerPressed(PointerPressedEventArgs e)
{
    base.OnPointerPressed(e);
    e.Pointer.Capture(this); // 开始捕获
}

protected override void OnPointerReleased(PointerReleasedEventArgs e)
{
    base.OnPointerReleased(e);
    e.Pointer.Capture(null); // 释放捕获
}
```

在整个应用中，同一时刻只能有一个元素持有指针捕获。这与操作系统的行为一致：同一个物理鼠标设备一次只能被一个元素捕获。当另一个控件开始捕获指针时（例如在弹出窗口中），原先的捕获会被释放，而最初的控件会收到一个 `PointerCaptureLost` 事件。

## 键盘事件

键盘事件会在当前获得焦点的元素上触发，并沿树向上冒泡。

| 事件 | 触发时机 |
|---|---|
| `KeyDown` | 某个按键被按下时。 |
| `KeyUp` | 某个按键被释放时。 |
| `TextInput` | 接收到字符输入时（经过 IME 处理后）。 |

### 处理键盘事件

```csharp
protected override void OnKeyDown(KeyEventArgs e)
{
    base.OnKeyDown(e);

    if (e.Key == Key.Enter)
    {
        // 处理 Enter 键
        e.Handled = true;
    }

    if (e.Key == Key.C && e.KeyModifiers.HasFlag(KeyModifiers.Control))
    {
        // 处理 Ctrl+C
        e.Handled = true;
    }
}
```

### `KeyEventArgs` 上的关键属性

| 属性 | 说明 |
|---|---|
| `Key` | 被按下的物理按键（来自 `Key` 枚举）。 |
| `KeyModifiers` | 当前按住的修饰键（Control、Shift、Alt、Meta）。 |
| `KeySymbol` | 此次按键产生的字符（如果有）。 |

## 隧道路由（预览）事件

对于使用 `Tunnel | Bubble` 路由的输入事件来说，隧道阶段会先触发。你可以通过指定路由策略参数，在隧道阶段拦截事件：

```csharp
myControl.AddHandler(InputElement.PointerPressedEvent, OnPreviewPointerPressed,
    RoutingStrategies.Tunnel);
```

```csharp
private void OnPreviewPointerPressed(object? sender, PointerPressedEventArgs e)
{
    // 这会在冒泡阶段之前触发
    // 设置 e.Handled = true 可以阻止事件继续传递到子元素
}
```

这对于在子控件处理输入之前，由父级统一拦截输入非常有用。

## 手势事件

Avalonia 在原始指针事件之上，提供了更高层级的手势事件：

| 事件 | 触发时机 |
|---|---|
| `Tapped` | 一次快速点击/轻触手势完成时。 |
| `DoubleTapped` | 一次双击/双轻触手势完成时。 |
| `Holding` | 检测到长按手势时（触摸）。 |

手势事件也会沿树向上冒泡。若要处理更复杂的手势（如捏合、拉动、滚动），请参阅 [Gestures](/docs/input-interaction/gestures)。

```xml
<Border Tapped="OnBorderTapped" Background="LightGray">
    <TextBlock Text="Tap me" />
</Border>
```

```csharp
private void OnBorderTapped(object? sender, TappedEventArgs e)
{
    // 响应点击手势
}
```

## 常见输入模式

### 拖拽检测

```csharp
private Point _pressPoint;
private bool _isDragging;

protected override void OnPointerPressed(PointerPressedEventArgs e)
{
    base.OnPointerPressed(e);
    _pressPoint = e.GetPosition(this);
    _isDragging = false;
    e.Pointer.Capture(this);
}

protected override void OnPointerMoved(PointerEventArgs e)
{
    base.OnPointerMoved(e);

    if (e.GetCurrentPoint(this).Properties.IsLeftButtonPressed)
    {
        var currentPoint = e.GetPosition(this);
        var delta = currentPoint - _pressPoint;

        if (!_isDragging && (Math.Abs(delta.X) > 3 || Math.Abs(delta.Y) > 3))
        {
            _isDragging = true;
        }

        if (_isDragging)
        {
            // 处理拖拽
        }
    }
}

protected override void OnPointerReleased(PointerReleasedEventArgs e)
{
    base.OnPointerReleased(e);
    _isDragging = false;
    e.Pointer.Capture(null);
}
```

### 窗口中的键盘快捷键

```csharp
public partial class MainWindow : Window
{
    public MainWindow()
    {
        InitializeComponent();
    }

    protected override void OnKeyDown(KeyEventArgs e)
    {
        base.OnKeyDown(e);

        if (e.Key == Key.S && e.KeyModifiers.HasFlag(KeyModifiers.Control))
        {
            SaveDocument();
            e.Handled = true;
        }
    }
}
```

:::tip
如果你更希望以声明方式处理键盘快捷键，可以考虑使用 [KeyBindings and HotKeys](/docs/input-interaction/keyboard-and-hotkeys)，而不是手动处理 `KeyDown`。
:::

## 另请参阅

- [Events Overview](/docs/events): How routed events work in Avalonia.
- [Pointer Input](/docs/input-interaction/pointer): Detailed pointer input reference.
- [Keyboard and Hotkeys](/docs/input-interaction/keyboard-and-hotkeys): Keyboard shortcuts and key bindings.
- [Gestures](/docs/input-interaction/gestures): Touch and multi-pointer gesture recognition.
- [Focus](/docs/input-interaction/focus): How keyboard focus works.
