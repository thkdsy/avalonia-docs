---
id: style-classes
title: 样式类
description: 了解如何在 Avalonia 中分配和使用样式类，以便按条件为控件应用样式。
doc-type: concept
---

你可以为一个 Avalonia 控件分配一个或多个*样式类*，并用这些类来辅助样式选择。样式类通过控件元素上的 `Classes` 属性来设置。如果要分配多个类，请使用空格分隔。

例如，下面这个按钮同时应用了 `h1` 和 `blue` 两个样式类：

```xml
<Button Classes="h1 blue"/>
```

## 伪类

和 CSS 一样，控件也可以拥有伪类；这些类并不是由用户定义，而是由控件本身在运行状态中提供。在选择器中，伪类名称始终以冒号开头。

例如，`:pointerover` 伪类表示当前指针正位于某个控件之上（即进入了它的边界范围内）。这个伪类与 CSS 中的 `:hover` 类似。

下面是一个 `:pointerover` 伪类选择器的示例：

<XamlPreview>

```xml
<UserControl xmlns="https://github.com/avaloniaui">
  <StackPanel>
    <StackPanel.Styles>
      <Style Selector="Border:pointerover">
        <Setter Property="Background" Value="Red"/>
      </Style>
    </StackPanel.Styles>
    <Border>
      <TextBlock Margin="10">Hover for red background</TextBlock>
    </Border>
  </StackPanel>
</UserControl>
```

</XamlPreview>

下面这个示例中，伪类选择器会修改控件模板内部元素的属性：

<XamlPreview>

```xml
<UserControl xmlns="https://github.com/avaloniaui">
  <StackPanel>
    <StackPanel.Styles>
      <Style Selector="Button:pressed /template/ ContentPresenter">
          <Setter Property="TextBlock.Foreground" Value="Red"/>
      </Style>
    </StackPanel.Styles>
   <Button Margin="10">Press for red text</Button>
  </StackPanel>
</UserControl>
```

</XamlPreview>

其他常见伪类还包括 `:focus`、`:disabled`、按钮的 `:pressed`，以及复选框的 `:checked`。

:::info
有关伪类的更多细节，请参阅 [Pseudoclasses](/docs/styling/pseudoclasses)。
:::

## 条件样式类

如果你希望根据绑定条件添加或移除某个样式类，可以使用下面这种特殊语法：

```xml
<Button Classes.accent="{Binding IsSpecial}" />
```

## 条件样式模式

Avalonia 没有 WPF 风格的 triggers。要实现条件样式，应改用样式类、伪类和绑定转换器的组合方式。

### 根据绑定属性切换外观

先为每种状态定义不同的样式类，再通过绑定按条件应用它们：

```xml
<StackPanel>
    <StackPanel.Styles>
        <Style Selector="Border.status-ok">
            <Setter Property="Background" Value="Green" />
        </Style>
        <Style Selector="Border.status-error">
            <Setter Property="Background" Value="Red" />
        </Style>
    </StackPanel.Styles>

    <Border Classes.status-ok="{Binding IsOnline}"
            Classes.status-error="{Binding !IsOnline}"
            Padding="8">
        <TextBlock Text="Service Status" />
    </Border>
</StackPanel>
```

### 对非布尔条件使用转换器

如果条件不是简单的布尔值，那么可以使用值转换器：

```xml
<Border Background="{Binding Priority, Converter={StaticResource PriorityToBrushConverter}}" />
```

### 将伪类与样式类组合使用

对已添加样式类的控件，再进一步针对特定交互状态进行样式设置：

```xml
<StackPanel.Styles>
    <Style Selector="Button.primary">
        <Setter Property="Background" Value="Blue" />
        <Setter Property="Foreground" Value="White" />
    </Style>
    <Style Selector="Button.primary:pointerover">
        <Setter Property="Background" Value="DarkBlue" />
    </Style>
    <Style Selector="Button.primary:pressed">
        <Setter Property="Background" Value="Navy" />
    </Style>
</StackPanel.Styles>
```

### 在自定义控件中定义伪类

你可以为控件特有的状态定义自定义伪类：

```csharp
public class StatusIndicator : TemplatedControl
{
    public static readonly StyledProperty<bool> IsActiveProperty =
        AvaloniaProperty.Register<StatusIndicator, bool>(nameof(IsActive));

    public bool IsActive
    {
        get => GetValue(IsActiveProperty);
        set => SetValue(IsActiveProperty, value);
    }

    protected override void OnPropertyChanged(AvaloniaPropertyChangedEventArgs change)
    {
        base.OnPropertyChanged(change);
        if (change.Property == IsActiveProperty)
        {
            PseudoClasses.Set(":active", change.GetNewValue<bool>());
        }
    }
}
```

然后再通过伪类选择器为它编写样式：

```xml
<Style Selector="local|StatusIndicator:active">
    <Setter Property="Background" Value="LimeGreen" />
</Style>
```

## 在代码中操作样式类

你可以在代码中通过 `Classes` 集合来操作样式类：

```csharp
control.Classes.Add("blue");
control.Classes.Remove("red");
control.Classes.Toggle("highlight");

// 检查某个类是否存在
if (control.Classes.Contains("blue"))
{
    // ...
}
```

## 另请参阅

- [Styles](/docs/styling/styles)
- [Pseudoclasses](/docs/styling/pseudoclasses)
- [Style selectors](/docs/styling/style-selectors)
