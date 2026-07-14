---
id: missing-features
title: 缺失的功能
description: Avalonia XPF 中不可用、受限或计划在未来支持的 WPF 功能摘要。
doc-type: reference
---

## 概述

XPF 会尽最大努力实现所有 WPF API，但由于跨平台限制或底层架构差异，某些功能很难或无法支持。本页列出了当前缺失、部分实现或不太可能添加的功能，以便你据此规划迁移。

## 有限制的功能

以下功能可用，但存在已知限制：

- **`WebBrowser`**：XPF 通过不同的机制提供网页内容嵌入。详情请参阅 [WebView 文档](/docs/app-development/embedding-web-content)。
- **`FlowDocument`**：支持基本的流文档渲染，但有以下限制：
  - 不支持分页文档。
  - 不支持 `PageHeader` 和 `PageFooter`。
  - 不支持 `Floater`。
  - 不支持某些表格功能，例如行跨度和列跨度。

## 计划在未来版本中支持的功能

以下功能需要大量工程投入，并将在后续版本中提供：

- `Viewport3D` 及相关 3D API
- `MediaElement` 和 `MediaPlayer`
- `InkCanvas`

如果你的应用依赖这些功能中的任何一个，请定期查看 [发行说明](/xpf/version-info/release-notes) 以了解其进展更新。

## 不太可能支持的功能

由于平台限制，以下功能不太可能得到支持：

- **多个 UI 线程**（多个 `Dispatcher` 实例）：macOS 只允许一个 UI 线程，而 Windows 和 Linux 的支持有限。更多详情请参阅 [与 WPF 的已知差异](/xpf/migration/known-differences#multiple-ui-threads)。
- **`HwndHost` / `HwndSource`**：这些类型与 Win32 窗口句柄紧密耦合，且没有跨平台等价物。
- **`XPS`**：XPS 文档支持依赖于 Windows 操作系统组件，而该组件在其他平台上不可用。

## 解决方法

如果你的应用依赖某个缺失或受限的功能，请考虑以下策略：

- **条件编译**：使用 `#if` 指令为 XPF 构建和 WPF 构建提供不同的实现。
- **运行时功能检测**：在使用某个功能之前检查其是否可用，并提供优雅的回退方案。
- **联系 XPF 团队**：如果某个缺失功能对你的应用至关重要，请联系 Avalonia 团队。功能优先级通常会受到客户需求的影响。

## 另请参阅

- [与 WPF 的已知差异](/xpf/migration/known-differences)
- [发行说明](/xpf/version-info/release-notes)
- [版本控制](/xpf/version-info/versioning)