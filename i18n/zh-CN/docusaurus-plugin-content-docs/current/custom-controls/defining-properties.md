---
id: defining-properties
title: 定义属性
description: 在自定义 Avalonia 控件上定义样式属性、直接属性和附加属性。
doc-type: how-to
---

import DefiningPropertyPreviewScreenshot from '/img/guides/ui-development/custom-controls/defining-property-preview.png';

## 为自定义控件定义属性

如果你正在创建自定义控件，通常会希望它具有可由 Avalonia 样式系统设置、可进行数据绑定，或可在 XAML 中配置的属性。为此，Avalonia 提供了三种属性类型：样式属性、直接属性和附加属性。

本页将带你了解每种属性类型的注册与使用方式，以便你为控件选择合适的方案。

:::info
有关如何在 Avalonia 中使用样式的更多信息，请参阅[样式指南](/docs/styling/styles)。
:::

## 样式属性

样式属性是最常见的属性类型。它会将值存储在 Avalonia 属性系统内部（而不是存储在后备字段中），这意味着它能够参与样式、动画和属性值优先级机制。当你希望用户能够对某个属性应用样式或动画时，应使用样式属性。

注册样式属性通常分为两步：

1. 使用 `AvaloniaProperty.Register` 将属性注册为 `static readonly` 字段。
2. 提供调用 `GetValue` 和 `SetValue` 的 CLR getter 和 setter。

### 命名约定

静态字段必须遵循 `PropertyNameProperty` 这一命名模式。Avalonia 会利用这一约定自动将 XAML 属性映射到对应属性上：

```csharp
public static readonly StyledProperty<double> CornerRadiusProperty = ...
```

这样你就可以在 XAML 中这样写：

```xml
<local:MyControl CornerRadius="8" />
```

### 注册新的样式属性

下面的示例注册了一个默认值为 `0.0` 的 `CornerRadius` 样式属性：

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

`Register` 方法支持多个可选参数：

| 参数 | 说明 |
|---|---|
| `name` | 属性名称，必须与 CLR 属性名一致。 |
| `defaultValue` | 属性的默认值。 |
| `inherits` | 该值是否沿可视树向下继承。 |
| `defaultBindingMode` | 默认绑定模式（`OneWay`、`TwoWay`、`OneTime`、`OneWayToSource`）。 |
| `validate` | 一个函数，对应拒绝的值应返回 `false`。 |
| `coerce` | 在应用值之前对其进行调整的函数。 |

### 复用现有样式属性

如果另一个控件已经定义了你需要的属性（例如 `Border` 上的 `Background`），请使用 `AddOwner` 而不是重新注册一个新的属性。这样可以确保属性身份保持一致，从而使针对该属性的样式能在所有共享它的控件之间正常工作：

```csharp
public class MyCustomControl : Control
{
    public static readonly StyledProperty<IBrush?> BackgroundProperty =
        Border.BackgroundProperty.AddOwner<MyCustomControl>();

    public IBrush? Background
    {
        get => GetValue(BackgroundProperty);
        set => SetValue(BackgroundProperty, value);
    }
}
```

### 为自定义属性设置样式

注册完样式属性后，用户就可以在样式中针对它进行设置。下面的示例通过样式设置自定义控件的 `Background`：

```xml title='MainWindow.axaml'
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:cc="using:AvaloniaCCExample.CustomControls"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        mc:Ignorable="d" d:DesignWidth="800" d:DesignHeight="450"
        x:Class="AvaloniaCCExample.MainWindow"
        Title="Avalonia Custom Control">

  <Window.Styles>
    <Style Selector="cc|MyCustomControl">
      <Setter Property="Background" Value="Yellow"/>
    </Style>
  </Window.Styles>

  <cc:MyCustomControl Height="200" Width="300"/>

</Window>
```

```csharp title='MyCustomControl.cs'
using Avalonia;
using Avalonia.Controls;
using Avalonia.Media;

namespace AvaloniaCCExample.CustomControls
{
    public class MyCustomControl : Control
    {
        public static readonly StyledProperty<IBrush?> BackgroundProperty =
            Border.BackgroundProperty.AddOwner<MyCustomControl>();

        public IBrush? Background
        {
            get { return GetValue(BackgroundProperty); }
            set { SetValue(BackgroundProperty, value); }
        }

        public sealed override void Render(DrawingContext context)
        {
            if (Background != null)
            {
                var renderSize = Bounds.Size;
                context.FillRectangle(Background, new Rect(renderSize));
            }
            base.Render(context);
        }
    }
}
```

样式属性既可在运行时生效，也可在预览面板中生效。

<Image light={DefiningPropertyPreviewScreenshot} alt="Preview of a custom control with a defined property" position="center" maxWidth={400} cornerRadius="true"/>

## 直接属性

`DirectProperty` 由传统的 C# 字段提供后备存储。它不参与样式或动画，但支持数据绑定和变更通知。以下情况适合使用直接属性：

- 你需要一个**只读**属性（样式属性不能是只读的）。
- 你希望获得**更好的性能**，因为值会直接从字段中读取。
- 该属性**不需要参与样式**（例如 `ItemsControl` 上的 `Items`）。

### 注册直接属性

使用 `AvaloniaProperty.RegisterDirect`，并提供指向后备字段的 getter 和 setter 委托：

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

:::warning
在 CLR setter 中一定要使用 `SetAndRaise`，而不是直接给后备字段赋值。`SetAndRaise` 会在一次调用中同时更新字段并触发属性变更通知。对直接属性调用 `SetValue` 会抛出异常。
:::

### 只读直接属性

若要创建只读属性，请在注册调用中省略 setter 委托，并将 CLR setter 保持为 `private`：

```csharp
public class MyControl : Control
{
    public static readonly DirectProperty<MyControl, bool> IsActiveProperty =
        AvaloniaProperty.RegisterDirect<MyControl, bool>(
            nameof(IsActive),
            o => o.IsActive);

    private bool _isActive;

    public bool IsActive
    {
        get => _isActive;
        private set => SetAndRaise(IsActiveProperty, ref _isActive, value);
    }
}
```

### 样式属性与直接属性对比

| 行为 | 样式属性 | 直接属性 |
|---|---|---|
| 参与样式 | 是 | 否 |
| 参与动画 | 是 | 否 |
| 支持值优先级 | 是 | 否（单一值） |
| 可继承值 | 是 | 否 |
| 支持强制修正 | 是 | 否 |
| 性能 | 属性存储查找 | 直接字段访问 |
| 可否只读 | 否 | 是 |

## 响应属性变化

你可以通过在控件中重写 `OnPropertyChanged` 来响应属性值变化：

```csharp
protected override void OnPropertyChanged(AvaloniaPropertyChangedEventArgs change)
{
    base.OnPropertyChanged(change);

    if (change.Property == BackgroundProperty)
    {
        // 使视觉失效，从而让控件使用新背景重新绘制。
        InvalidateVisual();
    }
}
```

这种方式同时适用于样式属性和直接属性。

## 常见陷阱

- **名称不匹配。** 传给 `Register` 的 `name` 参数必须与 CLR 属性名完全一致。名称不匹配会在运行时导致绑定和 XAML 错误。
- **在直接属性上使用 `SetValue`。** 直接属性必须使用 `SetAndRaise`。调用 `SetValue` 会抛出 `InvalidOperationException`。
- **为样式属性添加后备字段。** 样式属性的值存储在 Avalonia 属性系统中。如果你从本地字段读取，会得到过期数据。应始终使用 `GetValue` 和 `SetValue`。
- **忘记调用 `base.OnPropertyChanged`。** 如果你重写了 `OnPropertyChanged`，应始终先调用基类实现，以便框架处理该变化。

## 另请参阅

- [Avalonia 属性系统](/docs/properties)：样式属性、直接属性和附加属性的完整参考。
- [值优先级](/docs/properties/value-precedence)：Avalonia 如何解析相互竞争的属性值。
- [元数据与回调](/docs/properties/metadata-and-callbacks)：默认值、强制修正和验证。
- [定义事件](/docs/custom-controls/defining-events)：为自定义控件添加路由事件。
- [Attached properties](/docs/custom-controls/attached-properties): Create properties that can be set on other controls.
- [Custom control class](/docs/custom-controls/custom-control-class): Base class overview for custom controls.
