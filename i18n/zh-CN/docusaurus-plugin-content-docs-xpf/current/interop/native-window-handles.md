---
id: native-window-handles
title: 获取本机窗口句柄
description: 如何在 XPF 应用程序中检索本机平台窗口句柄，包括虚拟句柄与本机句柄之间的区别。
---

## 概述

XPF 使用一种系统：从各种 WPF API 调用返回的句柄是 _虚拟句柄_。通过这种方式，XPF 可以使用这些句柄拦截 API 调用，并自动将它们转换为相应的跨平台 API。这意味着许多 WPF API，例如 `WindowInteropHelper.Handle`，返回的是这些虚拟化的窗口句柄，以及 [模拟的 Win32 API](/xpf/third-party/win32-api-shims)。

## 何时需要本机句柄

在以下情况下，你可能需要访问真实的本机窗口句柄：

- 与需要窗口句柄的本机平台 API 交互
- 嵌入本机控件或渲染表面（OpenGL、DirectX、Metal）
- 使用 WPF 或 Avalonia API 中不可用的平台特定功能
- 集成本机可访问性或自动化框架

## 获取本机句柄

可以从 [底层的 Avalonia `Window`](/xpf/interop/embedding-avalonia-in-xpf#getting-the-avalonia-window) 获取窗口的本机句柄：

```csharp
using Atlantis;

var avaloniaWindow = XpfWpfAbstraction.GetAvaloniaWindowForWindow(myWpfWindow);
var platformHandle = avaloniaWindow.TryGetPlatformHandle();

if (platformHandle != null)
{
    IntPtr nativeHandle = platformHandle.Handle;
    string handleType = platformHandle.HandleDescriptor;
    // Use the native handle
}
```

## 按平台划分的句柄类型

返回的句柄类型取决于操作系统：

| Platform | Handle Type | HandleDescriptor | Notes |
|---|---|---|---|
| Windows | HWND | `"HWND"` | 标准 Win32 窗口句柄 |
| macOS | NSWindow* | `"NSWindow"` | 指向 NSWindow 对象的指针 |
| Linux (X11) | X11 Window | `"XID"` | X11 窗口标识符 |

:::caution
本机句柄不能传递给 Win32 API 模拟层。模拟层使用的是 XPF 的虚拟句柄，而不是本机平台句柄。如果你需要调用已替换的 Win32 API，请改用 `WindowInteropHelper.Handle` 中的虚拟句柄。
:::

## 获取 Avalonia `TopLevel`

对于不需要窗口句柄，但需要访问 Avalonia 层属性的操作（例如渲染缩放或输入处理），请使用 `GetAvaloniaTopLevelForWindow`：

```csharp
using Atlantis;

var topLevel = XpfWpfAbstraction.GetAvaloniaTopLevelForWindow(myWpfWindow);

// Access render scaling (useful for DPI-aware rendering)
double scaling = topLevel.RenderScaling;
```

## 示例：支持 DPI 的本机渲染

```csharp
using Atlantis;

private void SetupNativeRendering(System.Windows.Window wpfWindow)
{
    var avaloniaWindow = XpfWpfAbstraction.GetAvaloniaWindowForWindow(wpfWindow);
    var platformHandle = avaloniaWindow.TryGetPlatformHandle();

    if (platformHandle == null)
        return;

    double scaling = avaloniaWindow.RenderScaling;
    IntPtr handle = platformHandle.Handle;

    switch (platformHandle.HandleDescriptor)
    {
        case "HWND":
            // Windows: use HWND for DirectX/OpenGL context creation
            InitializeWindowsRenderer(handle, scaling);
            break;
        case "NSWindow":
            // macOS: use NSWindow for Metal/OpenGL context creation
            InitializeMacRenderer(handle, scaling);
            break;
        case "XID":
            // Linux: use X11 Window for OpenGL context creation
            InitializeLinuxRenderer(handle, scaling);
            break;
    }
}
```

## 另请参阅

- [在 XPF 中嵌入 Avalonia](/xpf/interop/embedding-avalonia-in-xpf)，用于从 XPF 访问 Avalonia 功能
- [性能：嵌入高性能内容](/xpf/configuration/performance#embedding-high-performance-content)，用于 OpenGL 集成