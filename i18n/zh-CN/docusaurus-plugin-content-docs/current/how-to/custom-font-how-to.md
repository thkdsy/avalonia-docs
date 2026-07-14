---
id: custom-font-how-to
title: 如何添加自定义字体
description: 以静态资源或嵌入式字体集合的方式，为 Avalonia 应用程序添加自定义字体。
doc-type: how-to
---

本指南将介绍两种向 Avalonia 应用程序添加自定义字体的方式：一种是把字体作为静态资源使用，另一种是把字体注册为嵌入式字体集合。

## 前置条件

- 一个 Avalonia 项目。本指南会使用 [Google Fonts sample project](https://github.com/AvaloniaUI/AvaloniaUI.QuickGuides/tree/main/GoogleFonts) 作为示例，但你可以把这些步骤迁移到自己的项目中。
- 一个字体文件（`.ttf` 或 `.otf`）。本指南以 [Nunito](https://fonts.google.com/specimen/Nunito) 为例。

## 将字体文件添加到项目中

1. 将字体文件复制到项目中的 **Assets/Fonts** 目录。
2. 打开你的 `.csproj` 文件，并确认该目录被包含为 `AvaloniaResource`：

```xml title="MyApp.csproj"
<AvaloniaResource Include="Assets\**" />
```

这样会把字体文件作为资源嵌入到构建产物中，以便 Avalonia 在运行时找到它们。如果你的项目里已经有一个覆盖 **Assets** 文件夹的 `AvaloniaResource` 配置，那么就不需要再额外添加了。

## 方案 A：将字体作为静态资源使用

这种方式会把字体声明为一个带名称的 XAML 资源。

### 声明字体资源

1. 打开 **App.axaml**。
2. 在 `<Application.Resources>` 中添加一个 [`FontFamily`](/api/avalonia/media/fontfamily) 资源，并使用[字体 URI 格式](/docs/styling/custom-fonts#font-uri-format)：

```xml title="App.axaml"
<Application xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             x:Class="MyApp.App"
             RequestedThemeVariant="Default">
    <Application.Styles>
        <FluentTheme />
    </Application.Styles>

    <Application.Resources>
        <FontFamily x:Key="NunitoFont">avares://MyApp/Assets/Fonts#Nunito</FontFamily>
    </Application.Resources>
</Application>
```

请将 `MyApp` 替换为你的程序集名称，并把 `Nunito` 替换为该字体的内部族名称。

### 应用字体

3. 打开一个 XAML 视图（例如 **MainWindow.axaml**）。
4. 使用 `StaticResource` 标记扩展设置 `FontFamily` 属性：

```xml title="MainWindow.axaml"
<TextBlock Text="Hello in Nunito"
           FontSize="24"
           FontFamily="{StaticResource NunitoFont}" />
```

5. 构建并运行应用程序。此时文本应当会以你的自定义字体显示。

任何带有 `FontFamily` 属性的控件都可以设置该值，因此你可以在 `TextBlock`、`Button`、`TextBox` 等控件上使用自定义字体。

## 方案 B：使用嵌入式字体集合

这种方式会把整个字体目录注册到一个自定义 URI 方案下，使你可以直接按名称引用字体，而无需为每个字体单独定义资源键。

### 创建字体集合类

1. 在项目中添加一个新的 C# 文件（例如 **MyFontCollection.cs**）。
2. 定义一个继承自 `EmbeddedFontCollection` 的类：

```csharp title="MyFontCollection.cs"
using System;
using Avalonia.Media.Fonts;

public sealed class MyFontCollection : EmbeddedFontCollection
{
    public MyFontCollection() : base(
        new Uri("fonts:MyFonts", UriKind.Absolute),
        new Uri("avares://MyApp/Assets/Fonts", UriKind.Absolute))
    {
    }
}
```

第一个 URI（`fonts:MyFonts`）是你在 XAML 中使用的方案名和集合键。第二个 URI 指向存放字体文件的资源目录。请将 `MyApp` 替换为你的程序集名称。

### 注册字体集合

3. 打开 **Program.cs**。
4. 使用 `AppBuilder.ConfigureFonts` 注册该字体集合：

```csharp title="Program.cs"
using Avalonia;
using System;

class Program
{
    [STAThread]
    public static void Main(string[] args) => BuildAvaloniaApp()
        .StartWithClassicDesktopLifetime(args);

    public static AppBuilder BuildAvaloniaApp() =>
        AppBuilder.Configure<App>()
            .UsePlatformDetect()
            .ConfigureFonts(fontManager =>
            {
                fontManager.AddFontCollection(new MyFontCollection());
            })
            .LogToTrace();
}
```

### 应用字体

5. 打开一个 XAML 视图（例如 **MainWindow.axaml**）。
6. 使用 `{scheme}:{collection-key}#{font-family-name}` 这种格式设置 `FontFamily` 属性：

```xml title="MainWindow.axaml"
<TextBlock Text="Hello in Nunito"
           FontSize="24"
           FontFamily="fonts:MyFonts#Nunito" />
```

7. 构建并运行应用程序。此时文本应当会以你的自定义字体显示。

如果你想从同一个集合中使用其他字体，只需修改 `#` 后面的名称即可。例如，`fonts:MyFonts#Roboto` 会从同一个 **Assets/Fonts** 目录中加载 Roboto 字体。

## 另请参阅

- [Custom fonts](/docs/styling/custom-fonts)
- [Assets](/docs/fundamentals/including-assets)
