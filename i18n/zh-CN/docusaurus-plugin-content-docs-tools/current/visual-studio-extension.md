---
id: visual-studio-extension
title: Avalonia Visual Studio 扩展
description: 使用 Avalonia Visual Studio 扩展来增强 XAML 编辑体验，包括 IntelliSense、错误高亮和内置 UI 预览器。
sidebar_label: Visual Studio 扩展
doc-type: reference
---

import TestXamlPreviewer from '/img/guides/ui-development/xaml-preview-and-design-settings/test-xaml-previewer.png';

## 功能

Avalonia Visual Studio 扩展为使用 Avalonia XAML 文件提供了更强大的工作方式。它主要通过以下能力实现这一点：

- 一个深度集成的增强型编辑器，带来更丰富的编辑体验。
- 一个预览器，让你无需运行应用程序即可看到 UI 的外观。

## 安装

安装说明请参阅 [设置你的 IDE](/docs/get-started/set-up-your-ide)。

### 增强型编辑器

Avalonia XAML 编辑器包含以下能力：

- 输入时提供更智能、更有帮助的 Intellisense。
- 错误高亮和修复建议。
- 自动导入 XAML 命名空间。
- 完整的 XAML 语法着色。
- “Go To Definition” 导航。
- 智能悬停提示。
- 自动文档格式化。
- 文档大纲支持，可折叠元素。

### 预览器

预览器允许你在不运行应用程序的情况下，查看当前打开文档的 UI 外观。

更多信息请参阅 [预览你的 UI 设计](/docs/app-development/xaml-preview-and-design-settings)。

<Image light={TestXamlPreviewer} alt="A screenshot demonstrating a test of the Avalonia XAML previewer." maxWidth={400} cornerRadius="true"/>

## 设置

这里提供了多个选项，用于配置编辑器和预览器的行为方式。

你可以在 Visual Studio 中通过 **Tools** 菜单里的 **Options** 访问这些设置。

![Options dialog](/img/vs-extension/visual-studio-avalonia-options.png)

| 设置 | 说明 | 可选值 |
|-----------------------|-------------|---------------|
| Default Document View | 打开文档时显示的内容 | Split（默认）- 同时显示代码和预览器<br />Design - 只显示预览器<br />Source - 只显示源代码 |
| Split Orientation | 以水平还是垂直方向进行分栏 | Horizontal（默认）- 编辑器和预览器左右并排显示<br />Vertical - 编辑器和预览器上下显示 |
| Swapped | 在 “Split” 模式下打开文档时，是否交换编辑器和预览器的默认位置 | 勾选时为 True |
| Default Zoom level | 如何调整预览内容的显示大小 | 100%（默认）<br />50%、75%、100%、125%、150%、200%<br />Fit to Width - 让预览占满可用宽度<br />Fit All - 填满整个预览器 |
| Minimum Log Verbosity | 扩展输出信息时允许的最小 LogLevel | Trace<br />Debug<br />Information（默认）<br />Warning<br />Error<br />Critical<br />None |

## 另请参阅

- [IDE 支持](/tools/ide/)
- [Avalonia 工具概览](/tools/)
