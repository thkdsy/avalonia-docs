---
id: android
title: 使用 Avalonia 开发 Android 应用
description: 配置用于构建 Avalonia 应用的 Android 开发环境，包括 SDK 与 workload 的安装。
doc-type: guide
---

## 配置开发环境

按照以下步骤使用 CLI 安装所需工具：

- 检查你是否安装了兼容版本的 .NET SDK。可与 Avalonia 配合使用的最低版本是 6.0.200。

:::info
你可以在这里查看 [可用的 .NET SDK 版本](https://dotnet.microsoft.com/en-us/download/dotnet)。
:::

- 你可能需要卸载旧版本的 _Android Workload_。可以执行以下命令：

```bash
dotnet workload uninstall android
```

- 安装 _Android Workload_。可以执行以下命令：

```bash
dotnet workload install android
```

:::info
你可能需要使用 _sudo_ 来执行上述命令。
:::

:::caution[Linux users: use the official Microsoft .NET SDK]
`dotnet workload` 命令要求使用微软官方的 .NET SDK。来自 Linux 发行版仓库的 .NET 包（例如 Arch Linux AUR、Ubuntu 的 `dotnet-sdk` apt 包，或 Fedora 的 `dotnet` dnf 包）可能不包含 workload 支持。如果 `dotnet workload install android` 因 `NETSDK1139` 错误而失败，请从 [微软 .NET 下载页](https://dotnet.microsoft.com/download) 安装 SDK，或使用 [安装脚本](https://learn.microsoft.com/dotnet/core/tools/dotnet-install-script)：

```bash
curl -sSL https://dot.net/v1/dotnet-install.sh | bash /dev/stdin --channel 10.0
```
:::

### 安装 Android SDK

安装 Android SDK 有多种方式，请选择与你的开发环境匹配的方案。

如果你使用 Visual Studio 或 Visual Studio for Mac，请按照 [Android SDK 安装指南](https://docs.microsoft.com/en-us/xamarin/android/get-started/installation/android-sdk) 进行操作。

如果你使用 _JetBrains Rider_，请参考 [Rider Xamarin 配置指南](https://www.jetbrains.com/help/rider/Xamarin.html)。

你也可以直接安装 [Android 命令行工具](https://developer.android.com/studio#command-tools)。

该工具集自带一个基于命令行的 SDK 管理器，可用于安装 SDK。成功安装 Android SDK 后，请把 SDK 路径加入到 PATH 环境变量中，可以直接在 bash 中设置，也可以写入 Linux 用户配置文件的 `.bashrc`。

```bash
export ANDROID_HOME=/path/to/sdk
export PATH=$PATH:$ANDROID_HOME/tools:$ANDROID_HOME/platform-tools
```

在构建、运行或部署 dotnet Android 项目时，你也可以在 `dotnet` 命令中直接指定 Android SDK 位置，只需设置 `AndroidSdkDirectory` 变量：

```bash
dotnet build ... /p:AndroidSdkDirectory=/path/to/sdk
```

请确保你已经通过系统包管理器安装了 JDK 11 或更高版本。如果你是通过上面提到的 Visual Studio 或 JetBrains Rider 完成环境配置，这一步通常已经自动完成。

还有一个仍在开发中的工具 _MAUI Check_，可以自动帮你安装所需的 SDK 和工具：

```bash
dotnet tool install -g Redth.Net.Maui.Check
maui-check
```

完成上述 _Android_ 开发环境配置后，你就可以构建 _Android_ 应用，并在本机平台的模拟器中运行它们。

## 另请参阅

- [部署到 Android](/docs/deployment/android)（模拟器、真机与发布）
- [在 Linux 上为 Visual Studio Code 配置 Android 调试](/tools/visual-studio-code/configure-vscode-debug-linux)
