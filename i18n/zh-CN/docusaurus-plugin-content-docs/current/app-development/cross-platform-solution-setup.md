---
index: cross-platform-solution-setup
title: 搭建跨平台解决方案
description: 使用共享核心项目和平台专属启动项目来组织 Avalonia 解决方案。
doc-type: explanation
---

尽管目标平台各不相同，Avalonia 项目仍然统一使用同一种解决方案文件格式（Visual Studio 的 `.SLN` 格式）。这意味着同一个解决方案可以在不同开发环境之间共享，为多平台应用开发提供一致的组织方式。

创建跨平台应用的第一步是先建立一个解决方案。本节将进一步说明后续步骤，也就是如何组织这些项目，以便使用 Avalonia 构建跨平台应用。

## 填充解决方案结构

`Avalonia Cross Platform Application` 模板会创建一套包含下列项目的解决方案结构，以便在多个平台之间共享和复用代码：

:::info
[Ensure you've installed the Avalonia Templates.](/docs/get-started/install-avalonia)
:::

### Core project
这是应用的核心部分，设计目标是与平台无关。它包含应用中所有可复用的组成部分，例如业务逻辑、视图模型和视图。其他所有项目都会引用这个核心项目。大多数开发工作通常都应集中在这里完成。

### Desktop project
这个项目让应用能够在 Windows、macOS 和 Linux 上运行，其输出类型为 `WinExe`。

### Android project
这是一个基于 `NET-Android` 的项目，并引用核心项目。它包含一个继承自 `AvaloniaMainActivity` 的 `MainActivity`，作为 Android 应用的入口点。

### iOS project
这是一个面向 iOS 和 iPadOS 平台的 `NET-iOS` 项目。该项目的入口点是 `AppDelegate`，它继承自 `AvaloniaAppDelegate`。

### Browser project
这个 WebAssembly（WASM）项目允许你的 Avalonia 应用在浏览器中运行。它的 `RuntimeIdentifier` 是 `browser-wasm`。

## 核心项目

共享代码项目应只引用所有平台都可用的程序集。通常这包括 `System`、`System.Core`、`System.Xml` 等通用框架命名空间。

这些共享项目的目标是尽可能实现更多应用功能，包括 UI 组件，从而最大化代码复用率。

通过将功能拆分到不同层中，代码会更容易管理、测试和在多个平台之间复用。这种分层架构方式能够提升 Avalonia 项目的开发效率与可扩展性。

## 平台专属应用项目

平台专属项目必须引用核心项目。这些项目的作用是让应用能够在 iOS、Android 和 WASM 等特定平台上运行。

虽然桌面平台通常可以共用一个项目，但有时也适合为 macOS 单独创建项目，并使用 [Xamarin.Mac Target Framework](https://learn.microsoft.com/en-us/xamarin/mac/platform/target-framework)。这样会更方便应用的分发与打包。

## 另请参阅

- [Cross-Platform Architecture](/docs/fundamentals/cross-platform-architecture)：解决方案结构与平台分支模式。
