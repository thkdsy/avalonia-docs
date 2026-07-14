---
id: installing-avalonia-pro
title: 安装 Avalonia Pro
description: 配置项目以使用 Avalonia Pro NuGet 包，并添加许可证密钥。
doc-type: how-to
tags:
  - avalonia pro
  - avalonia enterprise
---

本指南说明如何配置项目以使用 Avalonia Pro 包。这些包是 [Avalonia Pro 或 Enterprise](https://avaloniaui.net/pricing) 的一部分。

## 前置条件

开始之前，请确认你具备以下条件：

- 一个目标框架为 .NET 8 或更高版本的 Avalonia 项目。
- 一个有效的 Avalonia 许可证密钥。你可以从 [Avalonia portal](https://portal.avaloniaui.net) 获取。

## NuGet 包源

Avalonia Pro 包通过 [nuget.org](https://www.nuget.org/) 分发，不需要额外配置 NuGet 源。

:::note
在 2025 年 10 月 13 日之前，安装 Avalonia Pro 组件需要配置专用 NuGet 源。现在已不再需要该源，请直接使用 [nuget.org](https://www.nuget.org/)。
:::

## 添加 NuGet 包

通过运行 `dotnet add package` 命令安装所需的 Avalonia Pro 包。例如，添加媒体播放器控件：

```bash
dotnet add package Avalonia.Controls.MediaPlayer
```

将包名替换为你实际需要的名称。目前可用的 Avalonia Pro 包包括：

| Package | Description |
|---------|-------------|
| `Avalonia.Controls.MediaPlayer` | 音频与视频播放控件 |
| `Avalonia.Controls.WebView` | 原生 Web 内容嵌入 |
| `Avalonia.Controls.Markdown` | Markdown 文本渲染 |
| `Avalonia.Controls.TreeDataGrid` | 分层和扁平数据网格 |

## 添加许可证密钥

将你的 Avalonia 许可证密钥添加到可执行项目文件（`.csproj`）中：

```xml
<ItemGroup>
  <AvaloniaUILicenseKey Include="YOUR_LICENSE_KEY" />
</ItemGroup>
```

将 `YOUR_LICENSE_KEY` 替换为你在 [Avalonia portal](https://portal.avaloniaui.net) 账户中看到的密钥。

对于多项目解决方案，你可以将许可证密钥存放在[环境变量](https://learn.microsoft.com/en-us/visualstudio/msbuild/how-to-use-environment-variables-in-a-build)或[共享 props 文件](https://learn.microsoft.com/en-us/visualstudio/msbuild/customize-by-directory?view=vs-2022#directorybuildprops-example)中，以避免重复。

:::warning
不要将许可证密钥留空。若值为空，你可能无法构建或打开项目。
:::

## 验证安装

构建项目，以确认包已正确还原并且许可证密钥已被接受：

```bash
dotnet build
```

如果许可证密钥缺失或无效，你会看到构建警告。请检查 `<AvaloniaUILicenseKey>` 元素是否位于正确的项目文件中，以及其值是否与 portal 账户中显示的一致。

## 另请参阅

- [Avalonia Tools 概览](/tools/)
- [FAQ](/tools/faq)
