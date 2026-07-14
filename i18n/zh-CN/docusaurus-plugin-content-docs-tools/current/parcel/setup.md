---
id: setup
title: 配置 Avalonia Parcel
description: 安装、配置并激活 Avalonia Parcel。它是一个用于在 Windows、macOS 和 Linux 上构建、签名与打包 Avalonia 应用程序的工具。
sidebar_label: Setup
sidebar_position: 1
doc-type: tutorial
tags:
  - avalonia plus
  - avalonia pro
  - avalonia enterprise
---

Avalonia Parcel 是一个面向 Avalonia 应用程序的打包工具。它被设计为一个双应用方案（GUI 和控制台工具），可在 Windows、macOS 和 Linux 平台上处理应用程序的构建、签名和打包。

## 前提条件

| 要求 | 版本/说明 |
|------------|-----------------|
| .NET Runtime | 6.0 或更高版本 |
| Windows | 10 或更高版本 |
| macOS | 13 或更高版本 |
| Linux | 兼容 X11 且使用 glibc 2.27 或 musl 1.22.2 的发行版 |

## 第 1 步：安装 Avalonia Parcel

`Avalonia Parcel` 是一个原生 [.NET 工具](https://learn.microsoft.com/en-us/dotnet/core/tools/global-tools)，其更新机制由 SDK 提供。
本指南演示的是全局安装方式。本地安装同样可行，但请注意：该工具只能在它所安装到的项目中使用。

<Tabs>
<TabItem value="net10" label=".NET 10+" default>

```bash
dotnet tool install --global AvaloniaUI.Parcel
```

如果你是从 .NET 8/9 的安装方式升级而来，应先使用 `dotnet tool uninstall --global AvaloniaUI.Parcel.Windows` 或 `parcel uninstall` 将其卸载。

之后可通过运行 `dotnet tool update` 命令更新 Parcel。

<details>
<summary>Parcel 更新命令</summary>

```bash
dotnet tool update --global AvaloniaUI.Parcel
```

</details>


</TabItem>
<TabItem value="net8" label=".NET 8/9">

如果你使用的是早于 10 的 .NET SDK，则必须根据运行平台安装对应的特定包。

**Windows:**

```bash
dotnet tool install --global AvaloniaUI.Parcel.Windows
```

**macOS:**

```bash
dotnet tool install --global AvaloniaUI.Parcel.macOS
```

**Linux:**

```bash
dotnet tool install --global AvaloniaUI.Parcel.Linux
```

之后可通过运行 `dotnet tool update` 命令更新 Parcel。

<details>
<summary>Parcel 更新命令</summary>

**Windows:**

```bash
dotnet tool update --global AvaloniaUI.Parcel.Windows
```

**macOS:**

```bash
dotnet tool update --global AvaloniaUI.Parcel.macOS
```

**Linux:**

```bash
dotnet tool update --global AvaloniaUI.Parcel.Linux
```

</details>

</TabItem>
</Tabs>

## 第 2 步：运行工具

安装完成后，你可以通过终端运行以下命令来启动它：

```bash
parcel
```

该命令会启动一个 GUI 应用程序，你可以在其中打开或创建 parcel 项目。

或者，你也可以在终端中针对现有的 parcel 项目运行 CLI 命令：

```bash
parcel pack ./SampleApp.parcel -r osx-x64 -p dmg -o ./artifacts
```

该命令会基于预先配置好的 parcel 项目，将应用打包、签名，并生成一个 dmg 文件。

:::note
CLI 在免费 Community 许可证中不可用。
:::
## 第 3 步：激活工具
打开 Parcel 后，系统会要求你输入用于授权该工具的 `AvaloniaUI Portal` 凭据。
对于 CLI，你可以使用 `--licenseKey` 选项，或设置 `PARCEL_LICENSE_KEY` 环境变量。
## 延伸阅读
- [Parcel 命令行参考](/tools/parcel/command-line-reference)
- [模型上下文协议（MCP）](/tools/parcel/mcp)
- [Windows 打包](/tools/parcel/packaging-for-windows)
- [macOS 打包](/tools/parcel/packaging-for-macos)
- [Linux 打包](/tools/parcel/packaging-for-linux)
