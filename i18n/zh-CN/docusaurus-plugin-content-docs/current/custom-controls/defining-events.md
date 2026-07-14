---
id: defining-events
title: 定义事件
description: 在自定义 Avalonia 控件上定义并引发具有隧道和冒泡策略的路由事件。
doc-type: how-to
---

Avalonia 中的事件机制可让你的自定义控件传达信息，并将特定操作或状态变化通知给使用者。通过定义事件，你可以为控件使用者提供一种在其应用中响应这些事件的方式。本文将引导你完成自定义控件事件的定义过程。

## 路由事件

Avalonia 中的路由事件提供了一种可沿控件树传播（即“路由”）的事件处理机制，使多个控件能够响应同一个事件。路由事件具有以下关键特性：

- **事件路由**：路由事件可以沿树向上传播（冒泡）或向下传播（隧道），从而使不同层级的控件都能处理同一事件。这使事件处理更加灵活且更易集中管理。

- **事件处理程序**：路由事件通过事件处理程序来响应事件。处理程序可以关联到特定控件，也可以附加到可视树中的更高层级，以统一处理多个控件触发的事件。

- **已处理状态**：路由事件带有 `Handled` 属性，可用于将事件标记为已处理，从而阻止其继续传播。这让你能够更细粒度地控制事件处理流程。

- **事件路由策略**：Avalonia 支持多种路由事件策略，例如冒泡、隧道和直接路由。这些策略决定了控件接收和处理事件的顺序。
当你需要处理嵌套控件中发生的事件，或者希望将事件处理逻辑集中到可视树更高层时，Avalonia 路由事件尤其有用。

## 示例

下面示例展示了如何为一个假设的自定义滑块控件定义路由事件：

```csharp
public class MyCustomSlider : Control
{
    public static readonly RoutedEvent<RoutedEventArgs> ValueChangedEvent =
        RoutedEvent.Register<MyCustomSlider, RoutedEventArgs>(nameof(ValueChanged), RoutingStrategies.Direct);

    public event EventHandler<RoutedEventArgs> ValueChanged
    {
        add => AddHandler(ValueChangedEvent, value);
        remove => RemoveHandler(ValueChangedEvent, value);
    }

    protected virtual void OnValueChanged()
    {
        RoutedEventArgs args = new RoutedEventArgs(ValueChangedEvent);
        RaiseEvent(args);
    }
}
```

在这个示例中，`MyCustomSlider` 控件定义了一个名为 `ValueChangedEvent` 的自定义路由事件。该事件通过 `RoutedEvent` 系统注册，因此控件使用者可以订阅它。为了方便使用，还额外定义了一个 CLR 事件包装器，使该事件能够以符合标准 .NET API 习惯的方式被消费。

## 自定义事件参数

当事件需要携带额外数据时，可以创建一个继承自 `RoutedEventArgs` 的自定义类：

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

然后将事件注册改为使用这个自定义参数类型：

```csharp
public static readonly RoutedEvent<ValueChangedEventArgs> ValueChangedEvent =
    RoutedEvent.Register<MyCustomSlider, ValueChangedEventArgs>(
        nameof(ValueChanged), RoutingStrategies.Bubble);
```

对应的 CLR 事件包装器和引发方法也应同步更新：

```csharp
public event EventHandler<ValueChangedEventArgs> ValueChanged
{
    add => AddHandler(ValueChangedEvent, value);
    remove => RemoveHandler(ValueChangedEvent, value);
}

protected virtual void OnValueChanged(double oldValue, double newValue)
{
    var args = new ValueChangedEventArgs(ValueChangedEvent, oldValue, newValue);
    RaiseEvent(args);
}
```

## 路由策略

Avalonia 支持多种路由策略，用于控制事件如何沿可视树传播：

| 策略 | 行为 |
|----------|----------|
| `Direct` | 事件仅在源控件上触发 |
| `Bubble` | 事件从源控件向上冒泡到根节点 |
| `Tunnel` | 事件从根节点向下隧道传播到源控件 |
| `Bubble \| Tunnel` | 事件先向下隧道传播，再向上冒泡 |

你可以使用按位或运算符组合多种策略：

```csharp
RoutedEvent.Register<MyControl, RoutedEventArgs>(
    nameof(MyEvent), RoutingStrategies.Bubble | RoutingStrategies.Tunnel);
```

同时使用 `Bubble | Tunnel` 在以下场景很有帮助：你希望父级控件在源控件处理事件之前（隧道阶段）先有机会预览并拦截该事件，而源控件则在之后的冒泡阶段处理它。

## 在 XAML 中处理事件

你的自定义控件使用者可以直接在 XAML 中订阅该事件：

```xml
<local:MyCustomSlider ValueChanged="OnSliderValueChanged" />
```

对应的代码后置处理程序如下：

```csharp
private void OnSliderValueChanged(object? sender, ValueChangedEventArgs e)
{
    Debug.WriteLine($"Value changed from {e.OldValue} to {e.NewValue}");
}
```

## 类处理程序

类处理程序允许你在类型级别而不是单个实例级别注册事件处理逻辑。这对于需要应用到控件所有实例的默认行为非常有用：

```csharp
static MyCustomSlider()
{
    ValueChangedEvent.AddClassHandler<MyCustomSlider>((s, e) => s.OnValueChanged(e));
}

private void OnValueChanged(ValueChangedEventArgs e)
{
    // 处理值变化
}
```

类处理程序会先于实例处理程序执行，并且通常在静态构造函数中注册，因此会自动应用到控件的每一个实例上。

## 另请参阅

- [路由事件](/docs/input-interaction/routed-events)：路由事件的完整参考。
- [事件系统](/docs/events)：事件系统概览。
- [输入事件](/docs/events/input-events)：内置输入事件。
