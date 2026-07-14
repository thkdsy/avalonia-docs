---
id: property-value-inheritance
title: 属性值继承
description: Avalonia 属性如何在可视树中从父元素向后代元素传播值，包括内置可继承属性和自定义可继承属性。
doc-type: concept
---

属性值继承允许设置在父元素上的属性值沿可视树传播给其后代，而无需每个后代都显式设置该值。这一机制常用于 `FontSize`、`FontFamily`、`Foreground` 和 `FlowDirection` 等属性。

## 工作原理

当某个 Avalonia 属性在注册时设置了 `inherits: true`，如果当前元素上没有本地值、样式值或动画值，属性系统就会沿可视树向上检查祖先元素。第一个拥有该属性值的祖先，会将该值作为继承值提供给当前元素。

在 [值优先级](/docs/properties/value-precedence) 系统中，继承值具有较低优先级（仅高于 `Unset`）。子元素上的本地值、样式值或动画值始终会覆盖继承值。

## 内置可继承属性

Avalonia 中有若干常用属性被注册为可继承属性：

| 属性 | 定义位置 | 效果 |
|---|---|---|
| `FontFamily` | [`TextElement`](/api/avalonia/controls/documents/textelement) | 文本控件从父元素继承字体族。 |
| `FontSize` | `TextElement` | 文本控件从父元素继承字号。 |
| `FontStyle` | `TextElement` | 文本控件继承字体样式（italic、normal）。 |
| `FontWeight` | `TextElement` | 文本控件继承字重（bold、normal）。 |
| `Foreground` | `TextElement` | 文本控件继承前景画刷。 |
| `LetterSpacing` | `TextElement` | 文本控件继承字符间距。 |
| `FlowDirection` | `Visual` | 控件继承从左到右或从右到左的布局方向。 |
| `DataContext` | `StyledElement` | 控件从父元素继承其数据上下文。 |
| `RequestedThemeVariant` | `ThemeVariantScope` | 控件继承请求的主题变体（亮色/暗色）。 |

## Example

在父元素上设置 `FontSize`，会把该值应用到所有没有自行设置 `FontSize` 的后代文本控件上：

```xml
<StackPanel FontSize="18">
    <!-- 继承 FontSize="18" -->
    <TextBlock Text="Large text" />

    <!-- 使用自己的 FontSize 覆盖 -->
    <TextBlock Text="Small text" FontSize="12" />

    <!-- 同样继承 FontSize="18" -->
    <Button Content="Large button text" />
</StackPanel>
```

## 创建可继承属性

如果你要创建一个可继承其值的自定义属性，请在注册时设置 `inherits: true`：

```csharp
public class MyControl : Control
{
    public static readonly StyledProperty<bool> IsCompactProperty =
        AvaloniaProperty.Register<MyControl, bool>(
            nameof(IsCompact),
            defaultValue: false,
            inherits: true);

    public bool IsCompact
    {
        get => GetValue(IsCompactProperty);
        set => SetValue(IsCompactProperty, value);
    }
}
```

这样，`MyControl` 的任何后代都可以读取 `IsCompact` 的值。如果后代本身也是 `MyControl`（或者已经为该属性添加了所有权），它就会自动接收继承值。

### 让后代类型也能使用该属性

如果不同类型的后代也要读取这个继承属性，就需要先为其注册所有权：

```csharp
public class MyChildControl : Control
{
    public static readonly StyledProperty<bool> IsCompactProperty =
        MyControl.IsCompactProperty.AddOwner<MyChildControl>();

    public bool IsCompact
    {
        get => GetValue(IsCompactProperty);
        set => SetValue(IsCompactProperty, value);
    }
}
```

## 继承与 `DataContext`

`DataContext` 是最重要的可继承属性之一。当你在 `Window` 上设置 `DataContext` 后，该窗口中的所有控件都会继承它：

```csharp
public partial class MainWindow : Window
{
    public MainWindow()
    {
        InitializeComponent();
        DataContext = new MainWindowViewModel();
    }
}
```

```xml
<!-- 窗口中的所有控件都会继承 DataContext -->
<Window>
    <StackPanel>
        <!-- 绑定到 MainWindowViewModel.Name -->
        <TextBlock Text="{Binding Name}" />

        <!-- 绑定到 MainWindowViewModel.Email -->
        <TextBox Text="{Binding Email}" />
    </StackPanel>
</Window>
```

## 另请参阅

- [属性系统概览](/docs/properties)：属性类型与注册方式概览。
- [值优先级](/docs/properties/value-precedence)：继承值在优先级顺序中的位置。
- [数据上下文](/docs/data-binding/data-context)：DataContext 这一可继承属性如何与数据绑定协同工作。
