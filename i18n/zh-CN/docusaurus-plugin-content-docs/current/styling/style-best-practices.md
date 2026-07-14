---
id: style-best-practices
title: 样式最佳实践
---

本页汇总了在 Avalonia 中编写样式、主题与模板时的一些实用建议。遵循这些模式有助于让你的样式代码更易维护、性能更稳定。

## 选择器特异性

Avalonia 会按照声明顺序评估样式选择器。当两个选择器匹配同一个控件并设置同一个属性时，后声明的那个会生效。建议按照“从通用到具体”的方式组织选择器：

```xml
<!-- General: applies to all Buttons -->
<Style Selector="Button">
    <Setter Property="Background" Value="#6366F1" />
</Style>

<!-- More specific: applies to Buttons with the .primary class -->
<Style Selector="Button.primary">
    <Setter Property="Background" Value="#2563EB" />
</Style>

<!-- Most specific: applies to primary Buttons that are hovered -->
<Style Selector="Button.primary:pointerover">
    <Setter Property="Background" Value="#3B82F6" />
</Style>
```

应避免使用 `*` 或 `:is(Control)` 这类过于宽泛的选择器，因为它们会匹配树中的所有控件，容易带来意外副作用。

## 用样式类代替内联属性

相比直接在控件上设置属性，更推荐使用样式类：

```xml
<!-- Avoid: repeated inline styles -->
<Button Background="#6366F1" Foreground="White" CornerRadius="6" Padding="16,8"
        Content="Save" />
<Button Background="#6366F1" Foreground="White" CornerRadius="6" Padding="16,8"
        Content="Cancel" />

<!-- Prefer: shared style class -->
<Style Selector="Button.action">
    <Setter Property="Background" Value="#6366F1" />
    <Setter Property="Foreground" Value="White" />
    <Setter Property="CornerRadius" Value="6" />
    <Setter Property="Padding" Value="16,8" />
</Style>

<Button Classes="action" Content="Save" />
<Button Classes="action" Content="Cancel" />
```

这样可以减少重复代码，并且更方便在一个地方统一更新外观。

## 主题相关的值使用 DynamicResource

当引用主题颜色、字体或尺寸时，应使用 `DynamicResource` 而不是硬编码的值。这样可以确保样式会随主题切换（浅色/深色）而更新：

```xml
<Style Selector="Button.themed">
    <Setter Property="Background" Value="{DynamicResource SystemAccentColor}" />
    <Setter Property="Foreground" Value="{DynamicResource SystemControlForegroundBaseHighBrush}" />
</Style>
```

`StaticResource` 更适合那些运行时不会变化的值，例如布局常量或自定义转换器。

## 让模板保持精简

在创建自定义模板时，只绑定模板真正会用到的属性。不要为了简单的视觉变化去编写复杂模板：

```xml
<!-- For a color change, use a style, not a template -->
<Style Selector="Button.custom">
    <Setter Property="Background" Value="Red" />
</Style>

<!-- Only create a template when you need a different visual structure -->
<Style Selector="Button.pill">
    <Setter Property="Template">
        <ControlTemplate>
            <Border Background="{TemplateBinding Background}"
                    CornerRadius="999"
                    Padding="{TemplateBinding Padding}">
                <ContentPresenter Content="{TemplateBinding Content}" />
            </Border>
        </ControlTemplate>
    </Setter>
</Style>
```

## 按作用域组织样式

将样式放在视觉树中合适的层级：

| 作用域 | 声明位置 | 使用场景 |
|---|---|---|
| 应用级 | `App.axaml` 或由 `App.axaml` 引用的 `Styles` 文件 | 品牌色、排版、默认控件主题 |
| 窗口/页面级 | `<Window.Styles>` 或 `<UserControl.Styles>` | 页面专属覆盖样式 |
| 控件级 | `<Control.Styles>` | 不应泄漏到外部的组件专属样式 |

```xml
<!-- App.axaml: global styles -->
<Application.Styles>
    <FluentTheme />
    <StyleInclude Source="avares://MyApp/Styles/Global.axaml" />
</Application.Styles>
```

```xml
<!-- A page-specific override -->
<UserControl.Styles>
    <Style Selector="TextBlock.page-title">
        <Setter Property="FontSize" Value="28" />
        <Setter Property="FontWeight" Value="Bold" />
    </Style>
</UserControl.Styles>
```

## 避免类似 !important 的补丁式写法

Avalonia 没有类似 CSS 中 `!important` 的机制。如果你的样式没有生效，通常原因是：

1. **后面声明的其他样式覆盖了它**：调整样式声明顺序。
2. **本地值优先级更高**：本地值（直接设置在控件上的值）会覆盖样式值。删除本地值，或改用样式设置。
3. **控件主题覆盖了它**：控件主题中包含的伪类样式可能具有更高特异性。

可以使用 DevTools（F12）检查某个属性最终由哪个值源生效。

## 命名约定

样式类建议使用 kebab-case，保持与 CSS 习惯一致：

```xml
<!-- Good -->
<Button Classes="btn-primary" />
<Border Classes="card-elevated" />

<!-- Also acceptable: single words -->
<Button Classes="primary" />
```

模板部件建议使用 `PART_` 前缀：

```xml
<Border x:Name="PART_Background" />
<ContentPresenter x:Name="PART_ContentPresenter" />
```

## 为状态变化添加过渡

添加过渡效果可以让伪类状态切换更平滑：

```xml
<Style Selector="Button.smooth">
    <Setter Property="Transitions">
        <Transitions>
            <BrushTransition Property="Background" Duration="0:0:0.15" />
            <DoubleTransition Property="Opacity" Duration="0:0:0.15" />
        </Transitions>
    </Setter>
</Style>
```

过渡时长建议保持较短（100ms 到 300ms）。过长的动画会让界面显得迟缓，并延迟用户反馈。

## 在不同主题下测试

如果你的应用同时支持浅色和深色主题，请在两种主题下都测试你的自定义样式。可以使用 `ThemeVariant` 资源为不同主题提供不同的值：

```xml
<Style Selector="Border.card">
    <Setter Property="Background">
        <Setter.Value>
            <SolidColorBrush x:Key="CardBackground">
                <SolidColorBrush.Color>
                    <OnPlatform Default="White" />
                </SolidColorBrush.Color>
            </SolidColorBrush>
        </Setter.Value>
    </Setter>
</Style>
```

或者使用在 `App.axaml` 中定义的主题变体资源：

```xml
<Application.Resources>
    <ResourceDictionary>
        <ResourceDictionary.ThemeDictionaries>
            <ResourceDictionary x:Key="Light">
                <Color x:Key="CardColor">#FFFFFF</Color>
            </ResourceDictionary>
            <ResourceDictionary x:Key="Dark">
                <Color x:Key="CardColor">#1E1E1E</Color>
            </ResourceDictionary>
        </ResourceDictionary.ThemeDictionaries>
    </ResourceDictionary>
</Application.Resources>
```

## 另请参阅

- [Styles](/docs/styling/styles)
- [Style classes](/docs/styling/style-classes)
- [Style selectors](/docs/styling/style-selectors)
- [Control themes](/docs/styling/control-themes)
- [Theme variants](/docs/styling/theme-variants)
- [Sharing styles](/docs/styling/sharing-styles)
