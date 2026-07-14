---
title: Windows Forms
description: 将 Windows Forms 应用迁移到 Avalonia，以获得跨平台支持和现代化 UI。
doc-type: migration
---

Windows Forms 自 2002 年以来就一直支撑着生产环境应用。如果你手头有一个运行稳定的 WinForms 应用，那么继续让它运转下去当然有充分理由。但 Windows Forms 只能运行在 Windows 上，而且它所能提供的体验，与用户对现代 UI 的期待之间的差距也在逐年扩大。

Avalonia 为你提供了一条向前演进的路径。它是一个跨平台的 .NET UI 框架，具备基于 XAML 的布局系统、数据绑定、样式系统以及完整控件库，可运行在 Windows、macOS、Linux、iOS、Android 和 WebAssembly 上。如果你一直在使用 WinForms 开发，并开始思考下一步该怎么走，那么 Avalonia 正是为这类开发者准备的。

:::tip[迁移需要帮助？]
Avalonia 团队拥有将 Windows Forms 应用迁移到 Avalonia 的丰富经验。如果你希望获得专家协助，而不是独自推进，我们可以提供这项服务。更多信息请参阅 [Avalonia Services](https://avaloniaui.net/services)。
:::

## 会发生哪些变化

Windows Forms 与 Avalonia 在本质上属于完全不同的 UI 模型。不同于 XAML 框架之间的迁移（例如 WPF 到 Avalonia），这里大多数概念并不存在一一对应的映射关系。你应该预期的是学习新的模式，而不是简单翻译旧有写法。

| Windows Forms | Avalonia | 说明 |
|---|---|---|
| 设计器生成布局 | XAML 声明式布局 | 布局定义在 `.axaml` 文件中，而不是生成代码 |
| `Control` 基类 | `Control` / `TemplatedControl` | Avalonia 区分无模板控件与模板控件 |
| 几乎所有逻辑都靠事件处理 | 数据绑定 + MVVM | Avalonia 强烈鼓励将 UI 与逻辑分离 |
| `Dock` 和 `Anchor` 定位 | 布局面板（`Grid`、`StackPanel`、`DockPanel`） | 基于面板的布局取代锚点/停靠定位 |
| `DataGridView` | `DataGrid`（NuGet 包） | 独立包：`Avalonia.Controls.DataGrid` |
| `ToolStrip` / `MenuStrip` | `Menu` / `ToolTip` / `ContextMenu` | 控件名不同，但概念相同 |
| `Form` | `Window` | |
| [`UserControl`](/api/avalonia/controls/usercontrol) | `UserControl` | 名称相同，但基类体系不同 |
| `MessageBox.Show()` | 对话框窗口或自定义覆盖层 | 没有内置消息框 |
| GDI+ 绘制（`OnPaint`） | `DrawingContext` 或自定义 `Render` 重写 | 渲染 API 完全不同 |
| `Application.Run(new MainForm())` | `AppBuilder` 管线 | Avalonia 使用 builder 模式启动应用 |

## 迁移策略

从 WinForms 迁移到 Avalonia 不是一次简单的查找替换工作。最成功的方式通常是渐进式迁移：先用 Avalonia 开始编写新的界面，同时保留现有 WinForms 页面继续运行，然后再逐步把剩余部分迁过去。

### 方案 1：从头开始，逐步迁移

新建一个 Avalonia 项目，并从最简单的界面开始，一个一个地重建页面。这是最干净、最终效果也最好的方式，但前期投入会更大。

1. 按照[快速开始指南](/docs/get-started/create-your-first-project)创建新的 Avalonia 项目。
2. 搭建视图模型和数据层（这些通常可以直接从原 WinForms 项目中复用）。
3. 将每个界面重建为 Avalonia 的 `Window` 或 `UserControl`。
4. 当所有页面都迁移完成后，再下线原 WinForms 项目。

### 方案 2：在 WinForms 中托管 Avalonia 控件（仅限 Windows）

如果你需要在保持 WinForms 应用继续运行的同时逐步引入 Avalonia，那么可以使用 `WinFormsAvaloniaControlHost` 直接把 Avalonia 控件嵌入到 WinForms 窗口中。这样你就能在不动现有 WinForms 代码的前提下，用 Avalonia 开发新功能。

这种方式只适用于 Windows，因为 Windows Forms 本身就只能运行在 Windows 上。不过，如果你把 Avalonia 控件组织在独立类库中，那么这些控件之后仍然可以被复用于一个独立的跨平台 Avalonia 应用中。

有关具体接入方式，请参阅[在 Windows Forms 中嵌入 Avalonia](/docs/platform-specific-guides/windows#embedding-avalonia-in-windows-forms)。

## 需要重点学习的概念

如果你来自 WinForms 且没有 XAML 经验，那么以下几个领域会是最需要适应的部分：

- **[XAML 基础](/docs/xaml)：** Avalonia 使用 XAML 声明 UI 布局和结构，它将替代 WinForms 设计器。
- **[数据绑定](/docs/data-binding/introduction-to-data-binding)：** 你不再需要在事件处理程序中手动设置控件属性，而是把控件绑定到视图模型属性上，让变化自动流动。
- **[MVVM 模式](/docs/fundamentals/the-mvvm-pattern)：** Avalonia 围绕 UI（Views）与应用逻辑（ViewModels）分离这一思想设计，这是从 WinForms 迁移时最大的思维转变。
- **[样式](/docs/styling/styles)：** Avalonia 使用类 CSS 的样式系统、选择器和样式类，而不是逐个控件手动设属性。
- **[布局面板](/docs/layout)：** Avalonia 不依赖绝对定位或 dock/anchor，而是使用 `Grid`、`StackPanel`、`DockPanel` 等面板来排列控件。

## 另请参阅

- [Avalonia 快速开始](/docs/get-started/create-your-first-project)：创建你的第一个 Avalonia 应用。
- [在 Windows Forms 中嵌入 Avalonia](/docs/platform-specific-guides/windows#embedding-avalonia-in-windows-forms)：在现有 WinForms 应用中托管 Avalonia 控件。
- [控件参考](/controls)：完整的 Avalonia 控件文档。
