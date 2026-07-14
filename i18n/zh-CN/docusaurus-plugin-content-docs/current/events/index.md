---
id: index
title: 事件概览
description: 了解 Avalonia 中路由事件如何沿元素树传播。
doc-type: overview
---

Avalonia 使用与 WPF 类似的路由事件系统。路由事件可以沿元素树传播，因此父元素可以处理其子元素引发的事件。这是 Avalonia 中输入、交互和控件行为工作的基础。

## 事件路由策略

每个路由事件都有一种路由策略，用于决定事件如何沿元素树传播：

| 策略 | 方向 | 说明 |
|---|---|---|
| `Bubble` | 子元素到父元素 | 事件先在源元素上触发，然后沿着每一层父元素向上传播，直到根节点。这是最常见的策略。 |
| `Tunnel` | 父元素到子元素 | 事件先在根元素上触发，然后沿树向下传播到源元素。隧道路由通常用于预览或拦截场景。 |
| `Direct` | 仅源元素 | 事件只会在源元素上触发，不会沿元素树传播。 |

事件也可以组合多种策略。例如，许多输入事件会同时使用 `Tunnel | Bubble`，这意味着事件会先从根节点向下隧道传播，再从源元素向上冒泡返回。

### 冒泡示例

当用户点击位于 `Window` 中 `StackPanel` 内部的 `Button` 时：

```text
Window          ← 事件最后到达这里（冒泡）
  └─ StackPanel ← 事件第二个到达这里
       └─ Button ← 事件从这里开始（源）
```

### 隧道示例

同一棵元素树中的隧道事件如下：

```text
Window          ← 事件先从这里开始（隧道）
  └─ StackPanel ← 事件第二个到达这里
       └─ Button ← 事件最后到达这里（源）
```

## 处理路由事件

### 在 XAML 中

使用事件名作为属性即可附加事件处理程序：

```xml
<Button Click="OnButtonClick" Content="Click me" />
```

```csharp
private void OnButtonClick(object? sender, RoutedEventArgs e)
{
    // sender 是被点击的 Button
    // e.Source 是事件的原始源
}
```

### 在代码中

使用 `AddHandler` 和 `RemoveHandler`：

```csharp
myButton.AddHandler(Button.ClickEvent, OnButtonClick);

// 之后如需取消订阅：
myButton.RemoveHandler(Button.ClickEvent, OnButtonClick);
```

### 在父元素上处理冒泡事件

由于事件会沿树向上冒泡，因此你可以在父元素上处理子元素触发的事件：

```xml
<StackPanel Tapped="OnStackPanelTapped">
    <Button Content="Button 1" />
    <Button Content="Button 2" />
    <Button Content="Button 3" />
</StackPanel>
```

```csharp
private void OnStackPanelTapped(object? sender, TappedEventArgs e)
{
    // sender 是 StackPanel（处理程序附加的位置）
    // e.Source 是被点击的具体 Button
    if (e.Source is Button button)
    {
        Debug.WriteLine($"Tapped: {button.Content}");
    }
}
```

## 将事件标记为已处理

设置 `e.Handled = true` 可阻止事件继续路由：

```csharp
private void OnButtonClick(object? sender, RoutedEventArgs e)
{
    e.Handled = true; // 阻止父级处理程序接收到该事件
}
```

如果你需要接收那些已经被标记为已处理的事件，可以使用 `handledEventsToo` 参数：

```csharp
myPanel.AddHandler(Button.ClickEvent, OnButtonClick, RoutingStrategies.Bubble, handledEventsToo: true);
```

## `RoutedEventArgs` 属性

| 属性 | 类型 | 说明 |
|---|---|---|
| `Source` | `object?` | 最初引发事件的元素。 |
| `Handled` | `bool` | 事件是否已被处理。设为 `true` 可停止继续路由。 |
| `Route` | `RoutingStrategies` | 当前的路由阶段（`Tunnel`、`Bubble` 或 `Direct`）。 |
| `RoutedEvent` | `RoutedEvent` | 当前正在引发的路由事件。 |

## 注册自定义路由事件

在控件中定义自定义路由事件：

```csharp
public class MyControl : Control
{
    public static readonly RoutedEvent<RoutedEventArgs> ValueChangedEvent =
        RoutedEvent.Register<MyControl, RoutedEventArgs>(
            nameof(ValueChanged),
            RoutingStrategies.Bubble);

    public event EventHandler<RoutedEventArgs>? ValueChanged
    {
        add => AddHandler(ValueChangedEvent, value);
        remove => RemoveHandler(ValueChangedEvent, value);
    }

    protected virtual void OnValueChanged()
    {
        RaiseEvent(new RoutedEventArgs(ValueChangedEvent));
    }
}
```

### 自定义事件参数

对于需要携带额外数据的事件，可以创建一个自定义的 `RoutedEventArgs` 子类：

```csharp
public class ValueChangedEventArgs : RoutedEventArgs
{
    public ValueChangedEventArgs(RoutedEvent routedEvent, double oldValue, double newValue)
        : base(routedEvent)
    {
        OldValue = oldValue;
        NewValue = newValue;
    }

    public double OldValue { get; }
    public double NewValue { get; }
}
```

## 类处理程序

类处理程序允许你为某一类型的所有实例统一响应事件，通常会在静态构造函数中注册。类处理程序会在实例级处理程序之前执行。

```csharp
public class MyControl : Control
{
    static MyControl()
    {
        PointerPressedEvent.AddClassHandler<MyControl>((control, args) =>
        {
            control.OnPointerPressedInternal(args);
        });
    }

    private void OnPointerPressedInternal(PointerPressedEventArgs args)
    {
        // 为 MyControl 的所有实例统一处理
    }
}
```

如果控件实现需要在任何实例级处理程序将事件标记为已处理之前先拦截输入事件，类处理程序就非常有用。

## 后续阅读

- [路由事件](/docs/input-interaction/routed-events)：路由事件系统的详细参考。
- [生命周期事件](/docs/events/lifecycle-events)：控件创建、加载和卸载过程中触发的事件。
- [输入事件](/docs/events/input-events)：指针、键盘和手势事件。
- [添加交互](/docs/input-interaction/adding-interactivity)：处理用户交互的实用指南。
