---
id: win32-api-shims
title: Win32 API Shim
---

Avalonia XPF 实现了 WPF 的 API 表面，不过许多第三方库也依赖各种 Win32 API，而这些 API 在跨平台环境中不可用。

为了解决这个问题，Avalonia XPF 实现了一个 Win32 API 仿真层，使第三方库能够在非 Windows 平台上运行。此仿真层需要在你的 XPF 应用程序中显式启用。

## 何时启用 Win32 API shim

如果你的应用程序符合以下任一情况，请启用 shim 层：

- 它使用来自 DevExpress、Actipro、Syncfusion、Telerik 或 Infragistics 等厂商的第三方 WPF 控件
- 它在 macOS 或 Linux 上崩溃，并显示 `DllNotFoundException: Unable to load shared library 'user32.dll'` 或类似的 Win32 DLL 错误
- 它直接调用 Win32 API（P/Invoke）来进行窗口管理、显示信息或主题设置

如果你的应用程序只使用标准 WPF 控件，并且没有任何第三方库在内部调用 Win32 API，则可以跳过 shim 层。

如有疑问，请启用 shims。即使并非严格必需，也可以安全启用。

## 启用 Win32 API shims

此功能必须在任何程序集尝试调用 Win32 API 之前启用，因此在 `App` 类的构造函数或 `Program.Main` 中启用它是个不错的选择。

要在整个应用程序范围内启用 Win32 API 仿真，可以添加以下调用：

```csharp
  AvaloniaUI.Xpf.WinApiShim.WinApiShimSetup.AutoEnable();
```

你可以像这样排除已知提供非 Windows 平台支持的库：

```csharp
AvaloniaUI.Xpf.WinApiShim.WinApiShimSetup.
  .AutoEnable(asm =>
  {
    var name = asm.GetName().Name.ToLowerInvariant();
    if (name is "sqlite" or "jint" or "esprima" or "magick.net" or "magick.net.core")
      return true;
    return false;
  });
```

或者，也可以按库逐个启用该层：

```csharp
AvaloniaUI.Xpf.WinApiShim.WinApiShimSetup
  .AddLibrary(typeof(Type.In.Third.Party.Library).Assembly);
```

## 解决 `DllImportResolver` 冲突

某些第三方库（例如 Aspose）会在入口程序集上设置自己的 `DllImportResolver`。由于 .NET 只允许每个程序集设置一个 resolver，这会与 XPF 的 WinApiShim 发生冲突，从而导致 `InvalidOperationException: A resolver is already set for the assembly`。

使用 `AutoEnable` 的筛选回调来跳过冲突的程序集：

```csharp
AvaloniaUI.Xpf.WinApiShim.WinApiShimSetup.AutoEnable(asm =>
{
    var name = asm.GetName().Name;
    if (name != null && name.Contains("Aspose"))
        return true; // true = skip this assembly
    return false;
});
```

## 避免自定义程序集加载导致的死锁

如果你的应用程序使用自定义机制加载托管程序集（例如插件系统），`AutoEnable` 可能会在启动期间造成死锁。在这种情况下，请改用 `AddLibrary`，在程序集解析完成后逐个注册：

```csharp
// Instead of AutoEnable, add assemblies as they are loaded
AppDomain.CurrentDomain.AssemblyLoad += (sender, args) =>
{
    AvaloniaUI.Xpf.WinApiShim.WinApiShimSetup.AddLibrary(args.LoadedAssembly);
};
```

更多详情请参见 [自定义初始化](/xpf/configuration/customizing-initialization#custom-assembly-loading)。

## Win32 API shims 的作用范围

Win32 API shim 层的存在是为了让内部调用 Win32 API 的第三方 WPF 控件能够兼容。它**不是**一个通用的 Win32 仿真层。

要点：
- Shims 提供足够的 Win32 API 表面，以支持非 Windows 平台上的常见第三方 WPF 控件
- 在 Windows 上启用 shims 会将调用重定向到 shim 实现，而不是原生 Win32
- 并非所有 Win32 API 都可用（请参阅 [API 参考](/xpf/third-party/winapi-reference)）
- Windows 消息（例如 `WM_ACTIVATEAPP`）只会在支持的控件所需的范围内生成
- 在可能的情况下，直接使用 WPF 或 Avalonia API，而不要依赖 Win32 API shims

## 故障排除

### 应用程序在 macOS 或 Linux 上因 DllNotFoundException 崩溃

这是未启用 Win32 API shims 时最常见的症状。请将 `AvaloniaUI.Xpf.WinApiShim.WinApiShimSetup.AutoEnable()` 添加到你的 `App` 构造函数或 `Program.Main` 中，并确保它在任何第三方程序集加载之前运行。

### 使用 AutoEnable 时应用程序在启动期间冻结

如果你的应用程序使用自定义程序集加载机制（例如插件系统），`AutoEnable` 可能会在启动期间通过拦截程序集加载而导致死锁。请切换到 `AddLibrary`，以逐个注册程序集。参见 [自定义初始化：自定义程序集加载](/xpf/configuration/customizing-initialization#custom-assembly-loading)。

### shims 已启用，但某个特定库仍然失败

某些库调用了 shim 层中未包含的 Win32 API。请检查 [WinAPI Shim API 参考](/xpf/third-party/winapi-reference)，确认该库所需的 API 是否可用。如果缺少所需 API，请联系 Avalonia 支持团队。

### Linux 上出现 EntryPointNotFoundException

如果启用 Win32 API shims 后，原生 Linux API（例如 X11 函数）出现 `EntryPointNotFoundException`，说明 shim 层拦截了本应交给原生库处理的调用。请将原生 Linux API 调用移动到单独的程序集，并将其排除在 shim 之外。参见 [Linux：Win32 API Shim 冲突](/xpf/platforms/linux#win32-api-shim-conflicts-with-native-apis)。

### A resolver is already set for the assembly

当第三方库（例如 Aspose）设置了自己的 `DllImportResolver` 时，就会发生这种情况。请使用 `AutoEnable` 的筛选回调跳过该库的程序集。参见 [解决 DllImportResolver 冲突](#resolving-dllimportresolver-conflicts)。