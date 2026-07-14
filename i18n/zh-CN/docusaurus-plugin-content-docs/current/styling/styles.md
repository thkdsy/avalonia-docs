---
id: styles
title: 样式
description: 了解如何在 Avalonia 中使用样式、选择器和 Setter，在多个控件之间共享属性设置。
doc-type: concept
---

Avalonia 的样式系统是一种在多个控件之间共享属性设置的机制。
Avalonia 提供了三种主要的控件样式机制：

## 样式

- [Styles](/docs/styling/styles) 类似于 CSS 样式，通常用于根据控件在应用中的内容或用途来设置外观；例如为标题文本块创建样式。

## 控件主题

- [Control themes](/docs/styling/control-themes) 类似于 WPF/UWP 中的样式，通常用于为控件应用主题。

## 容器查询
- [Container queries](/docs/styling/container-queries) 是一组会根据容器尺寸条件应用的样式集合。

## 工作原理

从本质上说，样式机制包含两个步骤：选择和替换。你可以在 XAML 中定义这两个步骤的工作方式，不过通常会通过在控件元素上定义“类”标签来辅助选择过程。

:::info
Avalonia 样式系统中在控件元素上使用“类”标签的方式，与 CSS（层叠样式表）在 HTML 元素上的工作方式是类似的。
:::

样式系统在选择阶段会从某个控件出发，沿着[逻辑树](/docs/custom-controls/control-trees)向上查找，从而实现层叠样式。这意味着：定义在应用最高层级（即 `App.axaml` 文件）中的样式可以在任何位置使用，但仍然可能被距离控件更近的位置覆盖（例如窗口或用户控件内部）。

当选择阶段找到匹配项后，匹配控件的属性就会按照该样式中的 setter 进行修改。

## 样式的写法

一个 XAML 样式由两部分组成：一个 selector 属性，以及一个或多个 setter 元素。selector 的值是一个使用 Avalonia **样式选择器语法**的字符串。每个 setter 元素通过属性名指定要修改的属性，并给出替换后的新值。整体模式如下：

```xml
<Style Selector="selector syntax">
     <Setter Property="property name" Value="new value"/>
     ...
</Style>
```

:::info
Avalonia 的**样式选择器语法**与 CSS（层叠样式表）中使用的选择器语法是类似的。详细参考请参阅 [style selector syntax](/docs/styling/style-selector-syntax)。
:::

## 示例

下面这个示例展示了样式如何编写并应用到控件元素上，其中借助了 [style class](/docs/styling/style-classes) 来辅助选择：

```xml
<Window ... >
    <Window.Styles>
        <Style Selector="TextBlock.h1">
            <Setter Property="FontSize" Value="24"/>
            <Setter Property="FontWeight" Value="Bold"/>
        </Style>
    </Window.Styles>
    <StackPanel Margin="20">
       <TextBlock Classes="h1">Heading 1</TextBlock>
    </StackPanel>
</Window>
```

在这个示例中，所有带有 `h1` 样式类的 `TextBlock` 元素，都会以样式中指定的字体大小和字重显示。

## 样式放在哪里

你可以把样式放在 `Control` 或 `Application` 上的 `Styles` 集合元素中。例如，窗口的样式集合写法如下：

```xml
<Window.Styles>
   <Style> ...  </Style>
</Window.Styles>
```

样式集合所在的位置决定了其内部样式的作用域。在上面的示例中，这些样式会应用到该窗口及其内部所有内容上。如果把样式加到 `Application` 上，那么它就会全局生效。

## 选择器

样式选择器用于定义样式应作用于哪些控件。选择器支持多种格式，其中最简单的一种是：

```xml
<Style Selector="TargetControlClass.styleClassName">
```

这个选择器会匹配所有样式键为 `TargetControlClass`，并且带有 `styleClassName` 样式类的控件。

:::info
完整选择器列表请参阅 [style selector syntax](/docs/styling/style-selector-syntax)。
:::

## Setter 设置器

Setter 描述了当选择器匹配到控件时应执行什么修改。它本质上就是简单的属性/值对，格式如下：

```xml
<Setter Property="FontSize" Value="24"/>
<Setter Property="Padding" Value="4 2 0 4"/>
```

只要某个样式匹配到控件，该样式内部的所有 setter 就都会应用到这个控件上。

:::info
有关 setter 的更多说明，请参阅 [property setters](/docs/styling/property-setters)。
:::

## 嵌套样式

样式可以嵌套在其他样式内部。要创建嵌套样式，只需把子样式作为父 `<Style>` 元素的子元素，并让它的选择器以 [`Nesting selector (^)`](/docs/styling/style-selector-syntax) 开头：

```xml
<Style Selector="TextBlock.h1">
    <Setter Property="FontSize" Value="24"/>
    <Setter Property="FontWeight" Value="Bold"/>
    
    // highlight-start
    <Style Selector="^:pointerover">
        <Setter Property="Foreground" Value="Red"/>
    </Style>
    // highlight-end
</Style>
```

当你嵌套样式时，父样式的选择器会自动附加到子样式上。在上面的例子中，嵌套样式最终等效于选择器 `TextBlock.h1:pointerover`，也就是当指针移到该控件上方时，它会显示为红色前景色。

:::info
嵌套选择器必须存在，并且必须位于子选择器的起始位置。
:::

## 样式键

样式选择器匹配对象时，所依据的类型并不是控件的具体 CLR 类型，而是它的 `StyleKey` 属性值。

默认情况下，`StyleKey` 会返回当前实例自身的类型。不过，如果你的控件继承自 `Button`，但你希望它仍然按 `Button` 的方式被样式系统识别，那么你可以重写类中的 `StyleKeyOverride` 属性，并让它返回 `typeof(Button)`。

```csharp
public class MyButton : Button
{
    // `MyButton` will be styled as a standard `Button` control.
    protected override Type StyleKeyOverride => typeof(Button);
}
```

:::info
请注意，这一点与 WPF/UWP 的逻辑正好相反：在那些框架中，新派生控件默认会按其基类样式处理，除非你重写 `DefaultStyleKey`。而在 Avalonia 中，控件默认按其具体类型处理，除非你显式提供不同的样式键。
:::

:::info
在 Avalonia 11 之前，通常是通过实现 `IStyleable` 并重新实现 `IStyleable.StyleKey` 属性来覆盖样式键。出于兼容性考虑，这种机制在 Avalonia 11 中仍然受支持，但未来版本可能会移除。
:::

## 样式与资源

资源通常会和样式搭配使用，以帮助保持统一的视觉表现。你可以使用资源来定义应用内的标准颜色、图标，或者通过独立文件在多个应用之间共享这些资源。

:::info
有关如何在应用中使用资源，请参阅 [resource dictionaries](/docs/app-development/resource-dictionary)。
:::

## 另请参阅

- [Sharing styles](/docs/styling/sharing-styles)
- [Style classes](/docs/styling/style-classes)
- [Style selector syntax](/docs/styling/style-selector-syntax)
- [Property setters](/docs/styling/property-setters)
- [Resource dictionaries](/docs/app-development/resource-dictionary)
