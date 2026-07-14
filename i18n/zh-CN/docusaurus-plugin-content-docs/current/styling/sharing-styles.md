---
id: sharing-styles
title: 共享样式
---

import VsStylesTemplateScreenshot from '/img/guides/ui-development/styling/vs-styles-template.png';

你可以将样式定义在独立文件中，并在应用的任意层级引入它们。这样就能在多个窗口、用户控件，甚至多个项目之间共享一致的样式集合。

## 如何使用引入的样式

本指南演示如何通过在应用中引入独立的样式文件来共享样式。这种方式还支持在多个应用之间复用同一套样式。

要实现这一点，请在一个新的 XAML 文件中定义样式。根元素必须是 `Style` 或 `Styles`。例如：

```xml
<Styles xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <Style Selector="TextBlock.h1">
        <Setter Property="FontSize" Value="24"/>
        <Setter Property="FontWeight" Value="Bold"/>
    </Style>
</Styles>
```

Avalonia 的项目模板提供了快速向项目中添加样式文件的方法。操作步骤如下：

- 在 **Solution Explorer** 中右键单击你的项目。
- 点击 **Add** 和 **New Item**。
- 在 Avalonia 项目项中选择 **Styles (Avalonia)**。
- 为样式文件输入名称。

<Image light={VsStylesTemplateScreenshot} alt="Visual Studio Add New Item dialog showing the Styles (Avalonia) template" position="center" maxWidth={400} cornerRadius="true"/>

要使用定义在独立文件中的样式，你必须通过 [`StyleInclude`](/api/avalonia/markup/xaml/styling/styleinclude) 元素引用它。`Source` 属性用于指定样式文件的位置。你可以根据需要选择将这个元素放在哪个层级。

例如，如果要使用定义在 `/Styles` 文件夹中 `AppStyles.axaml` 文件里的样式，可以在窗口中这样写一个 `StyleInclude` 元素：

```xml
<Window ... >
    <Window.Styles>
        <StyleInclude Source="/Styles/AppStyles.axaml" />
    </Window.Styles>

    <StackPanel>
       <TextBlock Classes="h1">Heading 1</TextBlock>
       <TextBlock>This is not a heading and will not be changed.</TextBlock>
    </StackPanel>
</Window>
```

不过，更常见的做法是在 `App.axaml` 文件中引用样式文件，例如：

```xml
<Application... > 
    <Application.Styles>
        <FluentTheme />
        <StyleInclude Source="/AppStyles.axaml"/>
    </Application.Styles>
</Application>
```

这样就可以在整个应用中使用该独立文件里的样式。

你还可以通过 `avares://` 前缀引入来自其他程序集的样式：

```xml
<Application... > 
    <Application.Styles>
        <FluentTheme />
        <StyleInclude Source="avares://MyApp.Shared/Styles/CommonAppStyles.axaml"/>
    </Application.Styles>
</Application>
```

这会引用 `MyApp.Shared` 项目中的 `/Styles/CommonAppStyles.axaml` 文件。

## 故障排查：在引入的样式中找不到 StaticResource

当你将资源和样式拆分到多个被引入的文件中时，即使资源文件已经先于引用它的样式文件被引入，`StaticResource` 查找仍然可能失败。

这是因为每个 `StyleInclude` 都会先独立加载，然后才被附加到其父级 `Styles` 集合中。在加载阶段，被引入的样式无法访问同级的其他 include，因此不能解析定义在另一个文件中的 `StaticResource`。

例如，如果你有一个用于定义字体资源的 `Fonts.axaml`，以及一个引用这些字体资源的 `TextStyles.axaml`：

```xml
<Application.Styles>
    <StyleInclude Source="/Styles/Fonts.axaml" />
    <StyleInclude Source="/Styles/TextStyles.axaml" />
</Application.Styles>
```

那么 `TextStyles.axaml` 中指向 `Fonts.axaml` 所定义字体的 `StaticResource` 引用，在加载时就会失败。

你可以通过以下两种方式解决：

- **使用 `DynamicResource` 替代 `StaticResource`。** `DynamicResource` 会在运行时、所有样式附加完成之后再解析，因此能够找到定义在同级 include 中的资源。这是大多数场景下推荐的方式，而且性能影响通常可以忽略不计，因为主题资源很少在运行时频繁变化。
- **将资源定义在引用它们的样式文件中。** 如果某个样式文件需要字体或画刷资源，就把这些资源也放进同一个文件里，而不是依赖另一个独立的资源文件。

## 另请参阅

- [Styles](/docs/styling/styles)
- [Style precedence](/docs/styling/style-precedence)
