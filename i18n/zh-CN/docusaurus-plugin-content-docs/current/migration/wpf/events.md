---
id: events
title: 事件
description: WPF 与 Avalonia 在路由事件、隧道阶段和事件命名方面的差异。
doc-type: migration
---

Avalonia 的事件系统在概念上与 WPF 的路由事件模型相似。事件可以沿可视树向上冒泡，也可以向下隧道传播，并且你同样可以注册类处理器或实例处理器。不过，在 API 形式、事件命名以及隧道阶段的处理方式上，仍存在一些重要差异。本指南将概括从 WPF 迁移到 Avalonia 时需要了解的关键区别。

## 路由事件

WPF 和 Avalonia 都支持路由事件，但注册 API 不同。在 WPF 中你使用 `EventManager.RegisterRoutedEvent`，而在 Avalonia 中则使用 `RoutedEvent.Register`。

```csharp title='WPF'
public static readonly RoutedEvent TapEvent = EventManager.RegisterRoutedEvent(
    "Tap",
    RoutingStrategy.Bubble,
    typeof(RoutedEventHandler),
    typeof(MyControl));
```

```csharp title='Avalonia'
public static readonly RoutedEvent<RoutedEventArgs> TapEvent = RoutedEvent.Register<MyControl, RoutedEventArgs>(
    "Tap",
    RoutingStrategy.Bubble);
```

需要注意的关键差异：

- Avalonia 使用泛型 `RoutedEvent<TEventArgs>` 类型，因此对事件参数提供了更强的类型约束。
- Avalonia 的注册调用通过泛型参数指定拥有者类型和事件参数类型，而不是传入 `typeof()`。
- 在 Avalonia 中，委托类型可以从泛型参数中自动推断，因此不必显式指定。

## 类处理器

在 WPF 中，可以通过调用 [EventManager.RegisterClassHandler](https://msdn.microsoft.com/en-us/library/ms597875.aspx) 来为事件添加类处理器。而在 Avalonia 中，你需要直接在路由事件实例上调用 `AddClassHandler`。

```csharp title='WPF'
static MyControl()
{
    EventManager.RegisterClassHandler(typeof(MyControl), MyEvent, HandleMyEvent));
}

private static void HandleMyEvent(object sender, RoutedEventArgs e)
{
}
```

```csharp title='Avalonia'
static MyControl()
{
    MyEvent.AddClassHandler<MyControl>((x, e) => x.HandleMyEvent(e));
}

private void HandleMyEvent(RoutedEventArgs e)
{
}
```

请注意，在 WPF 中类处理器必须是静态方法；而在 Avalonia 中，类处理器本身不需要是静态的，通知会自动定向到正确的实例。在这种情况下，常规事件处理器中的 `sender` 参数也不再必要，整体类型信息依旧保持强类型。

## 隧道事件

在 WPF 中，隧道（preview）事件会以带有 `Preview` 前缀的独立 CLR 事件形式暴露出来。例如，`PreviewKeyDown` 就是 `KeyDown` 的隧道对应版本。它们是两个彼此独立的 CLR 事件，你可以分别订阅。

Avalonia 采用了不同的方式。它不存在单独的 `Preview*` CLR 事件；相反，隧道和冒泡共用同一个 `RoutedEvent` 实例。若要订阅隧道阶段，你需要调用 `AddHandler` 并传入 `RoutingStrategies.Tunnel`。

```csharp title='WPF'
// 在 WPF 中，直接订阅 Preview 事件
myControl.PreviewKeyDown += OnPreviewKeyDown;

void OnPreviewKeyDown(object sender, KeyEventArgs e)
{
    // 隧道阶段处理器
}
```

```csharp title='Avalonia'
// 在 Avalonia 中，使用 AddHandler 并传入 RoutingStrategies.Tunnel
myControl.AddHandler(InputElement.KeyDownEvent, OnPreviewKeyDown, RoutingStrategies.Tunnel);

void OnPreviewKeyDown(object? sender, KeyEventArgs e)
{
    // 隧道阶段处理器
}
```

你也可以通过组合标志位，一次性同时订阅隧道和冒泡两个阶段：

```csharp title='Avalonia'
myControl.AddHandler(
    InputElement.KeyDownEvent,
    OnKeyDown,
    RoutingStrategies.Tunnel | RoutingStrategies.Bubble);
```

## 添加事件处理器的方式

### XAML 事件处理器

在 XAML 中附加事件处理器，WPF 与 Avalonia 的方式是相同的：

```xml
<Button Click="OnButtonClick" />
```

### 在 code-behind 中使用 AddHandler

在 WPF 中，`AddHandler` 接收路由事件和委托。Avalonia 的 `AddHandler` 则还额外支持传入路由策略和 handled-events 行为参数。

```csharp title='WPF'
myButton.AddHandler(Button.ClickEvent, new RoutedEventHandler(OnButtonClick));
```

```csharp title='Avalonia'
myButton.AddHandler(Button.ClickEvent, OnButtonClick);
```

### handledEventsToo 参数

WPF 与 Avalonia 都支持在事件已经被标记为 handled 之后继续接收它。这个参数在两个框架中的作用基本一致。

```csharp title='WPF'
myControl.AddHandler(
    UIElement.MouseDownEvent,
    new MouseButtonEventHandler(OnMouseDown),
    handledEventsToo: true);
```

```csharp title='Avalonia'
myControl.AddHandler(
    InputElement.PointerPressedEvent,
    OnPointerPressed,
    RoutingStrategies.Bubble,
    handledEventsToo: true);
```

需要注意的是，在 Avalonia 中，你必须先指定 `RoutingStrategies` 参数，之后才能传入 `handledEventsToo`。

## 常见事件名称差异

与 WPF 相比，Avalonia 中许多输入事件的名称都发生了变化。下表列出了最常见的映射关系：

| WPF 事件 | Avalonia 对应项 | 说明 |
|---|---|---|
| `MouseLeftButtonDown` | `PointerPressed` | 通过 [`PointerUpdateKind`](/api/avalonia/input/pointerupdatekind) 判断按钮类型 |
| `MouseLeftButtonUp` | `PointerReleased` | 通过 `PointerUpdateKind` 判断按钮类型 |
| `MouseRightButtonDown` | `PointerPressed` | 通过 `PointerUpdateKind` 判断按钮类型 |
| `MouseRightButtonUp` | `PointerReleased` | 通过 `PointerUpdateKind` 判断按钮类型 |
| `MouseMove` | `PointerMoved` | |
| `MouseEnter` | `PointerEntered` | |
| `MouseLeave` | `PointerExited` | |
| `MouseWheel` | `PointerWheelChanged` | |
| `PreviewKeyDown` | 在 `KeyDownEvent` 上使用 `AddHandler` + `RoutingStrategies.Tunnel` | 不存在单独的 Preview 事件 |
| `PreviewKeyUp` | 在 `KeyUpEvent` 上使用 `AddHandler` + `RoutingStrategies.Tunnel` | 不存在单独的 Preview 事件 |
| `PreviewMouseDown` | 在 `PointerPressedEvent` 上使用 `AddHandler` + `RoutingStrategies.Tunnel` | 不存在单独的 Preview 事件 |

Avalonia 使用基于 pointer 的事件命名，是因为它支持超出鼠标范围的多种指针设备，包括触摸和笔输入。

## 自定义路由事件

在定义自定义路由事件时，WPF 与 Avalonia 的注册模式也有所不同。下面给出一组完整对比，展示如何定义、注册并触发自定义路由事件。

```csharp title='WPF'
public class MyControl : Control
{
    public static readonly RoutedEvent TapEvent = EventManager.RegisterRoutedEvent(
        "Tap",
        RoutingStrategy.Bubble,
        typeof(RoutedEventHandler),
        typeof(MyControl));

    public event RoutedEventHandler Tap
    {
        add => AddHandler(TapEvent, value);
        remove => RemoveHandler(TapEvent, value);
    }

    protected void OnTap()
    {
        RaiseEvent(new RoutedEventArgs(TapEvent));
    }
}
```

```csharp title='Avalonia'
public class MyControl : Control
{
    public static readonly RoutedEvent<RoutedEventArgs> TapEvent = RoutedEvent.Register<MyControl, RoutedEventArgs>(
        "Tap",
        RoutingStrategy.Bubble);

    public event EventHandler<RoutedEventArgs>? Tap
    {
        add => AddHandler(TapEvent, value);
        remove => RemoveHandler(TapEvent, value);
    }

    protected void OnTap()
    {
        RaiseEvent(new RoutedEventArgs(TapEvent));
    }
}
```

主要差异如下：

- Avalonia 使用泛型 `RoutedEvent<T>` 来提供类型安全。
- Avalonia 中的 CLR 事件包装器使用的是 `EventHandler<RoutedEventArgs>`，而不是 `RoutedEventHandler`。
- 注册过程使用泛型参数，而不是 `typeof()` 参数。

## 另请参阅

- [路由事件概览](/docs/events)
- [输入事件](/docs/events/input-events)
- [路由事件深入解析](/docs/input-interaction/routed-events)
