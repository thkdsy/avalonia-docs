---
id: themes
title: 主题
description: 故障排除 Avalonia UI 主题中的常见问题，包括缺少控件主题、意外的样式覆盖，以及应用程序窗口透明的问题。
doc-type: troubleshooting
---

## 我的控件主题没有被找到

如果 Avalonia 没有拾取到你的自定义控件主题，请确认该主题返回的 [style key](/docs/styling/styles) 同时匹配你的控件主题的 `x:Key` 和 `TargetType`。

常见原因包括：

- **`x:Key` 不匹配**：你在 XAML 中引用的键与控件主题资源中定义的键不一致。
- **`TargetType` 错误**：`ControlTheme` 上的 `TargetType` 与你尝试设置样式的控件不匹配。
- **主题未包含**：你没有添加指向包含该控件主题文件的 `StyleInclude` 或 `ResourceInclude`。

要诊断此问题，请在运行时打开 [Avalonia DevTools](/tools/developer-tools/installation) 并检查 `Styles` 面板。这会显示所选控件上当前生效的样式和主题，帮助你确认主题是否已加载并应用。

```xml title="示例：定义并引用控件主题"
<!-- 在你的主题文件中（例如 MyButtonTheme.axaml） -->
<ControlTheme x:Key="{x:Type Button}" TargetType="Button">
    <Setter Property="Background" Value="SlateBlue" />
    <!-- Additional setters and template here -->
</ControlTheme>

<!-- 在 App.axaml 中包含该主题 -->
<Application.Styles>
    <FluentTheme />
    <StyleInclude Source="/Themes/MyButtonTheme.axaml" />
</Application.Styles>
```

## 我的控件主题破坏了其他控件

许多 Avalonia 控件内部由其他 Avalonia 控件组成。如果你创建了一个针对某一类型所有控件的样式或控件主题，你可能会得到意料之外的结果，因为该样式会应用于视觉树中该类型的每一个实例，包括嵌套在其他控件内部的实例。

例如，如果你在 `Window` 中创建了一个针对 `TextBlock` 的样式，那么该样式会应用于窗口中的每一个 `TextBlock`，即使它们是其他控件模板的一部分（例如 `ListBox` 项或 `Button` 标签）。

```xml title="示例：一个无意中影响嵌套控件的样式"
<Window.Styles>
    <Style Selector="TextBlock">
        <Setter Property="Foreground" Value="Red" />
    </Style>
</Window.Styles>
```

要将样式限制为仅影响你打算影响的控件，请使用更具体的选择器。你可以通过样式类、名称或嵌套上下文来限定范围：

```xml title="示例：使用类选择器限定样式范围"
<Window.Styles>
    <Style Selector="TextBlock.heading">
        <Setter Property="Foreground" Value="Red" />
    </Style>
</Window.Styles>

<!-- 只有这个 TextBlock 会受到影响 -->
<TextBlock Classes="heading" Text="Page title" />
```

## 应用程序窗口是透明的，或者没有渲染任何内容

如果你的应用程序窗口看起来是透明的，或者没有显示可见内容，最可能的原因是没有安装 Avalonia 主题。Avalonia 需要一个基础主题（例如 `FluentTheme` 或 `SimpleTheme`）来提供默认的控件模板和样式。没有它，控件就没有可视化表现。

要修复此问题，请确保你的 `App.axaml` 包含一个主题：

```xml title="App.axaml"
<Application.Styles>
    <FluentTheme />
</Application.Styles>
```

如果你使用的是第三方主题，请确认：

- 该主题的 NuGet 包已安装到你的项目中。
- 该主题已包含在你的 `Application.Styles` 集合中。
- 该主题与你使用的 Avalonia 版本兼容。

如果在使用第三方主题后问题仍然存在，请联系该主题的维护者寻求支持。

## 另请参阅

- [主题概览](/docs/styling/themes)
- [样式](/docs/styling/styles)
- [样式故障排除](/docs/styling/styles)
- [如何使用控件主题](/docs/styling/control-themes)
- [开发者工具](/tools/developer-tools/installation)