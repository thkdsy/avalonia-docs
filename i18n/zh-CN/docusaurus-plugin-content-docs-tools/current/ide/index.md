---
id: index
title: IDE 支持
doc-type: overview
---

Avalonia 可以与您已经在使用的 .NET IDE 无缝配合。无论你偏好 Visual Studio、VS Code 还是 Rider，都可以开箱即用地获得 IntelliSense、调试和项目支持。不同之处主要体现在 XAML 编辑体验上，尤其是预览、代码补全和设计器工具方面。

## Visual Studio

[Avalonia for Visual Studio](/tools/visual-studio-extension) 扩展是 Avalonia Plus 的一部分，可提供完整的 XAML 编辑体验。它包括实时预览器、带自动命名空间导入的智能代码补全、带修复建议的错误高亮、拖放式设计器以及完整的 XAML 语法着色。

如果你在 Windows 上开发，这个扩展能提供最完整的 XAML 编辑和预览支持。

## Visual Studio Code

Avalonia for Visual Studio Code 扩展构建于与 Visual Studio 扩展相同的 XAML 解析器之上，这意味着两个 IDE 共享同一套底层引擎。Visual Studio 中提供的各类代码编辑增强能力，也会直接延续到 VS Code 中。

该扩展提供丰富的 IntelliSense 与上下文补全、完整的 `x:DataType` Quick Info（便于通过绑定检查数据上下文）、XAML 的 Go to Definition、自动命名空间导入、事件处理器生成，以及清晰且可操作的诊断信息。它还包含一个稳定可靠的 XAML 预览器，具备正确的 DPI 处理与 Zoom to Fit 支持。

## JetBrains Rider

Rider 为 Avalonia 开发提供了优秀的 .NET 支持，包括项目管理、调试和代码导航。Rider 本身不自带 Avalonia XAML 预览器，但社区维护的 [AvalonRider](https://plugins.jetbrains.com/plugin/14839-avalonrider) 插件可以直接在 IDE 中补充预览支持。

## 另请参阅

- [Avalonia for Visual Studio](/tools/visual-studio-extension)
- [AI 工具](/tools/ai-tools/)
- [Avalonia 工具概览](/tools/)
