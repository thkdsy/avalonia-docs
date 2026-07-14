---
id: windows
title: Windows
description: 在 Windows 上运行 XPF 应用程序时的 Windows 特定注意事项，包括 Win32 API 行为和兼容性说明。
---

## 概述

Windows 是 WPF 的原生平台，XPF 应用程序在 Windows 上运行时具有完整兼容性。本页介绍在使用 XPF 而非标准 WPF 时需要考虑的 Windows 特定事项。

## Win32 API 行为

当在 Windows 上启用 Win32 API shim 时，调用会通过 XPF shim 层路由，而不是直接调用原生 Win32 API。这确保了在开发和测试期间，与非 Windows 平台保持一致的行为。

如果你需要直接调用原生 Win32 API（绕过 shim），请将这些调用放在单独的程序集里，并将其从 shim 中排除：

```csharp
AvaloniaUI.Xpf.WinApiShim.WinApiShimSetup.AutoEnable(asm =>
{
    if (asm.GetName().Name == "MyNativeWindowsInterop")
        return true; // skip this assembly
    return false;
});
```

## WinForms 承载

XPF 可以在 Windows 上承载 WinForms 控件。要启用此功能，请将以下内容添加到 Windows 条件属性组中：

```xml
<PropertyGroup Condition="$([MSBuild]::IsOSPlatform('Windows'))">
    <XpfUseMicrosoftWindowsForms>true</XpfUseMicrosoftWindowsForms>
</PropertyGroup>
```

这会禁用 XPF 的 WinForms shim 层，并启用原生 WinForms 集成。WinForms 承载仅在 Windows 上可用；请为该属性组添加条件，以避免在其他平台上构建失败。

## STA 线程

某些 Windows 操作要求主线程标记为 STA（Single-Threaded Apartment，单线程单元）：

- 通过 COM 进行的剪贴板操作
- OLE 拖放
- 某些第三方 COM 组件

如果遇到 `COMException: CoInitialize was not called`，请在入口点添加 `[STAThread]` 属性。使用 [自定义初始化](/xpf/configuration/customizing-initialization) 时，XPF 会自动处理这一点。

## CefSharp

`CefSharp.Wpf.NetCore` 可在 Windows 上与 XPF 配合使用。如果 `CursorInteropHelper.Create()` 抛出 `NotImplementedException`，请升级到 XPF 1.6.0 或更高版本。

对于较旧的 XPF 版本，可通过从 `ChromiumWebBrowser` 派生并重写 `OnCursorChange` 来规避此问题：

```csharp
protected override void OnCursorChange(object sender, CursorChangeEventArgs e)
{
    // 将 CefSharp 光标类型映射到 WPF Cursors
    var cursor = e.CursorType switch
    {
        CefCursorType.Hand => Cursors.Hand,
        CefCursorType.IBeam => Cursors.IBeam,
        CefCursorType.Cross => Cursors.Cross,
        CefCursorType.Wait => Cursors.Wait,
        _ => Cursors.Arrow
    };
    Cursor = cursor;
}
```

关于跨平台浏览器嵌入，请参见 [Web 内容嵌入](/xpf/interop/web-content)。

## 窗口句柄

XPF 对诸如 `WindowInteropHelper.Handle` 之类的 WPF API 调用使用虚拟窗口句柄。在 Windows 上，这些是真正的 HWND。若要直接访问底层句柄，请使用 Avalonia 互操作 API：

```csharp
using Atlantis;

var avaloniaWindow = XpfWpfAbstraction.GetAvaloniaWindowForWindow(myWpfWindow);
var handle = avaloniaWindow.TryGetPlatformHandle();
// handle.Handle is the native HWND on Windows
```

更多详情请参见 [原生窗口句柄](/xpf/interop/native-window-handles)。

## 部署

有关发布、打包和安装程序指导，请参见 [Windows 部署](/xpf/deployment/windows)。