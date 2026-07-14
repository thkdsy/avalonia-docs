---
id: faq
title: 常见问题
---

## .NET 版本兼容性

**XPF 支持哪些 .NET 版本？**

XPF 可与 .NET 6、7、8、9 和 10 配合使用。不需要使用特定的 .NET 版本。

**较新 .NET 版本中的 WPF 功能是否可用？**

XPF 是从 .NET 6 的 WPF 分支出来的。后来在 WPF 中于较新 .NET 版本里添加的功能（例如 .NET 9 中的 Fluent 主题）不会自动可用。不过，XPF 团队会回移植部分选定功能。例如，`OpenFolderDialog`（在 .NET 8 WPF 中引入）在 XPF 中可用。

## 目标框架

**我应该使用 `net8.0` 还是 `net8.0-windows`？**

请使用 `net8.0-windows`（或者你偏好的带有 `-windows` 后缀的 .NET 版本）。XPF SDK 使该目标框架可在所有平台上工作，因此在为 Linux 或 macOS 构建时你无需更改它。许多第三方库（例如 DevExpress）要求使用 Windows 特定的 TFM 才能编译。

你可以使用纯粹的 `net8.0` TFM，但前提是解决方案中的所有项目都使用 XPF SDK，而不是 `Microsoft.NET.Sdk`。对于纯 TFM，你不能使用 `<EnableWindowsTargeting>`。

**我可以针对不同平台使用不同的目标框架吗？**

可以。如果你需要平台特定的 API，可以使用多目标编译（例如，`net8.0-windows;net8.0-macos`）。不过，对于大多数 XPF 应用来说，使用带有 XPF SDK 的单一 `net8.0-windows` TFM 是最简单的方式。

## Win32 API shim

**我需要启用 Win32 API shim 吗？**

如果你的应用使用了内部调用 Win32 API 的第三方控件，则需要启用 Win32 API shim。这在 DevExpress、Actipro、Syncfusion、Telerik 以及其他主要 WPF 控件供应商的产品中很常见。

**我怎么知道自己是否需要它们？**

如果你的应用在 Windows 上可以运行，但在 Linux 或 macOS 上因类似 `DllNotFoundException: Unable to load shared library 'user32.dll'` 的错误而失败，那么你需要启用 Win32 API shim。将以下内容添加到你的 `App` 构造函数或 `Program.Main` 中：

```csharp
AvaloniaUI.Xpf.WinApiShim.WinApiShimSetup.AutoEnable();
```

有关详细信息，包括如何排除特定程序集，请参见 [Win32 API Shims](/xpf/third-party/win32-api-shims)。

**在 Windows 上也需要启用 shim 吗？**

在 Windows 上启用 shim 会将 Win32 调用重定向到 shim 层，而不是原生 Win32。这样通常是安全的，并且在开发期间可确保跨平台行为一致。不过，如果你只部署到 Windows，则不需要它们。

## 许可证

**是什么标识我的应用程序用于许可验证？**

XPF 在运行时验证两个标识：

1. **程序集名称**：`Assembly.GetEntryAssembly().GetName().Name`
2. **进程可执行文件名称**：正在运行的进程名称

这两个值都必须与注册在许可证中的值匹配。

**我可以将同一个许可证用于多个应用程序吗？**

每个许可证只覆盖一个应用程序（由程序集名称和进程名称标识）。不同的应用程序需要单独的许可证。

**我的许可证过期后会怎样？**

XPF 许可证是永久的。你的应用程序将无限期继续工作。许可证过期意味着你将不再收到更新或工程支持，但已部署的应用程序不受影响。

**如何开始试用？**

Internal 和 Business 许可证可通过 [Avalonia 网站](https://avaloniaui.net/xpf)申请为期 30 天的免费试用。你可以随时从门户中开始新的试用。Enterprise 许可证可通过联系销售获取。

## 平台支持

**XPF 支持 Android 和 iOS 吗？**

Android 和 iOS 支持适用于 Enterprise 许可证，目前处于私有预览阶段。有关设置说明，请参见 [Mobile and Browser](/xpf/platforms/mobile-and-browser)。

**XPF 支持 WebAssembly 吗？**

WebAssembly 支持适用于 Enterprise 许可证，目前处于私有预览阶段。有关设置说明，请参见 [Mobile and Browser](/xpf/platforms/mobile-and-browser)。

**支持哪些 Linux 发行版？**

所有 XPF 许可证都支持 Tier 1 Linux 发行版（最新版本的 Ubuntu、Fedora 和 Debian）。Enterprise 许可证还包括 Tier 2，以及按约定支持 Tier 3 发行版。有关完整的层级划分，请参见 [Supported Platforms](/docs/supported-platforms#desktop-linux)。

**XPF 支持 RHEL（Red Hat Enterprise Linux）吗？**

支持。RHEL 8 及更高版本均受支持。与 Ubuntu 相比，需要额外进行一些设置。有关 RHEL 特定的软件包安装说明，请参见 [Linux: Other Dependencies](/xpf/platforms/linux#other-dependencies)。

## 常见问题

**我的应用程序在 Windows 上可以运行，但在 macOS/Linux 上崩溃。我该从哪里开始？**

1. 检查你是否需要 [Win32 API shims](/xpf/third-party/win32-api-shims)（查看是否有 `DllNotFoundException` 错误）
2. 确保已安装所有 [Linux dependencies](/xpf/platforms/linux#other-dependencies)
3. 针对你的具体错误，查看 [Troubleshooting](/xpf/troubleshooting) 页面
4. 启用 [XPF logging](/xpf/troubleshooting#listening-for-xpf-logs) 以获取详细诊断信息

**为什么 `Assembly.GetEntryAssembly().Location` 返回 null？**

这是单文件发布应用程序在 .NET 5+ 中的行为，并非 XPF 特有。请改用 `AppDomain.CurrentDomain.BaseDirectory`。

**为什么字体在 Windows 和 Linux 上的渲染不同？**

Windows 和 Linux 使用不同的文本渲染后端，因此出现一些视觉差异是预期之中的。请确保你的自定义字体作为资源嵌入到 `.csproj` 中，并且 XAML 中的字体族名称与字体文件中的内部名称一致。有关配置详情，请参见 [Getting Started: Fonts](/xpf/getting-started#fonts)。

**如何在 macOS 上获取渲染缩放（DPI）？**

WPF API `VisualTreeHelper.GetDpi()` 在 macOS 上可能不会返回准确的值。请使用 Avalonia 互操作 API：

```csharp
using Atlantis;

var topLevel = XpfWpfAbstraction.GetAvaloniaTopLevelForWindow(myWpfWindow);
double scaling = topLevel.RenderScaling;
```

**我可以从 Visual Studio 发布我的 XPF 应用程序吗？**

强烈建议通过命令行（`dotnet publish`）进行发布。Visual Studio 发布可能会生成不完整的输出，缺少原生库（例如 `libSkiaSharp`）。有关正确的发布命令，请参见各平台特定的部署指南。

**如何启用 XPF 日志以便排查问题？**

在启动应用程序之前设置以下环境变量：
- `XPF_LOG_OUTPUT`：`console`、`trace` 或 `file=/path/to/log.txt`（可与 `;` 组合）
- `XPF_LOG_LEVEL`：`Verbose`、`Debug`、`Information`、`Warning`、`Error` 或 `Fatal`

有关详细信息，请参见 [Troubleshooting: Listening for XPF Logs](/xpf/troubleshooting#listening-for-xpf-logs)。

**我应该在 XPF 中使用哪个网页浏览器控件？**

这取决于你的目标平台。请参见 [Web Content Embedding](/xpf/interop/web-content)，其中对 CefSharp、NativeWebView、NativeWebDialog 和 DotNetBrowser 进行了并排比较。