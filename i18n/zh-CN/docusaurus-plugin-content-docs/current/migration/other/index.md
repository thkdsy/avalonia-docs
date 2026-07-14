---
title: 其他框架
description: 从 Delphi、Qt、Electron 或 ASP.NET MVC 桌面应用迁移到 Avalonia。
doc-type: migration
---

Avalonia 并不只适用于来自微软 XAML 生态的开发者。如果你正在使用 Delphi、Qt、Electron 或 ASP.NET MVC 构建桌面或跨平台应用，并希望寻找一个现代化的 .NET 替代方案，那么 Avalonia 可能非常适合你。

本页会介绍：当你从 XAML 世界之外的框架迁移到 Avalonia 时，可以预期会遇到哪些变化。

:::tip[迁移需要帮助？]
Avalonia 团队拥有协助团队从多种 UI 框架迁移过来的经验。如果你希望获得专家指导，我们可以提供这项服务。更多信息请参阅 [Avalonia Services](https://avaloniaui.net/services)。
:::

## 从 Delphi（VCL / FireMonkey）迁移

Delphi 数十年来一直是桌面开发的重要工具。许多关键业务应用至今仍运行在 VCL 或 FireMonkey 上。但 Delphi 生态正在收缩：开发者更少、库更少、授权成本更高。寻找并招聘 Delphi 开发者也在一年比一年困难。

Avalonia 为你提供了一条通往 .NET 的路径。相较之下，.NET 拥有庞大得多的开发者群体、活跃的开源生态以及现代化工具链。迁移过程需要学习 XAML 和 MVVM，但两边的核心概念之间其实有不少可以顺畅映射的地方。

### 概念映射

| Delphi (VCL / FMX) | Avalonia | 说明 |
|---|---|---|
| `TForm` | `Window` | |
| `TFrame` | `UserControl` | 可复用 UI 组件 |
| `TPanel` | `Border` 或 `Panel` | |
| `TButton` | `Button` | |
| `TEdit` | [`TextBox`](/api/avalonia/controls/textbox) | |
| `TMemo` | `TextBox`，并设置 `AcceptsReturn="True"` | |
| `TLabel` | `TextBlock` | |
| `TCheckBox` | `CheckBox` | |
| `TRadioButton` | `RadioButton` | |
| `TComboBox` | `ComboBox` | |
| `TListBox` | `ListBox` | |
| `TTreeView` | `TreeView` | |
| `TStringGrid` / `TDBGrid` | `DataGrid`（NuGet 包） | |
| `TTabControl` | `TabControl` | |
| `TImage` | `Image` | |
| `TScrollBox` | `ScrollViewer` | |
| `TPopupMenu` | `ContextMenu` | |
| `TMainMenu` | `Menu` | |
| `TTimer` | `DispatcherTimer` | |
| `TAction` / `TActionList` | `ICommand`（MVVM 模式） | |
| 组件上的事件处理程序 | 数据绑定 + MVVM | 最大的架构变化 |
| DFM 表单文件 | `.axaml` XAML 文件 | 声明式布局 |
| `Application.CreateForm()` | `AppBuilder` 管线 | |

### 关键差异

- **没有可视化表单设计器（拖拽式）：** Avalonia 使用 XAML 来声明布局。虽然有实时预览器，但你写的是标记，而不是把组件拖到画布上。大多数开发者跨过最初学习曲线后，会发现这种方式更高效。
- **MVVM 取代事件驱动代码：** 在 Delphi 中，你通常直接把按钮点击事件连到某个方法上。在 Avalonia 中，则是将控件绑定到视图模型的属性和命令。这种分离让代码更易测试、也更易维护。
- **没有组件面板：** 你不再是从面板里拖出组件，而是在 XAML 中声明控件，并通过属性和绑定进行配置。
- **.NET 生态：** NuGet 取代了 Delphi 的组件市场，而 .NET 的包生态要大得多。

## 从 Qt（QML / Qt Widgets）迁移

Qt 是一个经过验证的跨平台框架，但它基于 C++、授权模式较复杂（LGPL 与商业授权）、并且 Qt Widgets 与 QML 的分裂也常常带来额外摩擦。如果你的团队已经在使用 .NET，或者计划转向 .NET，那么 Avalonia 能让你在不离开 .NET 生态的前提下获得跨平台 UI 框架。

### 概念映射

| Qt | Avalonia | 说明 |
|---|---|---|
| `QMainWindow` | `Window` | |
| `QWidget` | `Control` | |
| `QML` 声明式 UI | XAML 声明式 UI | 两者都是基于标记语言的 UI 定义方式 |
| Signals and slots | 数据绑定 + `ICommand` | Avalonia 使用 MVVM，而不是 signal/slot |
| `QVBoxLayout` / `QHBoxLayout` | `StackPanel` | |
| `QGridLayout` | `Grid` | |
| `QPushButton` | `Button` | |
| `QLineEdit` | `TextBox` | |
| `QTextEdit` | `TextBox`，并设置 `AcceptsReturn="True"` | |
| `QLabel` | `TextBlock` | |
| `QCheckBox` | `CheckBox` | |
| `QComboBox` | `ComboBox` | |
| `QListView` / `QListWidget` | `ListBox` | |
| `QTreeView` / `QTreeWidget` | `TreeView` | |
| `QTableView` | `DataGrid`（NuGet 包） | |
| `QTabWidget` | `TabControl` | |
| `QScrollArea` | `ScrollViewer` | |
| `QMenu` | `ContextMenu` / `Menu` | |
| Qt Style Sheets (QSS) | 类 CSS 的选择器与样式 | Avalonia 的样式系统在概念上与 QSS 很接近 |
| `QThread` / 线程亲和性 | `Dispatcher.UIThread` | 与 UI 线程亲和性的概念相同 |
| `.ui` 文件（Qt Designer） | `.axaml` 文件 | |
| `qmake` / `CMake` | MSBuild / `dotnet` CLI | |

### 关键差异

- **不需要 C++：** Avalonia 是纯 .NET 技术栈（C# 或 F#），不需要桥接层，也无需为基础 UI 工作编写 P/Invoke。
- **授权更简单：** Avalonia 采用 MIT 许可。没有 LGPL 合规压力，也不需要为框架本身支付商业授权费用。
- **样式系统容易上手：** 如果你用过 Qt Style Sheets，那么 Avalonia 的类 CSS 选择器会让你感觉很自然。它支持伪类、嵌套选择器和样式类。
- **单一 UI 语言：** Qt 在 Widgets（C++）和 QML（类似 JavaScript）之间存在分裂，而 Avalonia 统一使用 XAML。

## 从 Electron 迁移

Electron 应用本质上是打包了 Chromium 的 Web 应用。它们当然能工作，但往往占用大量内存和 CPU、启动较慢，而且与操作系统的原生体验割裂感较强。如果你的团队当初选择 Electron 是因为它能最快实现跨平台，那么 Avalonia 则可以在提供类似平台覆盖范围的同时，带来更原生的性能表现。

### 团队为什么会离开 Electron

- **内存占用：** 每个 Electron 应用都带着一个完整的 Chromium 实例。一个简单应用就可能消耗数百 MB 的内存。
- **启动时间：** 加载 Chromium 会带来明显延迟，在低端硬件上尤为明显。
- **缺少原生质感：** Electron 应用看起来更像网页，而不是桌面应用。窗口管理、快捷键和系统集成都需要额外处理。
- **更新与打包复杂：** 应用内置 Chromium，会导致构建体积更大、更新更重。

### Avalonia 能提供什么替代能力

- **原生性能：** Avalonia 直接渲染，不依赖浏览器引擎。内存占用和启动时间都会显著降低。
- **真正的跨平台：** 基于单一 .NET 代码库即可覆盖 Windows、macOS、Linux、iOS、Android 和 WebAssembly。
- **更符合桌面平台习惯的行为：** 窗口管理、系统菜单、键盘导航和托盘图标等，都更符合各平台用户预期。
- **使用 C# 而不是 JavaScript：** 强类型、可编译，并配有成熟的工具链和调试支持。

## 从 ASP.NET MVC / Blazor（Web 到桌面）迁移

如果你已经有一个使用 ASP.NET MVC 或 Blazor 构建的 Web 应用，并希望提供原生桌面体验，那么 Avalonia 会是一个非常自然的搭档。你的后端、数据层和业务逻辑（都基于 .NET）可以直接与 Avalonia 桌面客户端共享。你无需重写应用逻辑，只需要为它构建一个原生前端。

### 可以直接复用的部分

- **模型和 DTO：** 你的数据类通常可以在 Avalonia 中直接使用，无需改动。
- **服务和业务逻辑：** 任何不依赖 ASP.NET HTTP 管线的部分，都可以直接复用。
- **依赖注入：** Avalonia 可以与 `Microsoft.Extensions.DependencyInjection` 配合使用，沿用你在 ASP.NET 中已经熟悉的模式。
- **验证：** `INotifyDataErrorInfo` 和数据注解验证器都可以与 Avalonia 的绑定系统协同工作。

### 需要改变的部分

- **不再使用 HTML/CSS/Razor：** Avalonia 使用 XAML 做布局，并使用类 CSS 的样式系统来控制外观。虽然概念与 HTML 不同，但对于 UI 开发来说，XAML 往往更紧凑。
- **没有 HTTP 请求/响应周期：** 桌面应用是有状态的。你会把 UI 控件绑定到视图模型属性上，让界面实时更新，而不是每次请求都重新渲染页面。
- **导航由应用自行管理：** 不再有 URL 路由。导航通常通过根据应用状态切换视图来实现，或者使用 [NavigationPage](/controls/navigation/navigationpage) 提供基于栈的页面导航。

## 开始上手

无论你来自哪个框架，最好的起点其实都一样：

1. **[创建你的第一个 Avalonia 应用](/docs/get-started/create-your-first-project)：** 在几分钟内跑起一个可工作的应用。
2. **[学习 XAML 基础](/docs/xaml)：** 理解 Avalonia 如何声明 UI。
3. **[学习数据绑定](/docs/data-binding/introduction-to-data-binding)：** 理解 Avalonia 如何把 UI 与数据连接起来。
4. **[浏览控件](/controls)：** 看看开箱即用都提供了哪些控件。

## 另请参阅

- [样式](/docs/styling/styles)：了解 Avalonia 的类 CSS 样式系统如何工作。
- [MVVM 模式](/docs/fundamentals/the-mvvm-pattern)：将 UI 与逻辑分离。
- [控件参考](/controls)：完整的 Avalonia 控件文档。
