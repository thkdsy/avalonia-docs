---
id: properties
title: 属性系统
description: 将 WPF 中基于 DependencyProperty 的写法迁移到 Avalonia 的 StyledProperty 和 DirectProperty 类型。
doc-type: migration
---

Avalonia 的属性系统在概念上与 WPF 的 `DependencyProperty` 系统相似，但它使用了更简洁、强类型的泛型 API。如果你已经熟悉 WPF 的依赖属性，那么在 Avalonia 中仍然能看到大部分相同概念：样式、数据绑定、动画、值继承以及默认值都仍然通过属性系统来工作。主要差异在于属性注册语法，以及你如何响应属性变化。

## 属性类型对比

WPF 使用单一的 `DependencyProperty` 类来覆盖所有场景。而 Avalonia 将它拆分为三种不同类型，每种类型都针对特定用途进行了优化。这三种类型共享同一个基类：`AvaloniaProperty`。

| WPF | Avalonia | 适用场景 |
|---|---|---|
| `DependencyProperty` | `StyledProperty` | 参与样式、动画和值继承的属性 |
| `DependencyProperty`（只读） | `DirectProperty` | 只读属性、性能敏感属性，或包装 CLR 后备字段的属性 |
| `DependencyProperty.RegisterAttached` | `AttachedProperty` | 设置在子元素上的属性（例如 `Grid.Row`、`DockPanel.Dock`） |

## 注册方式

### StyledProperty

在 WPF 中，通常通过一个静态字段和 `DependencyProperty.Register` 调用来注册 `DependencyProperty`。而在 Avalonia 中，对应做法是使用 `AvaloniaProperty.Register<TOwner, TValue>`。

**WPF:**

```csharp
public class MyControl : Control
{
    public static readonly DependencyProperty BackgroundProperty =
        DependencyProperty.Register(
            nameof(Background),
            typeof(Brush),
            typeof(MyControl),
            new PropertyMetadata(Brushes.Transparent));

    public Brush Background
    {
        get => (Brush)GetValue(BackgroundProperty);
        set => SetValue(BackgroundProperty, value);
    }
}
```

**Avalonia:**

```csharp
public class MyControl : Control
{
    public static readonly StyledProperty<IBrush> BackgroundProperty =
        AvaloniaProperty.Register<MyControl, IBrush>(
            nameof(Background),
            defaultValue: Brushes.Transparent);

    public IBrush Background
    {
        get => GetValue(BackgroundProperty);
        set => SetValue(BackgroundProperty, value);
    }
}
```

请注意，Avalonia 通过泛型避免了在 `GetValue` 调用中进行类型转换，同时默认值是通过命名参数传入，而不是通过 metadata 对象提供。

### DirectProperty

`DirectProperty` 会直接从 CLR 后备字段读写，而不是经过 Avalonia 属性系统内部的值存储。这使它非常适合只读属性，或者那些你希望获得更高性能的属性。WPF 中没有完全对应的概念，最接近的是只读 `DependencyProperty`。

```csharp
public class MyControl : Control
{
    public static readonly DirectProperty<MyControl, string> StatusProperty =
        AvaloniaProperty.RegisterDirect<MyControl, string>(
            nameof(Status),
            o => o.Status);

    private string _status = "Ready";

    public string Status
    {
        get => _status;
        private set => SetAndRaise(StatusProperty, ref _status, value);
    }
}
```

要点如下：
- 使用 `SetAndRaise` 而不是 `SetValue` 来更新后备字段并触发变更通知。
- getter 访问器 lambda（`o => o.Status`）是必需的，这样属性系统才能读取当前值。

### AttachedProperty

附加属性在概念上的工作方式是相同的。在 WPF 中你使用 `DependencyProperty.RegisterAttached`；而在 Avalonia 中使用 `AvaloniaProperty.RegisterAttached`。

**WPF:**

```csharp
public class DockPanel : Panel
{
    public static readonly DependencyProperty DockProperty =
        DependencyProperty.RegisterAttached(
            "Dock",
            typeof(Dock),
            typeof(DockPanel),
            new PropertyMetadata(Dock.Left));

    public static Dock GetDock(DependencyObject element)
        => (Dock)element.GetValue(DockProperty);

    public static void SetDock(DependencyObject element, Dock value)
        => element.SetValue(DockProperty, value);
}
```

**Avalonia:**

```csharp
public class DockPanel : Panel
{
    public static readonly AttachedProperty<Dock> DockProperty =
        AvaloniaProperty.RegisterAttached<DockPanel, Control, Dock>(
            "Dock",
            defaultValue: Dock.Left);

    public static Dock GetDock(Control element)
        => element.GetValue(DockProperty);

    public static void SetDock(Control element, Dock value)
        => element.SetValue(DockProperty, value);
}
```

## 属性变更回调

### WPF 的方式

在 WPF 中，你会在注册属性时，通过 `PropertyMetadata` 传入 `PropertyChangedCallback`：

```csharp
public static readonly DependencyProperty IsActiveProperty =
    DependencyProperty.Register(
        nameof(IsActive),
        typeof(bool),
        typeof(MyControl),
        new PropertyMetadata(false, OnIsActiveChanged));

private static void OnIsActiveChanged(DependencyObject d, DependencyPropertyChangedEventArgs e)
{
    var control = (MyControl)d;
    control.UpdateVisualState();
}
```

### Avalonia 的方式

Avalonia 提供两种响应属性变化的方式。

**方式 1：重写 `OnPropertyChanged`**

对于控件作者来说，推荐的方式是在控件自身上重写 `OnPropertyChanged`：

```csharp
protected override void OnPropertyChanged(AvaloniaPropertyChangedEventArgs change)
{
    base.OnPropertyChanged(change);

    if (change.Property == IsActiveProperty)
    {
        var newValue = change.GetNewValue<bool>();
        UpdateVisualState();
    }
}
```

**方式 2：通过 `Changed.AddClassHandler` 注册类处理器**

你也可以注册一个静态类处理器，通常写在控件的静态构造函数中。这在理念上类似于 WPF 的 `PropertyChangedCallback`，但它是与属性定义分开注册的：

```csharp
static MyControl()
{
    IsActiveProperty.Changed.AddClassHandler<MyControl>((control, args) =>
    {
        control.UpdateVisualState();
    });
}
```

这两种方式在效果上是等价的。如果你需要在同一个位置处理多个属性的变化，那么重写 `OnPropertyChanged` 往往会更清晰。

## 默认值

在 WPF 中，默认值通过 `PropertyMetadata` 对象提供：

```csharp
new PropertyMetadata(defaultValue: Brushes.White)
```

在 Avalonia 中，默认值则通过 `Register` 方法上的命名参数传入：

```csharp
AvaloniaProperty.Register<MyControl, IBrush>(
    nameof(Background),
    defaultValue: Brushes.White);
```

如果你需要在派生类中覆盖默认值，请在派生类的静态构造函数中使用 `OverrideDefaultValue`：

```csharp
static MyDerivedControl()
{
    BackgroundProperty.OverrideDefaultValue<MyDerivedControl>(Brushes.Black);
}
```

## 值强制修正

在 WPF 中，你会在 `PropertyMetadata` 中提供 `CoerceValueCallback`：

```csharp
new PropertyMetadata(0.0, null, CoerceOpacity)
```

在 Avalonia 中，则是在注册属性时传入 `coerce` 函数：

```csharp
public static readonly StyledProperty<double> OpacityProperty =
    AvaloniaProperty.Register<MyControl, double>(
        nameof(Opacity),
        defaultValue: 1.0,
        coerce: CoerceOpacity);

private static double CoerceOpacity(AvaloniaObject sender, double value)
{
    return Math.Clamp(value, 0.0, 1.0);
}
```

这个修正函数会接收 `AvaloniaObject` 实例以及待写入的值，并返回修正后的值。

## 值优先级

WPF 与 Avalonia 都使用值优先级系统来决定属性的最终有效值。大致顺序（从高到低）如下：

1. Animation
2. Local value
3. Style triggers / Style setters
4. Template parent
5. Inherited value
6. Default value

有关 Avalonia 如何解析属性值的详细说明，请参阅 [值优先级](/docs/properties/value-precedence)。

## 常见陷阱

### 不存在带默认值的 PropertyMetadata 构造函数

在 WPF 中，你经常会写出 `new PropertyMetadata(someDefault)`。而在 Avalonia 中并不存在 `PropertyMetadata` 类，默认值需要通过 `defaultValue:` 命名参数直接传给 `Register`。

### 对于 DirectProperty，SetAndRaise 取代 SetValue

如果你注册的是 `DirectProperty`，那么在 CLR setter 中必须使用 `SetAndRaise`，而不能使用 `SetValue`。对 `DirectProperty` 调用 `SetValue` 会抛出异常。

```csharp
// Correct for DirectProperty
public string Status
{
    get => _status;
    private set => SetAndRaise(StatusProperty, ref _status, value);
}
```

### StyledProperty 的值存储在属性系统中

与 `DirectProperty` 不同，`StyledProperty` 不使用后备字段。它的值由 Avalonia 属性系统在内部存储。如果你尝试额外添加一个后备字段并从中读取，就会得到过期数据。因此应始终使用 `GetValue` 和 `SetValue`。

### 对共享属性使用 AddOwner，而不是 OverrideMetadata

在 WPF 中，你可能会通过 `OverrideMetadata` 在子类中复用一个已有的 `DependencyProperty`，同时提供不同的 metadata。而在 Avalonia 中，如果你想让一个属性在无关类型之间共享，对应模式是使用 `AddOwner`：

```csharp
public static readonly StyledProperty<IBrush> BackgroundProperty =
    Border.BackgroundProperty.AddOwner<MyControl>();
```

这会把同一个属性注册到你的控件类型上，并且你还可以在注册时顺便覆盖默认值：

```csharp
public static readonly StyledProperty<IBrush> BackgroundProperty =
    Border.BackgroundProperty.AddOwner<MyControl>(
        new StyledPropertyMetadata<IBrush>(Brushes.Gray));
```

## 另请参阅

- [Avalonia 属性系统](/docs/properties)
- [值优先级](/docs/properties/value-precedence)
- [在自定义控件上定义属性](/docs/custom-controls/defining-properties)
