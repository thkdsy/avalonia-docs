---
id: theme-variants
title: 设置主题变体
---

import OverriddenThemeVariant from '/img/guides/ui-development/overridden-theme-variant.png';
import CustomThemeDictionaries from '/img/guides/ui-development/custom-theme-dictionaries.png';

:::tip
由于主题变体与资源系统深度集成，因此理解 Avalonia 的 [resources](/docs/app-development/resource-dictionary) 非常重要。
:::

## 简介

在 Avalonia 中，*主题变体* 是指控件在所选主题下呈现出的某种特定视觉外观。

通过使用主题变体，你可以创建既美观又一致的用户界面，并让它适应不同的用户偏好或系统设置。例如，一个应用可以提供白底黑字的浅色主题变体，也可以提供黑底白字的深色主题变体。用户选择自己偏好的主题后，应用就会相应调整界面外观。

Avalonia 的内置主题 `SimpleTheme` 和 `FluentTheme` 无需额外代码就支持 `Dark` 和 `Light` 两种变体。这让应用在使用内置控件时，可以根据系统偏好动态适配。对于更高级的自定义场景，本页会说明如何定义依赖主题变体的自定义资源，并在界面中引用它们。

## 切换当前主题变体

默认情况下，Avalonia 会继承系统范围内用户偏好所设置的主题变体。
你的应用可以通过两个重要属性来控制主题变体：[ActualThemeVariant](#actualthemevariant-property) 和 [RequestedThemeVariant](#requestedthemevariant-property)。借助它们，你可以在应用中的不同层级管理和切换主题变体。

### `ActualThemeVariant` property

只读属性 `ActualThemeVariant` 用于获取某个控件、窗口或应用当前实际使用的 UI 主题。它表示当前真正应用到该元素上的主题变体。
这个属性在每个控件上都可用，并会沿树向下继承。样式系统在访问主题字典时，也会使用它的值。

### `RequestedThemeVariant` property

`RequestedThemeVariant` 属性允许你覆盖当前主题变体，并为 `Application`、`Window`（`TopLevel`）或 [`ThemeVariantScope`](/api/avalonia/controls/themevariantscope) 指定期望的主题变体。

如果你想覆盖整个应用的主题变体，而不是使用系统默认值：
```xml title="App.axaml"
<Application xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             x:Class="AvaloniaApplication.App"
    // highlight-start
             RequestedThemeVariant="Dark">
    // highlight-end
  <Application.Styles>
    <FluentTheme />
  </Application.Styles>
</Application>
```

你也可以使用 `ThemeVariantScope` 控件为某个特定子树重新定义主题变体。下面的示例中，窗口使用 Dark 变体，而内部的 `ThemeVariantScope` 则将其重新定义为 Light 变体：

```xml title="MainWindow.axaml"
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x='http://schemas.microsoft.com/winfx/2006/xaml'
        x:Class="AvaloniaApplication.MainWindow"
    // highlight-start
        RequestedThemeVariant="Dark"
    // highlight-end
        Background="Gray">
  <StackPanel Spacing="5" Margin="5">
    <Button Content="Dark button" />
    // highlight-start
    <ThemeVariantScope RequestedThemeVariant="Light">
    // highlight-end
      <Button Content="Light button" />
    </ThemeVariantScope>
  </StackPanel>
</Window>
```

<Image light={OverriddenThemeVariant} alt="A screenshot of two buttons, demonstrating opposite appearances when dark or light theme settings are overridden." position="center" maxWidth={400} cornerRadius="true"/>

如果要重置 `RequestedThemeVariant` 的值，请设置 `RequestedThemeVariant="Default"`。

:::tip
在支持该功能的平台上，更改窗口的 `RequestedThemeVariant` 也会影响窗口装饰的主题变体。
:::

## 定义并引用特定主题变体的自定义资源

在 Avalonia 中，可以通过 `ResourceDictionary` 的 `ThemeDictionaries` 属性定义特定主题变体的资源。

通常，开发者会使用 `Light` 或 `Dark` 作为主题变体的键。使用 `Default` 作为键，则表示这个主题字典是一个回退项：当其他主题字典中找不到对应的主题变体或资源键时，就会使用它。

延续上一个示例，下面为 `BackgroundBrush` 和 `ForegroundBrush` 添加按主题变体区分的不同值：
```xml title="MainWindow.axaml"
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x='http://schemas.microsoft.com/winfx/2006/xaml'
        x:Class="Sandbox.MainWindow"
        RequestedThemeVariant="Dark"
        Background="Gray">
  <Window.Resources>
    // highlight-start
    <ResourceDictionary>
      <ResourceDictionary.ThemeDictionaries>
        <ResourceDictionary x:Key='Light'>
          <SolidColorBrush x:Key='BackgroundBrush'>SpringGreen</SolidColorBrush>
          <SolidColorBrush x:Key='ForegroundBrush'>Black</SolidColorBrush>
        </ResourceDictionary>
        <ResourceDictionary x:Key='Dark'>
          <SolidColorBrush x:Key='BackgroundBrush'>DodgerBlue</SolidColorBrush>
          <SolidColorBrush x:Key='ForegroundBrush'>White</SolidColorBrush>
        </ResourceDictionary>
      </ResourceDictionary.ThemeDictionaries>
    </ResourceDictionary>
    // highlight-end
  </Window.Resources>
  
  <Window.Styles>
    // highlight-start
    <Style Selector="Button">
      <Setter Property="Background" Value="{DynamicResource BackgroundBrush}" />
      <Setter Property="Foreground" Value="{DynamicResource ForegroundBrush}" />
    </Style>
    // highlight-end
  </Window.Styles>

  <StackPanel Spacing="5" Margin="5">
    <Button Content="Dark button"
            Background="{DynamicResource BackgroundBrush}"
            Foreground="{DynamicResource ForegroundBrush}" />
    <ThemeVariantScope RequestedThemeVariant="Light">
      <Button Content="Light button"
              Background="{DynamicResource BackgroundBrush}"
              Foreground="{DynamicResource ForegroundBrush}" />
    </ThemeVariantScope>
  </StackPanel>
</Window>

```

<Image light={CustomThemeDictionaries} alt="A screenshot of two brightly colored buttons in blue and green." position="center" maxWidth={400} cornerRadius="true"/>

:::caution
定义在 `ThemeDictionaries` 中的资源，只有在使用 `DynamicResource` 标记扩展时才可用。`StaticResource` 无法找到这些资源；除非在 `ResourceDictionary` 的非 `ThemeDictionaries` 区域中存在同名键，否则运行时会抛出异常。
:::

有关资源使用的更多细节，请参阅 [resources](/docs/app-development/resource-dictionary) 页面。

## 另请参阅

- [Resource dictionaries](/docs/app-development/resource-dictionary)
- [Styles](/docs/styling/styles)
- [Control themes](/docs/styling/control-themes)
