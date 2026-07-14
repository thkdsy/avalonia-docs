---
id: styling-controls-how-to
title: "如何：为控件设置样式和主题"
description: 在 Avalonia 中通过颜色、变体、主题和可复用样式来自定义控件外观。
doc-type: how-to
---

本指南介绍如何在 Avalonia 应用程序中自定义控件外观，包括修改颜色、创建变体、适配浅色/深色主题，以及构建可复用样式。

## 修改控件颜色

### 使用样式类

你可以定义一个按控件类型和类名匹配的样式类，然后在 AXAML 中把该类应用到控件上。下面的示例为 [`Button`](/api/avalonia/controls/button) 创建了一个 `primary` 样式类，它会设置 `Background`、`Foreground`，以及悬停和按下状态的表现：

```xml
<Window.Styles>
    <Style Selector="Button.primary">
        <Setter Property="Background" Value="#6366F1" />
        <Setter Property="Foreground" Value="White" />
    </Style>
    <Style Selector="Button.primary:pointerover">
        <Setter Property="Background" Value="#818CF8" />
    </Style>
    <Style Selector="Button.primary:pressed">
        <Setter Property="Background" Value="#4F46E5" />
    </Style>
</Window.Styles>

<Button Classes="primary" Content="Submit" />
```

### 多个类组合使用

你可以在同一个控件上组合多个类，以叠加彼此独立的样式关注点。每个类都会贡献自己的一组属性设置，因此你可以自由混搭：

```xml
<Style Selector="Button.rounded">
    <Setter Property="CornerRadius" Value="999" />
</Style>
<Style Selector="Button.large">
    <Setter Property="Padding" Value="24,12" />
    <Setter Property="FontSize" Value="16" />
</Style>

<Button Classes="primary rounded large" Content="Submit" />
```

在这个例子中，`Button` 会同时应用 `primary`、`rounded` 和 `large` 这三个类的样式。

## 创建按钮变体

你可以先定义一个应用到所有 `Button` 的基础样式，再为每种按钮变体增加基于类的覆盖样式，从而为整个应用建立一套统一的按钮体系。这样既能保证视觉语言一致，又能在不同场景下保留足够灵活性：

```xml
<Application.Styles>
    <!-- 基础按钮样式 -->
    <Style Selector="Button">
        <Setter Property="CornerRadius" Value="6" />
        <Setter Property="Padding" Value="16,8" />
        <Setter Property="Transitions">
            <Transitions>
                <BrushTransition Property="Background" Duration="0:0:0.15" />
            </Transitions>
        </Setter>
    </Style>

    <!-- 主按钮变体 -->
    <Style Selector="Button.primary">
        <Setter Property="Background" Value="#6366F1" />
        <Setter Property="Foreground" Value="White" />
    </Style>
    <Style Selector="Button.primary:pointerover">
        <Setter Property="Background" Value="#818CF8" />
    </Style>
    <Style Selector="Button.primary:pressed">
        <Setter Property="Background" Value="#4F46E5" />
    </Style>

    <!-- 危险操作变体 -->
    <Style Selector="Button.danger">
        <Setter Property="Background" Value="#EF4444" />
        <Setter Property="Foreground" Value="White" />
    </Style>
    <Style Selector="Button.danger:pointerover">
        <Setter Property="Background" Value="#F87171" />
    </Style>
    <Style Selector="Button.danger:pressed">
        <Setter Property="Background" Value="#DC2626" />
    </Style>

    <!-- Ghost 变体（无背景） -->
    <Style Selector="Button.ghost">
        <Setter Property="Background" Value="Transparent" />
        <Setter Property="BorderThickness" Value="0" />
    </Style>
    <Style Selector="Button.ghost:pointerover">
        <Setter Property="Background" Value="#10000000" />
    </Style>
</Application.Styles>
```

之后你就可以在应用中的任意位置使用这些变体：

```xml
<StackPanel Orientation="Horizontal" Spacing="8">
    <Button Classes="primary" Content="Save" />
    <Button Classes="danger" Content="Delete" />
    <Button Classes="ghost" Content="Cancel" />
</StackPanel>
```

## 主题感知颜色

如果你希望自定义颜色在用户切换浅色/深色模式时自动适配，可以把它们定义在 `ThemeDictionaries` 中。每个字典分别对应 `Light` 或 `Dark`，Avalonia 会在运行时自动选择合适的一组：

```xml
<Application.Resources>
    <ResourceDictionary>
        <ResourceDictionary.ThemeDictionaries>
            <ResourceDictionary x:Key="Light">
                <SolidColorBrush x:Key="CardBackground" Color="#FFFFFF" />
                <SolidColorBrush x:Key="CardBorder" Color="#E5E7EB" />
                <SolidColorBrush x:Key="TextPrimary" Color="#111827" />
                <SolidColorBrush x:Key="TextSecondary" Color="#6B7280" />
            </ResourceDictionary>
            <ResourceDictionary x:Key="Dark">
                <SolidColorBrush x:Key="CardBackground" Color="#1F2937" />
                <SolidColorBrush x:Key="CardBorder" Color="#374151" />
                <SolidColorBrush x:Key="TextPrimary" Color="#F9FAFB" />
                <SolidColorBrush x:Key="TextSecondary" Color="#9CA3AF" />
            </ResourceDictionary>
        </ResourceDictionary.ThemeDictionaries>
    </ResourceDictionary>
</Application.Resources>
```

在样式中通过 `DynamicResource` 引用这些资源，这样当活动主题变化时，对应值也会同步更新：

```xml
<Style Selector="Border.card">
    <Setter Property="Background" Value="{DynamicResource CardBackground}" />
    <Setter Property="BorderBrush" Value="{DynamicResource CardBorder}" />
    <Setter Property="BorderThickness" Value="1" />
    <Setter Property="CornerRadius" Value="8" />
    <Setter Property="Padding" Value="16" />
</Style>
```

:::tip
对于主题字典中的值，请使用 `DynamicResource`，而不是 `StaticResource`。因为 `StaticResource` 只会在加载时解析一次，之后主题切换时不会自动更新。
:::

## 自定义 `TextBox` 外观

你可以把 `TextBox` 改造成“只显示下划线”的外观，而不是完整边框。下面的示例通过设置 `BorderThickness` 只显示底边，并在 `:focus` 和 `:error` 伪类下改变颜色：

```xml
<Style Selector="TextBox.underline">
    <Setter Property="BorderThickness" Value="0,0,0,2" />
    <Setter Property="BorderBrush" Value="#D1D5DB" />
    <Setter Property="Background" Value="Transparent" />
    <Setter Property="CornerRadius" Value="0" />
    <Setter Property="Padding" Value="0,8" />
</Style>
<Style Selector="TextBox.underline:focus">
    <Setter Property="BorderBrush" Value="#6366F1" />
</Style>
<Style Selector="TextBox.underline:error">
    <Setter Property="BorderBrush" Value="#EF4444" />
</Style>
```

## 卡片组件

你可以通过给 `Border` 控件应用类，来创建可复用的卡片样式。`card` 类表示带边框的平面卡片，而 `card-elevated` 则通过 `BoxShadow` 实现悬浮感：

```xml
<Style Selector="Border.card">
    <Setter Property="Background" Value="{DynamicResource CardBackground}" />
    <Setter Property="BorderBrush" Value="{DynamicResource CardBorder}" />
    <Setter Property="BorderThickness" Value="1" />
    <Setter Property="CornerRadius" Value="8" />
    <Setter Property="Padding" Value="16" />
</Style>

<Style Selector="Border.card-elevated">
    <Setter Property="Background" Value="{DynamicResource CardBackground}" />
    <Setter Property="BoxShadow" Value="0 4 6 0 #15000000" />
    <Setter Property="CornerRadius" Value="8" />
    <Setter Property="Padding" Value="16" />
</Style>
```

这些卡片样式可以这样使用：

```xml
<Border Classes="card">
    <StackPanel Spacing="8">
        <TextBlock Text="Card Title" FontWeight="Bold" />
        <TextBlock Text="Card content here" Foreground="Gray" />
    </StackPanel>
</Border>
```

## 将样式提取到共享文件

当样式数量逐渐增多时，你可以把它们提取到独立的 `.axaml` 文件中。这样不仅能让 `App.axaml` 更整洁，也便于在多个项目之间复用：

```xml title="Styles/ButtonStyles.axaml"
<Styles xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <Style Selector="Button.primary">
        <Setter Property="Background" Value="#6366F1" />
        <Setter Property="Foreground" Value="White" />
    </Style>
    <!-- more styles... -->
</Styles>
```

然后在 `App.axaml` 中通过 `StyleInclude` 引用这个文件。`avares://` URI 方案指向的是程序集中的嵌入资源：

```xml
<Application.Styles>
    <FluentTheme />
    <StyleInclude Source="avares://MyApp/Styles/ButtonStyles.axaml" />
</Application.Styles>
```

## 覆盖主题样式

应用程序自己的样式会在内置主题之后应用，因此你可以覆盖默认外观。为了确保自定义样式拥有更高优先级，请把它们放在 `Application.Styles` 中 `<FluentTheme />` 之后：

```xml
<Application.Styles>
    <FluentTheme />

    <!-- 覆盖所有 TextBlock，使用你偏好的字体 -->
    <Style Selector="TextBlock">
        <Setter Property="FontFamily" Value="Inter, Segoe UI, sans-serif" />
    </Style>

    <!-- Override ComboBox dropdown max height -->
    <Style Selector="ComboBox /template/ Popup#PART_Popup">
        <Setter Property="MaxHeight" Value="300" />
    </Style>
</Application.Styles>
```

`/template/` 选择器允许你深入控件模板内部，定位其中的内部部件。在这个例子里，它针对的是 `ComboBox` 模板内部名为 `PART_Popup` 的 `Popup`。

## 使用伪类进行条件样式设置

伪类允许你根据控件当前状态应用不同样式，而无需编写 code-behind。Avalonia 会在控件状态变化时自动重新评估这些伪类选择器：

```xml
<!-- 禁用状态 -->
<Style Selector="Button:disabled">
    <Setter Property="Opacity" Value="0.5" />
</Style>

<!-- Checked state (ToggleButton, CheckBox, RadioButton) -->
<Style Selector="ToggleButton:checked">
    <Setter Property="Background" Value="#6366F1" />
    <Setter Property="Foreground" Value="White" />
</Style>

<!-- Focus visible (keyboard navigation only) -->
<Style Selector="Button:focus-visible">
    <Setter Property="BorderBrush" Value="#6366F1" />
    <Setter Property="BorderThickness" Value="2" />
</Style>
```

常见伪类包括 `:pointerover`、`:pressed`、`:disabled`、`:focus`、`:focus-visible`、`:checked` 和 `:error`。完整列表请参阅 [Pseudo-classes](/docs/styling/pseudoclasses)。

## 另请参阅

- [Styles](/docs/styling/styles)
- [Style Classes](/docs/styling/style-classes)
- [Style Selectors](/docs/styling/style-selectors)
- [Pseudo-classes](/docs/styling/pseudoclasses)
- [Style Best Practices](/docs/styling/style-best-practices)
- [Sharing Styles](/docs/styling/sharing-styles)
- [Control Template Walkthrough](/docs/styling/control-template-walkthrough)
- [Theme Variants](/docs/styling/theme-variants)
