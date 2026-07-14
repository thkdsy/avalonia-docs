---
id: windows
title: Windows 部署
description: 了解如何在 Windows 上发布、打包和部署您的 Avalonia XPF 应用程序，包括独立部署、单文件发布和安装程序选项。
doc-type: guide
---

## 发布

您可以使用标准 .NET CLI 将 XPF 应用程序发布到 Windows。要创建依赖框架的部署（需要目标计算机上安装 .NET 运行时），请运行：

```bash
dotnet publish -r win-x64 -c Release
```

对于包含 .NET 运行时、因此您的用户无需单独安装它的独立部署，请添加 `--self-contained` 标志：

```bash
dotnet publish -r win-x64 -c Release --self-contained
```

如果您的目标是 ARM64 Windows 设备，请改用 `win-arm64` 运行时标识符：

```bash
dotnet publish -r win-arm64 -c Release --self-contained
```

## 单文件发布

XPF 支持在 Windows 上进行单文件发布。这会将您的应用程序及其依赖项打包为一个可执行文件。将以下属性添加到您的项目文件中：

```xml
<PropertyGroup>
    <PublishSingleFile>true</PublishSingleFile>
    <SelfContained>true</SelfContained>
    <RuntimeIdentifier>win-x64</RuntimeIdentifier>
</PropertyGroup>
```

:::caution
当您使用单文件发布时，`Assembly.GetEntryAssembly().Location` 会返回空字符串。请改用 `AppDomain.CurrentDomain.BaseDirectory` 来获取应用程序目录。
:::

## ReadyToRun 编译

您可以启用 ReadyToRun（R2R）预先生成编译，以减少应用程序的启动时间。将以下属性添加到您的项目文件中：

```xml
<PropertyGroup>
    <PublishReadyToRun>true</PublishReadyToRun>
</PropertyGroup>
```

ReadyToRun 会将托管程序集预编译为本机代码，这意味着 JIT 编译器在启动时需要做的工作更少。代价是发布输出的体积会更大。

有关更多详细信息，请参阅 [性能优化](/xpf/configuration/performance#reducing-startup-time-with-readytorun)。

## WinForms 托管

如果您的应用程序托管了 WinForms 控件，请在项目文件中的 Windows 条件属性组里添加以下属性：

```xml
<PropertyGroup Condition="$([MSBuild]::IsOSPlatform('Windows'))">
    <XpfUseMicrosoftWindowsForms>true</XpfUseMicrosoftWindowsForms>
</PropertyGroup>
```

将 `XpfUseMicrosoftWindowsForms` 设置为 `true` 会禁用 WinForms shim 层并启用原生 WinForms 集成。此选项仅在 Windows 上可用，这也是为什么需要条件判断保护的原因。

## STA 线程

某些 Windows API（尤其是剪贴板操作和 COM 互操作）要求主线程使用单线程单元（STA）模型。如果您遇到消息为 "CoInitialize was not called" 的 `COMException`，请确保您的入口点使用了 `[STAThread]` 属性：

```csharp
[STAThread]
public static void Main(string[] args)
{
    // Your application startup code
}
```

当您使用 [自定义初始化](/xpf/configuration/customizing-initialization) 时，XPF SDK 会自动处理 STA 线程。

## Windows 安装程序

您发布的 XPF 应用程序是标准的 .NET 应用程序，因此可以使用任何 Windows 安装程序技术对其进行打包。常见选项包括：

- **MSIX**：现代 Windows 打包格式，支持自动更新以及干净的安装/卸载。
- **WiX Toolset**：一个开源安装程序制作框架，用于创建 MSI 和 MSIX 包。
- **Inno Setup**：一个免费且广泛使用的 Windows 应用程序安装包生成器。
- **NSIS**：一个可脚本化的安装系统，拥有庞大的插件生态系统。

## 另请参见

- [macOS 部署](/xpf/deployment/macos)
- [Linux 部署](/xpf/deployment/linux)
- [自定义初始化](/xpf/configuration/customizing-initialization)
- [性能优化](/xpf/configuration/performance)