---
id: styles
title: 样式
---

## 选择器没有目标

像 CSS 选择器一样，_Avalonia UI_ 选择器在没有任何可匹配的控件时，不会抛出错误或警告。样式会静默失败，不会显示。

:::info
检查是否使用了不存在的名称或类。
:::

:::info
检查是否使用了子选择器，而实际上没有可匹配的子元素。
:::

## 包含文件顺序

样式按照声明顺序应用。如果包含了多个针对同一控件属性的样式文件，则最后包含的样式会覆盖前面的样式。例如：

```xml title="Style1.axaml"
<Style Selector="TextBlock.header">
    <Setter Property="Foreground" Value="Green" />
</Style>
```

```xml title="Style2.axaml"
<Style Selector="TextBlock.header">
    <Setter Property="Foreground" Value="Blue" />
    <Setter Property="FontSize" Value="16" />
</Style>
```

```xml
<StyleInclude Source="Style1.axaml" />
<StyleInclude Source="Style2.axaml" />
```

这里来自文件 **Styles1.axaml** 的样式先被应用，因此文件 **Styles2.axaml** 中样式的 setter 具有优先级。最终得到的 TextBlock 将具有 FontSize="16" 和 Foreground="Blue"。同样的顺序优先级规则也适用于样式文件内部。

## 本地设置的属性具有优先级

直接在控件上定义的本地值通常比任何样式值具有更高的优先级。因此在这个例子中，文本块将具有红色前景：

```xml
<Style Selector="TextBlock.header">
    <Setter Property="Foreground" Value="Green" />
</Style>
...
<TextBlock Classes="header" Foreground="Red" />
```

你可以在 `BindingPriority` 枚举中查看完整的值优先级列表，其中枚举值越小，优先级越高。

| BindingPriority | Value      | Comment                                              |
|-----------------|------------|------------------------------------------------------|
| `Animation`     | -1         | The highest priority - even overrides a local value  |
| `LocalValue`    | 0          | A local value is set on the property of the control. |
| `StyleTrigger`  | 1          | This is triggered when a style becomes active.       |
| `Template`      | 2          |                                                      |
| `Style`         | 3          |                                                      |
| `Inherited`     | 4          | Value inherited from a parent control.               |
| `Unset`         | 2147483647 |                                                      |

:::caution
例外是 `Animation` 值拥有最高优先级，甚至可以覆盖本地值。
:::

:::info
某些默认的 _Avalonia UI_ 样式在其模板中使用本地值，而不是模板绑定或样式 setter。这使得在不替换整个模板的情况下，无法更新模板属性。
:::

### 缺少样式伪类（触发器）选择器

假设有这样一种情况：你可能会期望第二个样式覆盖前一个样式，但实际上并没有：

```xml
<Style Selector="Border:pointerover">
    <Setter Property="Background" Value="Blue" />
</Style>
<Style Selector="Border">
    <Setter Property="Background" Value="Red" />
</Style>
...
<Border Width="100" Height="100" Margin="100" />
```

在这个代码示例中，`Border` 平时具有红色背景，当指针悬停其上时则变为蓝色。这是因为与 CSS 一样，更具体的选择器具有优先级。当你想用一个样式覆盖任意状态（pointerover、pressed 或其他状态）的默认样式时，这就会成为一个问题。要实现这一点，你还需要为这些状态创建新的样式。

:::info
当出现这种情况时，请访问 Avalonia 源代码，查找[原始模板](https://github.com/AvaloniaUI/Avalonia/tree/master/src/Avalonia.Themes.Fluent/Controls)，并将带有伪类的样式复制到你的代码中。
:::

### 带伪类的选择器不会覆盖默认样式

下面这段样式代码可能会被认为可以覆盖默认样式并正常工作：

```xml
<Style Selector="Button">
    <Setter Property="Background" Value="Red" />
</Style>
<Style Selector="Button:pointerover">
    <Setter Property="Background" Value="Blue" />
</Style>
```

你可能会期望 `Button` 默认是红色，而在指针悬停时变为蓝色。实际上，只有第一个样式的 setter 会被应用，第二个会被忽略。

原因隐藏在 Button 的模板中。你可以在 Avalonia 源代码中找到默认模板（[Simple](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Themes.Simple/Controls/Button.xaml) 主题和 [Fluent](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Themes.Fluent/Controls/Button.xaml) 主题），不过为了方便，这里我们将 Fluent 主题中的模板简化如下：

```xml
<Style Selector="Button">
    <Setter Property="Background" Value="{DynamicResource ButtonBackground}"/>
    <Setter Property="Template">
        <ControlTemplate>
            <ContentPresenter Name="PART_ContentPresenter"
                              Background="{TemplateBinding Background}"
                              Content="{TemplateBinding Content}"/>
        </ControlTemplate>
    </Setter>
</Style>
<Style Selector="Button:pointerover /template/ ContentPresenter#PART_ContentPresenter">
    <Setter Property="Background" Value="{DynamicResource ButtonBackgroundPointerOver}" />
</Style>
```

实际的背景是由 `ContentPresenter` 渲染的，默认情况下它绑定到按钮的 `Background` 属性。然而在 pointer-over 状态下，选择器是直接将背景应用到 `ContentPresenter`（`Button:pointerover /template/ ContentPresenter#PART_ContentPresenter`）上。这就是为什么在前面的代码示例中我们的 setter 会被忽略。修正后的代码也应该直接针对 content presenter：

```xml
<!-- 这里 #PART_ContentPresenter 名称选择器不是必需的，但为了让样式更具体而添加 -->
<Style Selector="Button:pointerover /template/ ContentPresenter#PART_ContentPresenter">
    <Setter Property="Background" Value="Blue" />
</Style>
```

:::info
你可以在默认主题（Simple 和 Fluent）中的所有控件上看到这种行为，而不仅仅是 Button。并且不仅仅是 Background，其他依赖状态的属性也一样。
:::

:::info
为什么默认样式会直接更改 `ContentPresenter` 的 `Background` 属性，而不是更改 `Button.Background` 属性？

这是因为如果用户给按钮设置了本地值，它会覆盖所有样式，并让按钮始终保持相同的颜色。更多细节请参见这个[已回退的 PR](https://github.com/AvaloniaUI/Avalonia/pull/2662#issuecomment-515764732)。
:::

### 当样式不再应用时，特定属性的先前值不会被恢复

在 Avalonia 中，我们有多种属性类型，其中一种是 Direct Property，它根本不支持样式。这些属性采用简化方式工作，以实现更低的开销和更高的性能，并且不会根据优先级存储多个值。相反，只会保存最新值，并且无法恢复。你可以在[定义属性](/docs/custom-controls/defining-properties)指南中找到更多细节。

典型示例是 [CommandProperty](https://api-docs.avaloniaui.net/docs/P_Avalonia_Controls_Button_Command)。它被定义为 DirectProperty，永远不会正常工作。将来，尝试对 direct property 进行样式设置将会导致编译时错误，参见 [#6837](https://github.com/AvaloniaUI/Avalonia/issues/6837)。