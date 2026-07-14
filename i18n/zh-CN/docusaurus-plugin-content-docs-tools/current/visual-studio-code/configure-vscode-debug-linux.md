---
id: configure-vscode-debug-linux
title: 在 Linux 上配置 Android 调试
sidebar_label: Android 调试（Linux）
description: 在 Linux 上配置 Visual Studio Code，以使用 Mono 调试器构建、部署并调试 Avalonia Android 项目。
doc-type: how-to
---

# 在 Linux 上配置 Android 调试

本指南将带你在 Linux 上配置 Visual Studio Code，以便构建、部署并调试基于 Avalonia 的 Android 项目。该工作流使用 Mono Debug 扩展，通过本地端口附加到正在运行的 Android 应用。

## 前置条件

开始之前，请确认你具备以下条件：

- 已在 Linux 上安装 Visual Studio Code。
- 已安装 .NET SDK（6.0 或更高版本），并且可通过 `PATH` 访问。
- 正在运行的 Android 模拟器，或一台通过 USB 连接并已启用开发者模式的 Android 真机。
- 已从 [Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=ms-vscode.mono-debug) 安装 **Mono Debug** 扩展。

## 配置启动配置文件

在工作区中打开（或创建）`.vscode/launch.json` 文件，并添加两个配置：一个在附加前先构建和部署，另一个用于附加到已运行的应用。

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug - Android",
      "type": "mono",
      "preLaunchTask": "run-debug-android",
      "request": "attach",
      "address": "localhost",
      "port": 10000
    },
    {
      "name": "Attach - Android",
      "type": "mono",
      "request": "attach",
      "address": "localhost",
      "port": 10000
    }
  ]
}
```

`port` 的值可以是任意未被其他应用程序或操作系统占用的端口。上面的示例使用 `10000`。

- **Debug - Android** 会先运行预启动任务来构建并部署应用，然后附加调试器。
- **Attach - Android** 会跳过构建步骤，直接连接到已经在设备或模拟器上运行的应用。

## 配置构建任务

打开（或创建）`.vscode/tasks.json` 文件，并添加一个任务，用于在启用 Mono 调试服务器的情况下构建并部署你的 Android 项目。

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "run-debug-android",
      "command": "dotnet",
      "type": "shell",
      "args": [
        "build",
        "--no-restore",
        "-t:Run",
        "${workspaceFolder}/<ProjectName>.Android.csproj",
        "-p:TargetFramework=net6.0-android",
        "-p:Configuration=Debug",
        "-p:AndroidAttachDebugger=true",
        "-p:AndroidSdbHostPort=10000",
        "-p:AndroidSdbTargetPort=10000"
      ],
      "problemMatcher": "$msCompile"
    }
  ]
}
```

请将 `<ProjectName>` 替换为你 Android 专用 Avalonia 项目的名称。

:::info
`launch.json` 中的 `port` 值必须与 `tasks.json` 中的 `AndroidSdbHostPort` 和 `AndroidSdbTargetPort` 保持一致。如果这些值不同，调试器将无法连接。
:::

## 开始调试

1. 在 Visual Studio Code 中打开 **Run and Debug** 面板（Ctrl+Shift+D）。
2. 从配置下拉框中选择 **Debug - Android**。
3. 按下 **F5**，或点击绿色播放按钮。

.NET 运行时会将你的应用构建并部署到已连接的设备或模拟器上。应用启动后，Mono 调试器会附加到配置的端口，你就可以像平常一样设置断点、检查变量并单步执行代码。

如果你的应用已经在设备上运行，则改为选择 **Attach - Android**，以跳过构建步骤并直接连接。

## 另请参阅

- [IDE 支持](/tools/ide/)
- [Avalonia tools 概览](/tools/)
- [Visual Studio 扩展](/tools/visual-studio-extension)
