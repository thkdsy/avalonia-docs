---
id: themes
title: 主题
---

import FluentThemeNormalScreenshot from '/img/concepts/ui-concepts/styling/fluent-theme-normal.png';
import FluentThemeForestScreenshot from '/img/concepts/ui-concepts/styling/fluent-theme-forest.png';
import SimpleThemeScreenshot from '/img/concepts/ui-concepts/styling/simple-theme.png';

在 Avalonia 中，主题是一整套面向内置控件的 [control themes](/docs/styling/control-themes) 和 [theme resources](/docs/styling/theme-variants)。

## 官方主题

Avalonia 提供了两个内置主题：

### Fluent 主题

- [Fluent Theme](#fluent) 是一个现代化主题，灵感来自 [Microsoft 的 Fluent Design System](https://en.wikipedia.org/wiki/Fluent_Design_System)。

### Simple 主题

- [Simple Theme](#simple) 是一个极简、轻量的主题，内置样式相对较少。

## 社区主题

社区开发的主题也有不少，它们处于不同的发展阶段。

### Material.Avalonia 

- [Material.Avalonia](https://github.com/AvaloniaCommunity/Material.Avalonia) 是一个现代化主题，灵感来自 [Google 的 Material Design System](https://m3.material.io/)。

### Semi.Avalonia

- [Semi.Avalonia](https://github.com/irihitech/Semi.Avalonia) 的设计灵感来自 [Semi Design](https://semi.design/en-US)
  
### Classic.Avalonia

- [Classic.Avalonia](https://github.com/BAndysc/Classic.Avalonia) 是一个经典风格主题，灵感来自 Windows 9x 系列的设计。

## Fluent 主题

Avalonia 的 Fluent 主题灵感来自 Microsoft 的 Fluent Design System。这是一整套用于构建视觉效果出色、交互自然的用户界面的设计规范与组件体系。Fluent Design 强调现代、简洁的美学风格，平滑的动画，以及直观的交互体验。它在不同平台之间提供一致而精致的外观，同时也借助 Avalonia 的样式系统为开发者保留了充足的灵活性。

<Image light={FluentThemeNormalScreenshot} alt="Fluent theme" position="center" maxWidth={400} cornerRadius="true" />

### 如何使用

首先，安装 [Avalonia.Themes.Fluent](https://www.nuget.org/packages/Avalonia.Themes.Fluent/) NuGet 包。

:::info
如果你不确定如何添加 NuGet 包，可以查看 [Visual Studio](https://learn.microsoft.com/en-us/nuget/quickstart/install-and-use-a-package-in-visual-studio) 或 [Rider](https://www.jetbrains.com/help/rider/Using_NuGet.html) 的相关文档。
:::

然后在 `Application` 中引入该主题：

```xml title="App.axaml"
<Application xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             x:Class="AvaloniaApplication.App">
  <Application.Styles>
    // highlight-start
    <FluentTheme />
    // highlight-end
  </Application.Styles>
</Application>
```

:::note
如果你还需要显式指定浅色或深色主题变体，请参阅 [Theme variants](/docs/styling/theme-variants)。
:::

### 更改主题密度

Fluent 主题内置了两组密度变体。
如果你想让界面更紧凑，可以设置 `DensityStyle` 属性：

```xml title="App.axaml"
<Application xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             x:Class="AvaloniaApplication.App">
  <Application.Styles>
    // highlight-start
    <FluentTheme DensityStyle="Compact" />
    // highlight-end
  </Application.Styles>
</Application>
```

### 创建自定义颜色调色板

虽然 `FluentTheme` 已经内置了浅色和深色变体所需的资源，但你仍然可以覆盖这些变体所使用的基础调色板。
当你希望沿用同一套主题结构、但换成不同配色时，这会非常有用。

要做到这一点，可以为每个变体定义自定义的 `ColorPaletteResources`：

```xml title="App.axaml"
<Application xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             x:Class="AvaloniaApplication.App">
  <Application.Styles>
    <FluentTheme>
    // highlight-start
      <FluentTheme.Palettes>
        <!-- 浅色主题变体的调色板 -->
        <ColorPaletteResources x:Key="Light" Accent="Green" RegionColor="White" ErrorText="Red" />
        <!-- 深色主题变体的调色板 -->
        <ColorPaletteResources x:Key="Dark" Accent="DarkGreen" RegionColor="Black" ErrorText="Yellow" />
      </FluentTheme.Palettes>
    // highlight-end
    </FluentTheme>
  </Application.Styles>
</Application>
```

`ColorPaletteResources` 提供了很多可单独覆盖的颜色属性，并且可针对每个变体独立设置。你只需要重写那些你真正想改的属性，其他部分都会保留默认值。在上面的例子里，就只覆盖了少数几个颜色。

如果你没有覆盖 `Accent`，那么 Avalonia 会在可用时使用操作系统的强调色。
`Accent` 支持绑定，并且可以在运行时修改；而调色板中的其他属性出于性能考虑，只会在启动时读取一次，并以静态方式使用。

你也可以在 code-behind 中构建调色板，但规则是一样的：只有 `Accent` 可以动态更新。

:::note
FluentTheme 只支持 `Dark` 和 `Light` 两种主题变体，无法为自定义变体单独定义调色板。
:::

### 使用在线编辑器创建自定义调色板

Microsoft 的 Fluent Theme Editor 已被移植到 Avalonia，并且可以与 `FluentTheme` 配合使用。
[Avalonia Theme Editor](https://theme.xaml.live/) 支持以下功能：

1. 编辑浅色和深色变体的调色板颜色。
2. 预览当前调色板效果。
3. 将当前调色板导出为 XAML 代码，便于复制到 `App.axaml` 中。
4. 把当前颜色保存为 json 文件，并从文件系统重新加载。
5. 当调色板颜色对比度过低时自动给出提示。
6. 提供快速起步预设。

下面是一个使用在线工具中 Forest 调色板预设的 FluentTheme 示例：

<Image light={FluentThemeForestScreenshot} alt="Fluent theme forest palette" position="center" maxWidth={400} cornerRadius="true" />

## Simple 主题

Avalonia 的 Simple 主题旨在保持极简和轻量，内置样式较少。它为你在其基础上构建自定义样式提供了一个干净的起点。由于其视觉和结构复杂度都较低，因此也很适合运行在嵌入式设备上的应用程序。

<Image light={SimpleThemeScreenshot} alt="A screenshot of a user interface, demonstrating the appearances of various UI controls using a simple design theme." position="center" maxWidth={400} cornerRadius="true"/>

### 如何使用

首先，安装 [Avalonia.Themes.Simple](https://www.nuget.org/packages/Avalonia.Themes.Simple/) NuGet 包。

:::info
如果你不确定如何添加 NuGet 包，可以查看 [Visual Studio](https://learn.microsoft.com/en-us/nuget/quickstart/install-and-use-a-package-in-visual-studio) 或 [Rider](https://www.jetbrains.com/help/rider/Using_NuGet.html) 的相关文档。
:::

然后在 `Application` 中引入该主题：

```xml title="App.axaml"
<Application xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             x:Class="AvaloniaApplication.App">
  <Application.Styles>
    // highlight-start
    <SimpleTheme />
    // highlight-end
  </Application.Styles>
</Application>

```

:::note
如果你需要显式指定浅色或深色主题变体，请参阅 [Theme variants](/docs/styling/theme-variants)。
:::

## 另请参阅

- [Control themes](/docs/styling/control-themes)
- [Theme variants](/docs/styling/theme-variants)
- [Styles](/docs/styling/styles)
