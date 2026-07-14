---
id: getting-started
title: 入门
---

:::tip[使用 AI 帮助迁移]
如果你使用支持 MCP 的 AI 编码助手（VS Code、Cursor、Rider、Claude Code 等），[Build MCP](/tools/ai-tools/build-mcp) 服务器可以一步步带你完成这个过程。让你的助手迁移 WPF 项目，它会分析依赖项、配置 NuGet 源、切换 SDK，并以交互方式排查问题。有关配置说明，请参见 [Build MCP 设置](/tools/ai-tools/build-mcp#setting-up-the-mcp-server)。
:::

## 第 1 步：准备你的 WPF 项目

:::note
本文档以 .NET 7.0 为例，但 XPF 支持 .NET 6.0 及以上版本。 

建议使用 .NET 8（当前 LTS）或 .NET 9。
:::

请确保你的项目已经更新/迁移到至少 `net6.0-windows`，并使用 SDK 风格的 `.csproj` 格式。SDK 风格项目以 `<Project Sdk="Microsoft.NET.Sdk">` 开头，而不是使用带有 `<Import>` 元素的旧式冗长格式。

如果你的项目仍在使用传统的 `.csproj` 格式，请使用 .NET Upgrade Assistant，或手动进行转换。关键更改包括：
- 用 SDK 风格的根元素 `<Project Sdk="Microsoft.NET.Sdk">` 替换冗长的 XML
- 设置 `<TargetFramework>net8.0-windows</TargetFramework>`
- 添加 `<UseWpf>true</UseWpf>`
- 移除显式文件包含（SDK 风格项目会自动包含文件）

在继续之前，请确认你的项目能够在 .NET 8（或更高版本）上配合 WPF 正常构建和运行。

:::danger
这一步 **至关重要**。XPF 不适用于旧式/传统的 `.csproj` 格式，也不适用于低于 6.0 的 .NET 版本。你必须先转换项目，并确保 WPF 在现代 .NET 版本上正常工作，然后才能尝试使用 XPF。
:::

:::danger
如果你在 Linux 上运行，请在安装 .NET 之前先查看 [linux](/xpf/platforms/linux) 指南。
:::

## 第 2 步：添加 `NuGet.config`

在解决方案根目录创建一个 `NuGet.config` 文件，或修改现有文件，使其包含以下内容：

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <clear />
    <add key="api.nuget.org" value="https://api.nuget.org/v3/index.json" />
    <add key="xpf" value="https://xpf-nuget-feed.avaloniaui.net/v3/index.json" />
    <add key="avalonia-nightly" value="https://nuget-feed-all.avaloniaui.net/v3/index.json" />
  </packageSources>
  <packageSourceCredentials>
    <xpf>
      <add key="Username" value="license" />
      <add key="ClearTextPassword" value="<YOUR_LICENSE_KEY>" />
    </xpf>
  </packageSourceCredentials>
</configuration>
```

## 第 3 步：使用 XPF SDK

在可执行的 WPF 项目中，在 `.csproj` 中将 SDK 更改为使用 XPF SDK。第一行：

```xml
<Project Sdk="Microsoft.NET.Sdk">
``` 

应更改为：

```xml
<Project Sdk="Xpf.Sdk/1.6.0">
```

:::note
XPF 正在积极开发中，CI 构建版本会频繁变化。这里给出的版本是撰写本文时的最新版本，但很可能已经有更新的版本可用。你可以在 https://xpf-nuget-feed.avaloniaui.net/packages/xpf.sdk 查找最新的 CI 构建版本。更多信息请参见 [夜间构建](/xpf/version-info/versioning)。
:::

:::tip
如果你有多个项目需要使用相同的 XPF SDK 版本，可以 [在 `global.json` 中指定此版本](/xpf/configuration/centralizing-multiple-xpf-projects)
:::

## 第 4 步：添加你的许可证密钥

在可执行程序的 `.csproj` 中添加：

```xml
  <ItemGroup>
    <RuntimeHostConfigurationOption Include="AvaloniaUI.Xpf.LicenseKey" Value="<YOUR_LICENSE_KEY>" />
  </ItemGroup>  
```

注意，如果你使用的是生产许可证，项目的 AssemblyName 必须与你的许可证密钥匹配

## 第 5 步：清理你的解决方案

更改项目 SDK 需要清理现有的生成产物：

- 在命令行中运行 `dotnet clean`；或
- 在 IDE 中使用 `Build -> Clean Solution`；或
- 手动删除 `obj`/`bin` 目录

## 第 6 步：运行项目

你应该能够在你偏好的 IDE 中使用 Avalonia XPF 运行项目，或者使用 `dotnet run`。

:::tip
如果在 Linux 上运行，请参阅 [Linux](/xpf/platforms/linux) 页面，了解如何安装 .NET 和所需依赖项。
:::

## 其他项目

如果你有使用 WPF API 的非可执行项目，并且需要在 Linux 或 macOS 上构建这些项目，你可以按上述方式更改 SDK；或者如果你的目标是 `net7.0-windows`，请添加：

```xml
<PropertyGroup>
  <EnableWindowsTargeting>true</EnableWindowsTargeting>
</PropertyGroup>
```

到相应的项目文件中。

或者，在解决方案根目录创建一个 `Directory.Build.props` 文件，并写入以下内容：

```xml
<Project>
  <PropertyGroup>
    <EnableWindowsTargeting>true</EnableWindowsTargeting>
  </PropertyGroup>
</Project>  
```

## 目标框架

理想情况下，所有引用 XPF 的项目都应使用 `net6.0-windows` 或 `net7.0-windows` TFM。你也可以使用 `net6.0` 或 `net7.0` TFM，但在这种情况下你不能使用 `<EnableWindowsTargeting>`，而必须使用 XPF SDK。

:::tip
使用 XPF SDK 时，`-windows` 目标框架（例如 `net8.0-windows`）可在所有平台上工作。你不需要更改目标框架即可在 Linux 或 macOS 上构建或运行。某些第三方库需要 Windows 特定的 TFM，因此保留 `net8.0-windows` 往往是最简单的做法。
:::

## WinForms 承载（仅限 Windows）

如果你的应用需要在 XPF 中承载 WinForms 控件，请在 `.csproj` 中 Windows 条件化的 `PropertyGroup` 里添加以下内容：

```xml
<PropertyGroup Condition="$([MSBuild]::IsOSPlatform('Windows'))">
    <XpfUseMicrosoftWindowsForms>true</XpfUseMicrosoftWindowsForms>
</PropertyGroup>
```

这会禁用 WinForms shim 层并启用原生 WinForms 集成。请注意，WinForms 承载仅在 Windows 上可用，如果没有正确添加条件，在其他平台上会导致构建失败。

## 迁移提示

### 项目文件

1. 将所有项目转换为 .NET 8.0 及以上版本。旧的项目文件格式（非 SDK 风格 `.csproj`）在 Windows 之外无法工作。
2. 强烈建议先在 Windows 上完成第 (1) 步，以免在其他平台上处理难以调试的 Windows 特定依赖问题。请将应用中 .NET 7.0 中已弃用的功能（如 AppDomain、CodeDOM、WCF、`System.Web`、XmlSerializer，以及仅限 Windows 的硬依赖 API，如 `System.Management.Instrumentation` 和 `System.Drawing.Common`）替换或移除，并使用跨平台友好的替代方案。
3. 在执行第 (1) 步时，请留意应用可能使用的任何自定义 MSBuild Tasks。通过运行 `dotnet build` 确保这些 Tasks 仍能在 .NET 7.0 上工作。不要在 Visual Studio 内测试，以便确认它在外部也能工作。
4. 将所有 PCL（Portable Class Libraries）转换为 `netstandard` 库。
5. 从 `.csproj` 中移除任何 `ApplicationDefinition` 条目。
6. 移除那些分别定义 `Configuration`、`Platform`、`ProjectGuid`、`OutputType`、`RootNamespace` 等属性的冗长 `PropertyGroup` 元素。SDK 风格项目会为所有这些属性提供合理的默认值。

### 依赖项

7. 如果你使用了基于 .NET Framework 的 nuget 包，请尽量寻找该包的更新版本（`netstandard2.0`、`netcoreapp2.0`+、`net5.0`+）。大多数情况下，这些 .NET Framework 包也能跨平台工作，但这并不保证。 
8. 如果项目文件中有 `Reference` 项链接到一个独立的 `dll`，请尽量按照第 (4) 步所述在 NuGet 上寻找替代方案。如果它是托管程序集，通常可以工作，但同样不保证。
9. 如果你有任何原生二进制文件，请尝试找托管等效物，或为目标平台重新编译它们。在 .NET 8+ 上，使用 `System.Runtime.InteropServices.NativeLibrary` 和 `DllImport` 进行原生互操作。
10. 将依赖项更新到最新版本，尤其是 Actipro、DevExpress、Syncfusion 和 Telerik 等第三方组件。

### Windows

11. 避免使用自定义窗口外壳控件（例如 WPF 的 WindowChrome、MahApps 的 MetroWindow、DevExpress 的 DXWindow）以及任何会自定义窗口边框或行为的内容，因为不能保证它们适配目标平台的 UI 设计（例如 macOS 上的自定义 MetroWindow）。XPF 目标平台的最佳设计是单视图应用（例如像网站或移动应用那样）。

### 资源与设置

12. 资源文件（`.resx`）不会在 Visual Studio 之外重新生成。请考虑使用 JSON 文件或其他不依赖 Visual Studio 的本地化方案。
13. Visual Studio 文本模板（T4、*.template 文件）在 .NET 7.0 中也已弃用。请使用源生成器作为替代。
14. 资源文件（`.resx`）中的图像或位图与 .NET 7.0 不兼容：请考虑改用 WPF 的资源方案。
15. 避免使用 `App.Config` / `System.Configuration.ConfigurationManager`，因为在不允许在与可执行程序集相同位置写入的平台（macOS、移动端、WASM 等）上，它无法正确持久化；请使用第三方/自研方案为你的应用写入持久化配置数据。

### 文件系统访问

16. 请确保你的文件访问代码能够处理区分大小写的文件系统，并使用 `Path.DirectorySeparatorChar`，而不是硬编码目录分隔符。 

### 字体

17. 自定义字体必须在 `.csproj` 中作为 `<Resource>` 项包含。如果字体未作为资源嵌入，应用在非 Windows 平台上可能会崩溃或回退到默认字体：
    ```xml
    <ItemGroup>
        <Resource Include="Fonts\*.ttf" />
    </ItemGroup>
    ```
18. 字体匹配在 WPF 和 XPF 之间的工作方式不同。带有非标准样式名称的字体（例如用 "Condense" 而不是 "Condensed"）可能无法正确匹配。如果字体未按预期渲染，请确认 XAML 中的字体族名称与字体文件中的内部名称一致。
19. 若要自定义字体回退行为（例如指定缺失字符所使用的字体），请在你的 [自定义初始化](/xpf/configuration/customizing-initialization) 中配置 `FontManagerOptions`：
    ```csharp
    .With(new FontManagerOptions
    {
        FontFallbacks = new[]
        {
            new FontFallback { FontFamily = "My Fallback Font" }
        }
    })
    ```

### 不支持的控件

20. 避免使用 WPF 的拼写检查和 XPS 功能，因为 XPF 不支持这些功能。
21. 如果你的应用需要一些高级且专门的 WPF 功能，例如 Shaders、3D 和 Media，请联系 Avalonia 团队，以获取最佳实现方式的指导。