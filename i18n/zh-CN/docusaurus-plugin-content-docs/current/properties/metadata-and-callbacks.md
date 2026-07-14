---
id: metadata-and-callbacks
title: 元数据与回调
---

每个 Avalonia 属性都带有关联的元数据，用于控制其默认值、绑定行为以及可选的值修正逻辑。你可以在注册属性时指定这些元数据，也可以为派生类型覆盖它们。

## 样式属性元数据

`StyledPropertyMetadata<T>` 类用于控制样式属性的行为：

| 参数 | 类型 | 说明 |
|---|---|---|
| `defaultValue` | `T` | 属性默认值；当没有其他值源提供值时使用。 |
| `defaultBindingMode` | `BindingMode` | 当绑定没有显式指定模式时使用的默认绑定模式。 |
| `coerce` | `Func<AvaloniaObject, T, T>?` | 在属性值应用前，用于调整或约束该值的回调。 |
| `enableDataValidation` | `bool` | 该属性是否参与数据验证。 |

## 默认值

在注册属性时指定默认值：

```csharp
public static readonly StyledProperty<double> OpacityProperty =
    AvaloniaProperty.Register<MyControl, double>(nameof(Opacity), defaultValue: 1.0);
```

### 覆盖默认值

派生控件可以修改继承而来的属性默认值：

```csharp
public class MySpecialButton : Button
{
    static MySpecialButton()
    {
        // 修改 MySpecialButton 的默认 Background
        BackgroundProperty.OverrideDefaultValue<MySpecialButton>(Brushes.LightBlue);
    }
}
```

在覆盖时，你也可以提供完整的元数据：

```csharp
static MySpecialButton()
{
    BackgroundProperty.OverrideMetadata<MySpecialButton>(
        new StyledPropertyMetadata<IBrush?>(Brushes.LightBlue));
}
```

:::caution
元数据覆盖必须在该类型的静态构造函数中注册。如果在该类型的任意实例已经创建之后再去覆盖元数据，将导致未定义行为。
:::

## 值修正

值修正回调会在属性值被存储前对其进行调整。这对于强制约束很有用，例如将一个数字限制在合法范围内。

```csharp
public static readonly StyledProperty<double> ProgressProperty =
    AvaloniaProperty.Register<MyControl, double>(
        nameof(Progress),
        defaultValue: 0.0,
        coerce: CoerceProgress);

private static double CoerceProgress(AvaloniaObject sender, double value)
{
    // 限制在 0 到 100 之间
    return Math.Clamp(value, 0.0, 100.0);
}

public double Progress
{
    get => GetValue(ProgressProperty);
    set => SetValue(ProgressProperty, value);
}
```

值修正回调会接收 `AvaloniaObject` 实例和待应用的值，并返回调整后的值。每当属性的最终有效值发生变化时，值修正都会执行，而不管该值来自哪里（本地值、样式、动画等）。

### 触发重新修正

如果你的值修正逻辑依赖于另一个属性，那么当那个属性变化时，可以触发重新修正：

```csharp
protected override void OnPropertyChanged(AvaloniaPropertyChangedEventArgs change)
{
    base.OnPropertyChanged(change);

    if (change.Property == MaximumProperty)
    {
        // 当 Maximum 变化时，重新修正 Progress
        CoerceValue(ProgressProperty);
    }
}
```

## 值验证

值验证回调会拒绝那些对该属性来说永远无效的值。与值修正不同，验证不会调整值；它只会返回 `true` 表示接受，或返回 `false` 表示拒绝。无效值会抛出异常。

```csharp
public static readonly StyledProperty<int> ColumnSpanProperty =
    AvaloniaProperty.Register<MyControl, int>(
        nameof(ColumnSpan),
        defaultValue: 1,
        validate: v => v > 0);

public int ColumnSpan
{
    get => GetValue(ColumnSpanProperty);
    set => SetValue(ColumnSpanProperty, value);
}
```

如果将 `ColumnSpan` 设为 `0` 或负数，就会抛出异常。

:::info
验证逻辑只能在注册时设置一次，不能针对不同类型单独覆盖。如果你需要按类型或实例来调整值，请使用值修正。
:::

## 响应属性变化

### 重写 `OnPropertyChanged`

在自定义控件中，响应属性变化最常见的方式如下：

```csharp
protected override void OnPropertyChanged(AvaloniaPropertyChangedEventArgs change)
{
    base.OnPropertyChanged(change);

    if (change.Property == IsExpandedProperty)
    {
        var wasExpanded = change.GetOldValue<bool>();
        var isExpanded = change.GetNewValue<bool>();
        UpdateVisualState(isExpanded);
    }
}
```

### `GetObservable`

从外部代码中订阅某个特定对象上的变化：

```csharp
myControl.GetObservable(MyControl.IsExpandedProperty)
    .Subscribe(isExpanded =>
    {
        Console.WriteLine($"IsExpanded is now {isExpanded}");
    });
```

### 类处理器

你可以注册一个会对某个类型的所有实例生效的处理器。这通常在静态构造函数中完成：

```csharp
static MyControl()
{
    IsExpandedProperty.Changed.AddClassHandler<MyControl>((control, args) =>
    {
        control.OnIsExpandedChanged(args);
    });
}

private void OnIsExpandedChanged(AvaloniaPropertyChangedEventArgs args)
{
    // 处理变化
}
```

## 直接属性元数据

直接属性使用 `DirectPropertyMetadata<T>`：

| 参数 | 类型 | 说明 |
|---|---|---|
| `unsetValue` | `T` | 清除属性时所使用的值；它相当于直接属性的实际默认值。 |
| `defaultBindingMode` | `BindingMode` | 默认绑定模式。 |
| `enableDataValidation` | `bool` | 该属性是否参与数据验证。 |

直接属性不支持通过元数据来实现值修正或值验证。此类检查应直接写在 CLR 属性 setter 中：

```csharp
private int _retryCount;

public int RetryCount
{
    get => _retryCount;
    set
    {
        if (value < 0)
            throw new ArgumentOutOfRangeException(nameof(value));
        SetAndRaise(RetryCountProperty, ref _retryCount, value);
    }
}
```

## 另请参阅

- [属性系统概览](/docs/properties)：属性类型和注册方式概览。
- [值优先级](/docs/properties/value-precedence)：属性系统如何从多个值源中解析最终值。
- [属性值继承](/docs/properties/property-value-inheritance)：值如何沿树向下传播。
