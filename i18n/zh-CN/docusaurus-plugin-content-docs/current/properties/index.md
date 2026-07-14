---
id: index
title: Avalonia 属性系统
---

Avalonia 拥有自己的属性系统，它扩展了标准的 .NET 属性模型。Avalonia 属性支持样式、数据绑定、动画、属性值继承和变更通知。理解这一属性系统，是创建自定义控件并高效使用框架的基础。

## 属性类型

Avalonia 定义了三种属性类型，分别适用于不同场景：

| 属性类型 | 基类 | 使用场景 |
|---|---|---|
| **样式属性** | `StyledProperty<T>` | 参与样式系统的属性。这是最常见的类型。 |
| **直接属性** | `DirectProperty<TOwner, TValue>` | 由传统 C# 字段支撑，并暴露给 Avalonia 属性系统以支持绑定。适用于性能敏感或只读属性。 |
| **附加属性** | `AttachedProperty<T>` | 可设置在任意 [`AvaloniaObject`](/api/avalonia/avaloniaobject) 上的属性，通常由布局面板使用（例如 `Grid.Row`、`DockPanel.Dock`）。 |

## 样式属性

`StyledProperty` 是 Avalonia 中的标准属性类型。样式属性的值存储在 Avalonia 属性系统中，而不是后备字段里，因此它们可以参与样式、动画和值优先级计算。

### 注册样式属性

```csharp
public class MyControl : Control
{
    public static readonly StyledProperty<double> CornerRadiusProperty =
        AvaloniaProperty.Register<MyControl, double>(nameof(CornerRadius), defaultValue: 0.0);

    public double CornerRadius
    {
        get => GetValue(CornerRadiusProperty);
        set => SetValue(CornerRadiusProperty, value);
    }
}
```

`Register` 方法接受以下参数：

| 参数 | 说明 |
|---|---|
| `name` | 属性名称，必须与 CLR 属性名一致。 |
| `defaultValue` | 属性的默认值。 |
| `inherits` | 该属性值是否沿可视树向下继承。 |
| `defaultBindingMode` | 默认绑定模式（`OneWay`、`TwoWay`、`OneTime`、`OneWayToSource`）。 |
| `validate` | 一个验证函数；对永远非法的值返回 `false`。 |
| `coerce` | 一个在应用前调整值的函数（参见 [元数据与回调](/docs/properties/metadata-and-callbacks)）。 |

### 复用已有属性

如果另一个控件已经定义了你需要的属性，请使用 `AddOwner`，而不是重新注册一个新属性：

```csharp
public class MyControl : Control
{
    public static readonly StyledProperty<IBrush?> BackgroundProperty =
        Border.BackgroundProperty.AddOwner<MyControl>();

    public IBrush? Background
    {
        get => GetValue(BackgroundProperty);
        set => SetValue(BackgroundProperty, value);
    }
}
```

这样可以确保属性标识保持唯一，因此针对 `Background` 的样式能够在所有共享该属性的控件之间一致生效。

## 直接属性

`DirectProperty` 由普通 C# 字段支撑。Avalonia 属性系统会通过你提供的 getter 和 setter 委托来读写它。直接属性适用于以下情况：

- 你需要一个不参与样式系统的属性（例如 `ItemsControl` 上的 `Items`）。
- 你需要一个只读属性。
- 你希望避免样式属性值存储机制带来的额外开销。

### 注册直接属性

```csharp
public class MyControl : Control
{
    public static readonly DirectProperty<MyControl, string?> StatusProperty =
        AvaloniaProperty.RegisterDirect<MyControl, string?>(
            nameof(Status),
            o => o.Status,
            (o, v) => o.Status = v);

    private string? _status;

    public string? Status
    {
        get => _status;
        set => SetAndRaise(StatusProperty, ref _status, value);
    }
}
```

:::info
请在 setter 中使用 `SetAndRaise`，而不是直接为字段赋值。这个方法会在一次调用中同时更新后备字段并触发属性变更通知。
:::

### 只读直接属性

省略 setter 委托即可创建只读属性：

```csharp
public static readonly DirectProperty<MyControl, bool> IsActiveProperty =
    AvaloniaProperty.RegisterDirect<MyControl, bool>(
        nameof(IsActive),
        o => o.IsActive);
```

### 与样式属性的关键区别

| 行为 | 样式属性 | 直接属性 |
|---|---|---|
| 参与样式系统 | 是 | 否 |
| 参与动画 | 是 | 否 |
| 支持值优先级 | 是 | 否（只有单一值） |
| 可以继承值 | 是 | 否 |
| 支持值修正 | 是 | 否 |
| 性能 | 属性存储查找 | 直接字段访问 |
| 是否可只读 | 否 | 是 |

## 附加属性

`AttachedProperty` 是一种可设置在任意 `AvaloniaObject` 上的样式属性。附加属性通常由父级布局面板定义，并设置在其子元素上。

### 注册附加属性

```csharp
public class MyPanel : Panel
{
    public static readonly AttachedProperty<int> ColumnProperty =
        AvaloniaProperty.RegisterAttached<MyPanel, Control, int>("Column", defaultValue: 0);

    public static int GetColumn(Control element) => element.GetValue(ColumnProperty);
    public static void SetColumn(Control element, int value) => element.SetValue(ColumnProperty, value);
}
```

### 在 XAML 中使用附加属性

```xml
<local:MyPanel>
    <Button local:MyPanel.Column="1" Content="In column 1" />
</local:MyPanel>
```

## 获取和设置值

所有 Avalonia 属性都通过 `AvaloniaObject` 基类进行读写：

```csharp
// 获取属性值
double radius = myControl.GetValue(MyControl.CornerRadiusProperty);

// 设置属性值
myControl.SetValue(MyControl.CornerRadiusProperty, 8.0);

// 清除属性值（回退到默认值或样式值）
myControl.ClearValue(MyControl.CornerRadiusProperty);
```

## 观察属性变化

你可以观察某个特定对象上的属性变化：

```csharp
// 使用 GetObservable 订阅变化
myControl.GetObservable(MyControl.CornerRadiusProperty)
    .Subscribe(newValue => Console.WriteLine($"CornerRadius changed to {newValue}"));
```

或者在你的控件类中重写 `OnPropertyChanged`：

```csharp
protected override void OnPropertyChanged(AvaloniaPropertyChangedEventArgs change)
{
    base.OnPropertyChanged(change);

    if (change.Property == CornerRadiusProperty)
    {
        var oldValue = change.GetOldValue<double>();
        var newValue = change.GetNewValue<double>();
        // 响应该变化
    }
}
```

## 另请参阅

- [值优先级](/docs/properties/value-precedence)：了解 Avalonia 如何在样式、动画和本地值之间解析竞争的属性值。
- [元数据与回调](/docs/properties/metadata-and-callbacks)：了解默认值、值修正和验证机制。
- [属性值继承](/docs/properties/property-value-inheritance)：了解属性如何从祖先控件继承值。
- [定义属性](/docs/custom-controls/defining-properties)：为自定义控件添加属性的实用指南。
- [附加属性](/docs/custom-controls/attached-properties)：创建附加属性的实用指南。
