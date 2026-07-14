---
id: known-differences
title: 与 WPF 的已知差异
---

## 概述

XPF 保持与 WPF 的 API 和二进制兼容性，但由于渲染引擎不同（使用 Skia 而不是 MilCore）以及跨平台要求，行为上存在一些差异。本页记录了已知差异，以帮助你规划迁移。

## 渲染

### 虚线描边和线帽

Skia 对虚线描边及线帽的渲染方式与 WPF 的 milcore 引擎不同。如果你的应用使用带自定义线帽（三角形、圆形）的 `StrokeDashArray`，视觉输出可能会与 WPF 和 XPF 之间略有差异。这是 Skia 渲染后端中的一种根本性差异。

### 模糊效果

与 WPF 的硬件加速管线相比，Skia 中的模糊效果（`BlurEffect`、`DropShadowEffect`）计算开销更大。大量使用模糊效果的应用可能会看到帧率下降。有关缓解策略，请参见 [性能：模糊效果](/xpf/configuration/performance#blur-effects)。

## 控件

### ContextMenu

在 WPF 中，当你以编程方式打开上下文菜单时，`PlacementTarget` 会被隐式设置。在 XPF 中，你必须显式设置它：

```csharp
myContextMenu.PlacementTarget = targetElement;
myContextMenu.IsOpen = true;
```

右键上下文菜单的行为与 WPF 完全一致，无需任何更改。

### MessageBox

`System.Windows.MessageBox` 和 `System.Windows.Forms.MessageBox` 都受支持，但渲染输出可能与原生 Windows 消息框不同：

- 文本换行位置可能不同（XPF 将宽度限制在大约 400px）
- 图标位置可能会略有差异
- 按钮样式使用平台的原生外观

### TextBox 触控行为

在触摸屏设备上，在 `TextBox` 上拖动手指会导致文本滚动/滑动。此行为继承自 WPF。若要禁用它：

```csharp
ScrollViewer.SetPanningMode(myTextBox, PanningMode.None);
```

或在 XAML 中：

```xml
<TextBox ScrollViewer.PanningMode="None" />
```

### FlowDocument

FlowDocument 可用，但有一些限制：
- 不支持分页
- 不支持浮动元素
- 表格支持有限

完整列表请参见 [缺失功能](/xpf/version-info/missing-features)。

## 窗口管理

### 透明窗口

XPF 使用 `WS_EX_NOREDIRECTIONBITMAP`，而不是 `WS_EX_LAYERED`（WPF 使用后者）。这意味着：

- 不支持按像素的点击穿透。对窗口透明区域的鼠标点击不会传递到下方窗口。
- 对于覆盖层场景，请将内容嵌入单个窗口，而不是叠加透明窗口。有关 OpenGL 嵌入，请参见 [性能：嵌入高性能内容](/xpf/configuration/performance#embedding-high-performance-content)。

### 多个 UI 线程

WPF 支持在独立的调度器线程上创建窗口。XPF 不支持在 macOS 上使用多个 UI 线程（该平台只允许一个）。在 Windows 和 Linux 上，对多个调度器的支持也有限。依赖于在单独线程上显示启动画面或进度窗口的模式，应重构为使用主调度器。

### Window.ShowActivated

`ShowActivated` 属性在 XPF 1.6.0 及更高版本中受支持。

### 窗口关闭事件

当窗口通过编程或关闭按钮关闭时，`Closing` 事件会触发一次。在较早的 XPF 版本（1.6.0 之前），使用 `Window.Close()` 时 `Closing` 可能会触发两次。

若要覆盖关闭行为，请处理 `Closing` 事件并设置 `e.Cancel = true`：

```csharp
protected override void OnClosing(CancelEventArgs e)
{
    e.Cancel = true; // Prevent close
    Hide();          // Hide instead
}
```

### Win32 窗口消息

XPF 的 Win32 API shim 层会在支持的第三方控件所需范围内生成窗口消息（例如 `WM_ACTIVATEAPP`、`WM_SETFOCUS`）。并非所有平台都会生成所有 Win32 消息。如果你的应用依赖特定窗口消息进行窗口间通信，请改用 .NET IPC 机制（例如命名管道或内存映射文件）。

## API

### VisualTreeHelper.GetDpi

`VisualTreeHelper.GetDpi()` 在 macOS 上可能无法返回准确值。请改用 Avalonia 互操作 API：

```csharp
using Atlantis;

var topLevel = XpfWpfAbstraction.GetAvaloniaTopLevelForWindow(myWindow);
double scaling = topLevel.RenderScaling;
```

### CursorInteropHelper.Create

`CursorInteropHelper.Create()` 未完全实现，因为它依赖原生 Windows 光标句柄。XPF 1.6.0+ 提供了回退到默认光标的机制。有关自定义光标映射，请参见 [Windows: CefSharp](/xpf/platforms/windows#cefsharp)。

### SystemSounds.Beep

`System.Media.SystemSounds.Beep` 在 macOS 和 Linux 上会抛出 `PlatformNotSupportedException`。请在调用此 API 时加入平台检查，或在跨平台构建中移除它们。

### System.Drawing.Common

`System.Drawing.Common`（GDI+）在非 Windows 平台上已被弃用。依赖 GDI+ 进行渲染的第三方控件（例如某些 DevExpress 控件）会在 macOS 和 Linux 上抛出异常。有关解决方法，请参见 [macOS: GDI+ 和 System.Drawing.Common](/xpf/platforms/macos#gdi-and-systemdrawingcommon)。

## 文件对话框

### FilterIndex

`OpenFileDialog` 和 `SaveFileDialog` 的 `FilterIndex` 在 XPF 1.6.0 及更高版本中完全受支持。

### Linux 上的 `InitialDirectory`

在较旧的 Linux 发行版上，如果 GNOME 版本不支持 DBus 文件对话框协议，`InitialDirectory` 可能会被忽略。有关解决方法，请参见 [Linux: 较旧发行版上的文件对话框](/xpf/platforms/linux#file-dialogs-on-older-distributions)。

### OpenFolderDialog

跨平台文件夹选择请使用 `Microsoft.Win32.OpenFolderDialog`。某些第三方文件夹对话框实现（例如 DevExpress FolderDialog）可能无法在 macOS 上正常工作。

### FolderBrowserDialog（System.Windows.Forms）

`System.Windows.Forms.FolderBrowserDialog` 在 XPF 中受支持，但会映射到平台的原生文件夹选择器。在 Linux 和 macOS 上，对话框的外观和行为会与 Windows 不同。若要获得一致的行为，建议使用 `Microsoft.Win32.OpenFolderDialog`。

### 对话框迁移模式

将 WPF 对话框代码迁移到 XPF 以用于跨平台时：

- 尽可能将 `System.Windows.Forms.OpenFileDialog` 替换为 `Microsoft.Win32.OpenFileDialog`
- 避免设置没有跨平台等效项的 Windows 特定对话框属性（例如 `DereferenceLinks`）
- 在 macOS 上，避免在窗口激活期间显示模态对话框。请参见 [macOS: 启动和模态对话框](/xpf/platforms/macos#startup-and-modal-dialogs)

## 剪贴板

有关剪贴板差异的完整说明，请参见 [剪贴板](/xpf/migration/clipboard)。

## 字体

- WPF 和 XPF 的字体匹配规则不同。带有非标准样式名称的字体可能无法被正确匹配。
- 可自定义字体回退行为。请参见 [入门：字体](/xpf/getting-started#fonts)。
- 跨平台渲染在不同平台上使用不同的文本后端，因此视觉差异是预期内的。