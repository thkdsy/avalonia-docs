---
id: welcome
title: 欢迎
description: 开始使用 Avalonia 跨平台 .NET UI 框架。查找安装指南、教程、迁移路径和 API 参考。
doc-type: overview
---

<head>
  <title>Avalonia 文档</title>
  <meta
    name="description"
    content="Avalonia 跨平台 .NET UI 框架文档。使用单一代码库为 Windows、macOS、Linux、iOS、Android 和 WebAssembly 构建应用。"
  />
</head>

欢迎来到 Avalonia 文档。无论你是在构建第一个应用，还是准备迁移现有项目，这套文档都涵盖了从安装到部署的完整内容。

:::info

这些文档针对的是 Avalonia 12。如果你需要 Avalonia 11 文档，请访问 [v11.docs.avaloniaui.net](https://v11.docs.avaloniaui.net/)。

:::

## 什么是 Avalonia？

Avalonia 是一个用于 .NET 应用开发的开源跨平台 UI 框架。它使用自己的渲染引擎来绘制控件，因此你的应用在每个平台上的外观和行为都能保持一致。你可以使用 C# 或 F# 配合 XAML 编写一次 UI，然后部署到：

- **Windows**（10、11）
- **macOS**（Apple Silicon 和 Intel）
- **桌面 Linux**（X11 和 Wayland）
- **嵌入式 Linux**（例如 Raspberry Pi 等设备上的 framebuffer）
- **iOS** 和 **Android**
- **WebAssembly**

有关确切版本和架构细节，请参阅[支持的平台](/docs/supported-platforms)。

## 核心能力

| 能力 | 说明 |
|---|---|
| **跨平台渲染** | Avalonia 自带渲染引擎，可在所有平台上输出像素一致的界面。Skia 是默认后端，团队也正在与 Google Flutter 团队合作，将 [Impeller](https://avaloniaui.net/blog/avalonia-partners-with-google-s-flutter-t-eam-to-bring-impeller-rendering-to-net) 渲染引擎带到 .NET。无需原生控件包装层，也没有平台特有的怪异行为。 |
| **XAML 与代码后置** | 你可以使用 XAML 以声明式方式描述 UI，也可以完全用代码构建界面。如果你用过 WPF 或 UWP，会觉得 Avalonia XAML 很熟悉。 |
| **样式系统** | 一个受 CSS 启发的样式系统，支持选择器、样式类、伪类和控件主题。参阅 [样式](/docs/styling/styles)。 |
| **数据绑定** | 支持构建时校验的编译绑定、完整 MVVM 模式，以及与 CommunityToolkit.Mvvm 的集成。参阅 [数据绑定](/docs/data-binding/introduction-to-data-binding)。 |
| **丰富控件库** | 提供 60+ 内置控件，包括 DataGrid、TreeView、TabControl、Calendar 等，且都支持样式和模板定制。 |
| **无障碍支持** | 内置支持屏幕阅读器和跨平台键盘导航。 |
| **DevTools** | 运行时按 <kbd>F12</kbd> 可检查可视树、属性、样式和布局。 |

## 选择你的路径

### 初次接触 Avalonia？

1. [安装 Avalonia](/docs/get-started/install-avalonia)，并[配置你的 IDE](/docs/get-started/set-up-your-ide)
2. [创建你的第一个项目](/docs/get-started/create-your-first-project)
3. [跟随入门教程](/docs/get-started/starter-tutorial) 构建一个温度转换器应用
4. [学习基础概念](/docs/fundamentals/avalonia-xaml)：XAML、控件、布局和可视树

### 来自 WPF？

Avalonia 的 API 有意保持与 WPF 接近，但在样式、模板和属性系统方面仍有一些重要差异。

- [WPF 迁移指南](/docs/migration/wpf)：逐章节对比说明
- [WPF 速查表](/docs/migration/wpf/cheat-sheet)：快速查看 WPF 概念与 Avalonia 对应关系

如果你希望在不重写现有 WPF 应用的前提下让它跨平台运行，那么 [Avalonia XPF](/xpf) 可在 Avalonia 渲染引擎之上提供二进制兼容的 WPF 支持。

### 从 Avalonia 11 升级？

Avalonia 12 默认启用了编译绑定，引入了新的剪贴板 API，并更新了窗口装饰等多项内容。

- [Avalonia 12 中的破坏性变更](/docs/avalonia12-breaking-changes)：完整变更列表及逐项迁移说明

### 想找示例？

- [示例与教程](/docs/samples-tutorials)：入门应用、真实案例和视频讲解

## 需要帮助？

如果你遇到问题，可以先查看[故障排查](/troubleshooting)页面，或者前往 [GitHub Discussions](https://github.com/AvaloniaUI/Avalonia/discussions) 与社区交流。

如果要报告 bug，请在 [GitHub](https://github.com/AvaloniaUI/Avalonia) 上提交 issue。

## 另请参阅

- [支持的平台](/docs/supported-platforms)
- [示例与教程](/docs/samples-tutorials)
- [Avalonia GitHub 仓库](https://github.com/AvaloniaUI/Avalonia)
