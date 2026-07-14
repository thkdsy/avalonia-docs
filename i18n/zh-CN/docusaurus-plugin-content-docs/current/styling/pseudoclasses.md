---
id: pseudoclasses
title: 伪类
description: 了解如何在 Avalonia 中使用伪类，根据焦点、悬停、按下等控件状态应用样式。
doc-type: concept
---

import CustomPseudoclassScreenshot from '/img/reference/styles/custom-pseudoclass.gif';

Avalonia 中的*伪类*与 CSS 中的伪类类似，它们是由 `Control` 暴露出的关键字，用来表示控件的某种特定状态。这些状态会在 [style selectors](/docs/styling/style-selectors) 中使用，以便有条件地为控件应用样式。例如，[`Button`](/api/avalonia/controls/button) 在被按下时可以显示不同外观，而 `TextBox` 在被禁用时也可以显示另一种外观。

伪类状态由 `Control` 的 `PseudoClasses` 属性进行跟踪。按照约定，伪类名称都以 `:` 开头，例如 `:pointerover` 或 `:pressed`。

## 常见伪类

这些伪类由 `InputElement` 定义，因此每个 `Control` 都可以使用：

| 伪类 | 说明 |
|:-----------------|----------------------------------------------------------------|
| `:disabled` | 控件已禁用，无法交互。 |
| `:pointerover` | 指针经过命中测试后位于控件上方。 |
| `:focus` | 控件拥有焦点。 |
| `:focus-within` | 控件自身拥有焦点，或其某个后代元素拥有焦点。 |
| `:focus-visible` | 控件拥有焦点，并且应显示可视化焦点提示。 |

各个控件还会定义一些与自身状态相关的额外伪类。例如，`CheckBox` 会暴露 `:checked`，而 `Button` 会暴露 `:pressed`。

## 在选择器中使用伪类

只需将伪类追加到选择器后面，就可以匹配该伪类。下面的示例会在 `CheckBox` 处于选中状态时，将文本设置为粗体：

```xml
<Window.Styles>
    <Style Selector="CheckBox:checked">
        <Setter Property="FontWeight" Value="Bold" />
    </Style>
</Window.Styles>

<CheckBox Content="Pseudoselectors" />
```

一个控件可以同时激活多个伪类，你也可以在同一个选择器中同时匹配多个伪类：

```xml
<Style Selector="Button.red:focus:pointerover">
```

这个选择器会匹配带有 `red` 样式类，并且同时激活了 `:focus` 与 `:pointerover` 伪类的 `Button` 控件。

## 创建自定义伪类

在创建自定义控件时，你可以定义自定义伪类来暴露控件状态。可以为类添加 `[PseudoClasses]` 特性以获得 IDE 支持，并通过 `PseudoClasses.Set` 来切换状态。

:::note
`PseudoClasses` 集合是一个 `protected` 属性。自定义伪类只能在控件类内部设置，因此它们必须通过继承来实现。
:::

下面的示例定义了一个 `Button` 子类，它会根据指针位于按钮的哪个区域来设置不同的伪类。

```csharp
[PseudoClasses(":left", ":right", ":middle")]
public class AreaButton : Button
{
    protected override void OnPointerMoved(PointerEventArgs e)
    {
        base.OnPointerMoved(e);
        var pos = e.GetPosition(this);

        if (pos.X < Bounds.Width * 0.25)
            SetAreaPseudoclasses(true, false, false);
        else if (pos.X > Bounds.Width * 0.75)
            SetAreaPseudoclasses(false, true, false);
        else
            SetAreaPseudoclasses(false, false, true);
    }

    protected override void OnPointerExited(PointerEventArgs e)
    {
        base.OnPointerExited(e);
        SetAreaPseudoclasses(false, false, false);
    }

    private void SetAreaPseudoclasses(bool left, bool right, bool middle)
    {
        PseudoClasses.Set(":left", left);
        PseudoClasses.Set(":right", right);
        PseudoClasses.Set(":middle", middle);
    }
}
```

由于 `AreaButton` 派生自 `Button`（即 `TemplatedControl`），因此它需要自己的 `ControlTheme`，这样选择器才能针对这些新伪类进行匹配：

```xml
<ControlTheme
    x:Key="{x:Type local:AreaButton}"
    BasedOn="{StaticResource {x:Type Button}}"
    TargetType="local:AreaButton" />
```

定义好控件主题后，就可以使用嵌套选择器来为这些自定义伪类设置样式：

```xml
<Window.Styles>
    <Style Selector="local|AreaButton">
        <Setter Property="Content" Value="Testing Area" />
        <Setter Property="MinWidth" Value="200" />

        <Style Selector="^:left">
            <Setter Property="Content" Value="Left" />
        </Style>
        <Style Selector="^:right">
            <Setter Property="Content" Value="Right" />
        </Style>
        <Style Selector="^:middle">
            <Setter Property="Content" Value="Middle" />
        </Style>
    </Style>
</Window.Styles>

<local:AreaButton />
```

<Image light={CustomPseudoclassScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

自定义控件会自动继承其基类上的伪类，因此 `AreaButton` 也会响应 `InputElement` 提供的 `:pointerover`、`:focus` 以及其他内置伪类。

## 另请参阅

- [Style selectors](/docs/styling/style-selectors)
- [Style selector syntax](/docs/styling/style-selector-syntax)
- [Control themes](/docs/styling/control-themes)
