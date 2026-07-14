---
id: index
title: 从 WPF 迁移
description: 使用等效控件、绑定方式和 MVVM 模式，将 WPF 应用迁移到 Avalonia。
doc-type: migration
---

Avalonia 与 WPF 共享许多概念，因此你现有的知识可以直接迁移过来。大多数控件、布局面板、数据绑定和 MVVM 模式都以相同方式工作，或者至少存在清晰的对应关系。这两个框架之间最主要的差异集中在样式系统、数据模板、属性系统以及事件命名上。

:::tip[已经拥有大型 WPF 代码库？]
如果你的目标是在 macOS、Linux 或 Web 上运行现有 WPF 应用，那么未必需要完全重写。[XPF](/xpf) 可以在仅做少量代码修改的前提下跨平台运行 WPF 应用，因此你可以在保留现有代码库的同时发布到新平台。
:::

## 关键差异

**样式系统** 是最大的概念变化。Avalonia 使用一种类似 CSS 的系统来替代 WPF 中基于资源字典的样式和触发器，这套系统由选择器、样式类和伪类组成。完整说明请参阅 [样式](/docs/styling/styles)。

**数据模板** 的工作方式与 WPF 类似，但它们存放在 `DataTemplates` 集合中，而不是资源中，并且支持接口匹配和派生类型匹配。请参阅 [数据模板](/docs/data-templates/introduction-to-data-templates)。

**属性系统** 使用强类型泛型（`StyledProperty`、`DirectProperty`、`AttachedProperty`）来代替单一的 `DependencyProperty` 类。请参阅 [属性系统](/docs/properties)。

**事件** 仍然遵循相同的路由事件模型，但命名改为基于指针（例如用 `PointerPressed` 代替 `MouseLeftButtonDown`），并且通过路由策略标志来处理 tunnelling，而不是使用单独的 `Preview*` 事件。请参阅 [事件](/docs/events)。

**控件** 大体上是相同的。少数控件名称有所不同，或者需要单独安装 NuGet 包。请参阅 [控件](controls)。

**布局** 面板（`Grid`、`StackPanel`、`DockPanel` 等）基本相同，只增加了一些小功能，例如 `StackPanel` 上的 `Spacing` 以及简写的 `ColumnDefinitions` 语法。请参阅 [布局](/docs/layout)。

## 从哪里开始

如果你想先快速做一个并排对照，可以从 **[速查表](/docs/migration/wpf/cheat-sheet)** 开始。它用紧凑的表格覆盖了 XAML 命名空间、属性系统、样式、数据绑定、控件、事件、命令、模板、线程、动画、图形以及文件结构等内容。

如果你想深入理解每个主题，请继续阅读上方链接的各个专题指南。

## 另请参阅

- [WPF 到 Avalonia 速查表](/docs/migration/wpf/cheat-sheet)：快速并排参考。
- [样式](/docs/styling/styles)：类似 CSS 的样式系统迁移指南。
- [控件](controls)：控件名称映射。
- [属性系统](/docs/properties)：属性系统差异说明。
