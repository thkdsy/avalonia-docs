---
id: installation
title: 安装 Avalonia Plus Developer Tools
sidebar_label: Installation
sidebar_position: 1
doc-type: tutorial
tags:
  - avalonia plus
  - avalonia pro
  - avalonia enterprise
---

在本教程中，你将安装 Avalonia Plus Developer Tools，为项目添加诊断支持包，并验证应用与工具之间的连接。

## 前提条件

### Developer Tools 要求

| 要求 | 版本/说明 |
|------------|-----------------|
| .NET Runtime | 6.0 或更高版本 |
| Windows | 10 或更高版本 |
| macOS | 13 或更高版本 |
| Linux | 兼容 X11 且使用 glibc 2.27 或 musl 1.22.2 的发行版 |

运行该工具不需要管理员或 sudo 权限。如果你计划远程使用 Developer Tools，可能需要配置防火墙例外规则。

### Diagnostics Support 要求

支持包要求 **Avalonia 11.2.0** 或更高版本，并基于兼容 **.NET Standard 2.0** 的 API 构建。

该包兼容 Browser 与 Android/iOS 项目。

:::note

已预配置 Developer Tools 的演示项目可见 [AvaloniaUI/AvaloniaUI.DeveloperTools/samples/SimpleToDoList](https://github.com/AvaloniaUI/AvaloniaUI.DeveloperTools/tree/main/samples/SimpleToDoList#simpletodolist)。

:::

## 第 1 步：安装 AvaloniaUI Developer Tools

AvaloniaUI Developer Tools 当前是一个原生 [.NET 工具](https://learn.microsoft.com/en-us/dotnet/core/tools/global-tools)，其更新机制由 SDK 提供。
本指南演示的是全局安装方式。不过也可以进行本地安装，但有一个限制：该工具只能在与工具安装所在解决方案/项目相同的工作目录，或其子目录中运行。

<Tabs>
<TabItem value="net10" label=".NET 10+" default>

```bash
dotnet tool install --global AvaloniaUI.DeveloperTools
```

如果你是从 .NET 8/9 的安装方式升级而来，应先使用 `dotnet tool uninstall --global AvaloniaUI.DeveloperTools.Windows` 或 `avdt uninstall` 将其卸载。

之后可通过运行 `dotnet tool update` 命令更新 Developer Tools。

<details>
<summary>Developer Tools 更新命令</summary>

```bash
dotnet tool update --global AvaloniaUI.DeveloperTools
```

</details>


</TabItem>
<TabItem value="net8" label=".NET 8/9">

如果你使用的是早于 10 的 .NET SDK，则必须根据运行平台安装对应的特定包。

**Windows:**

```bash
dotnet tool install --global AvaloniaUI.DeveloperTools.Windows
```

**macOS:**

```bash
dotnet tool install --global AvaloniaUI.DeveloperTools.macOS
```

**Linux:**

```bash
dotnet tool install --global AvaloniaUI.DeveloperTools.Linux
```

之后可通过运行 `dotnet tool update` 命令更新 Developer Tools。

<details>
<summary>Developer Tools 更新命令</summary>

**Windows:**

```bash
dotnet tool update --global AvaloniaUI.DeveloperTools.Windows
```

**macOS:**

```bash
dotnet tool update --global AvaloniaUI.DeveloperTools.macOS
```

**Linux:**

```bash
dotnet tool update --global AvaloniaUI.DeveloperTools.Linux
```

</details>

</TabItem>
</Tabs>

## 第 2 步：安装 Diagnostics Support 包

`Diagnostics Support` 包负责在用户应用与 Developer Tools 进程之间建立连接桥梁。

根据应用架构的不同，该包既可以安装在包含 Program AppBuilder 的可执行项目中，也可以安装在包含 Application 的共享项目中。

两种情况下使用的命令相同：

```bash
dotnet add package AvaloniaUI.DiagnosticsSupport
```

:::note

旧包 `Avalonia.Diagnostics` 可以安全移除，新的 `Developer Tools` 已不再使用它。

:::

## 第 3 步：配置项目

安装 `DiagnosticsSupport` 包后，你需要在 `Application` 类中启用它：

```csharp
public override void Initialize()
{
    AvaloniaXamlLoader.Load(this);

#if DEBUG
    this.AttachDeveloperTools();
#endif
}
```

或者，也可以在 AppBuilder 上使用 `.WithDeveloperTools()` 扩展方法。

这些方法还接受 `DeveloperToolsOptions` 选项类，从而允许你自定义 `Diagnostics Support` 的配置。详情请参阅 [DeveloperToolsOptions 参考](/tools/developer-tools/options)。

默认使用端口 **29414**，通常该端口是可用的。你也可以通过选项进行配置。

## 第 4 步：运行工具

当目标应用运行后，按下 <kbd>F12</kbd> 初始化连接。
`Diagnostics Support` 会自动运行 `Developer Tools` 可执行文件，并在进程之间建立连接。
在 `macOS` 上首次运行时，可能会因 Gatekeeper 验证而耗费数秒；后续启动会更快。

## 第 5 步：激活工具

打开 Developer Tools 后，系统会要求你输入用于授权该工具的 `AvaloniaUI Portal` 凭据。这是该工具唯一需要联网的时候。之后即可离线使用，直到许可证密钥会话过期。

![Tool Activation](/img/tools/dev-tools/tool-activation.png)

## 第 6 步：完成！

激活完成后，与应用的连接会恢复，并打开包含各种工具的窗口。

## 另请参阅

- [Elements 工具](/tools/developer-tools/elements-tool) 文档
- 自定义 [DeveloperToolsOptions 配置](/tools/developer-tools/options) 参考
- [模型上下文协议（MCP）](/tools/developer-tools/mcp)
- [常见问题](/tools/faq)
- [设置](/tools/developer-tools/settings)
- [快捷键](/tools/developer-tools/shortcuts)
- [附加到浏览器或移动应用](/tools/developer-tools/attaching-applications)
- [附加到远程工具](/tools/developer-tools/attaching-to-the-remote-tool)
- [问题报告](/troubleshooting/tools/developer-tools)
