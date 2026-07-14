---
id: compatibility
title: 库兼容性
---

本页介绍 Avalonia XPF 与特定第三方库的兼容性说明。有关启用 Win32 API shim（大多数非 Windows 平台上的第三方库都需要它），请参见 [Win32 API Shims](/xpf/third-party/win32-api-shims)。

## DevExpress

DevExpress 控件在 XPF 中被广泛使用。入门步骤：

1. 在 `App` 构造函数或 `Program.Main` 中启用 Win32 API shim（DevExpress 控件需要）：
   ```csharp
   AvaloniaUI.Xpf.WinApiShim.WinApiShimSetup.AutoEnable();
   ```

   如果你的应用还使用了提供自身跨平台支持的库（且不应被 shim 拦截），请使用过滤回调：
   ```csharp
   AvaloniaUI.Xpf.WinApiShim.WinApiShimSetup.AutoEnable(asm =>
   {
       var name = asm.GetName().Name?.ToLowerInvariant();
       // 跳过自行处理跨平台支持的程序集
       if (name is "sqlite" or "skiasharp")
           return true; // true = 跳过此程序集
       return false;
   });
   ```

2. 确认 shim 在正确的位置启用。一个常见错误是在 macOS 启动器项目的 `Program.cs` 中启用 shim，但没有在 `App.xaml.cs` 中启用（或反过来）。在任何第三方程序集尝试调用 Win32 API 之前，必须先启用 shim。

3. 注意以下平台相关限制：

   - **GDI+ 依赖**：某些 DevExpress 控件（DocumentPreviewControl、PdfViewerControl、XtraReport）依赖 `System.Drawing.Common`（GDI+），而它在非 Windows 平台上已被弃用。请在可用时启用 DevExpress 的 Skia 渲染引擎。有关特定控件的 Skia 支持，请联系 DevExpress 支持。
   - **LoadingDecorator**：将 `UseSplashScreen=true` 的 DevExpress `LoadingDecorator` 需要多个 UI 线程，而这在 macOS 上不受支持。可改用 `WaitIndicator` 作为替代。
   - **Linux 依赖**：Linux 上的 DevExpress 控件需要 `libgdiplus`。参见 [Linux: Other Dependencies](/xpf/platforms/linux#other-dependencies)。

4. 请使用官方 DevExpress XPF 示例作为正确配置的参考：[Avalonia-XPF-Samples/DevExpressApp](https://github.com/AvaloniaUI/Avalonia-XPF-Samples/tree/master/src/DevExpressApp)。

DevExpress 维护了一个演示应用，展示其哪些控件已在 XPF 中完成测试。

## CefSharp

`CefSharp.Wpf.NetCore` 是为 Windows 设计的，并包含 Windows 原生的 Chromium 二进制文件。它不能在 Linux 或 macOS 上运行。

如果 CefSharp 因 `CursorInteropHelper.Create()` 抛出 `NotImplementedException`，请升级到 XPF 1.6.0 或更高版本，该版本提供了回退方案。作为旧版本的变通办法，可从 `ChromiumWebBrowser` 派生并重写 `OnCursorChange`，将 CefSharp 光标类型映射到 WPF `Cursors`。

有关跨平台浏览器替代方案，请参见 [Web Content Embedding](/xpf/interop/web-content)。

## Dragablz

Dragablz 使用的 Win32 API 在 XPF shim 层中并未完全实现（例如用于特定窗口外框效果的 `DwmGetWindowAttribute`）。在 Linux 上，这会导致运行时异常。

推荐做法是 fork Dragablz 库，并移除或保护这些不受支持的平台 API 调用。不受支持的调用通常位于窗口外框和标签拖拽分离代码中，这部分可以用跨平台替代方案替换。

## Caliburn.Micro

在 XPF 中使用 Caliburn.Micro 时，你可能会在启动期间遇到线程异常（例如，“调用线程无法访问此对象，因为另一个线程拥有该对象”）。这通常是由于 Caliburn.Micro 的 `WindowManager` 在非 UI 线程上访问了 WPF 窗口属性。请确保所有窗口操作都发生在调度器线程上。

## WinForms 控件

XPF 中对 WinForms 承载的支持仅限 Windows。要启用原生 WinForms 集成：

```xml
<PropertyGroup Condition="$([MSBuild]::IsOSPlatform('Windows'))">
    <XpfUseMicrosoftWindowsForms>true</XpfUseMicrosoftWindowsForms>
</PropertyGroup>
```

这会禁用 XPF 的 WinForms shim 层。此条件设置可确保你的项目在其他平台上仍能构建。对于跨平台部署，请为 Windows 上由 WinForms 控件处理的功能提供替代 UI。

## Aspose

Aspose 库会在某些程序集上设置自己的 `DllImportResolver`。由于 .NET 对每个程序集只允许一个 resolver，这与 XPF 的 WinApiShim 发生冲突。有关解决方法，请参见 [Win32 API Shims: Resolving DllImportResolver Conflicts](/xpf/third-party/win32-api-shims#resolving-dllimportresolver-conflicts)。

## 兼容性数据库

第三方控件的 [compatibility database](https://avaloniaui.net/xpf/packages) 可供查询。该数据库提供主要厂商控件的最新状态信息。

:::info
如果你发现某个标记为 `Fix In Progress` 或 `Untested` 的控件对你的应用至关重要，请联系支持团队。Avalonia 团队致力于与你合作，确保兼容性。
:::

### 兼容性说明

* **纯 WPF 控件**：纯粹用 WPF 实现的第三方控件通常都能正常工作，即使未列在兼容性数据库中。
* **未列出的厂商**：数据库中没有某个控件厂商，并不代表它不兼容。请测试你需要的控件。
* **已知挑战**：问题最常出现在使用 GDI 或 WinForms 组件的控件上。