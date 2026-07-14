---
id: web-content
title: Web 内容嵌入
---

XPF 为在应用程序中嵌入 Web 内容提供了多种选项。正确的选择取决于您的目标平台和嵌入需求。

## 平台对比

| 功能 | CefSharp | NativeWebView | NativeWebDialog | DotNetBrowser |
|---|---|---|---|---|
| Windows | 支持 | 支持 | 支持 | 支持 |
| macOS | 不支持 | 支持 | 支持 | 支持 |
| Linux | 不支持 | 不支持 | 支持 | 支持 |
| 嵌入方式 | 窗口内控件 | 窗口内控件 | 独立对话框 | 窗口内控件 |
| 键盘输入 | 完整 | 完整 | 有限 | 完整 |
| 样式/CSS 控制 | 完整 | 完整 | 有限 | 完整 |
| 引擎 | Chromium | WebView2 / WebKit | WebKit / WebView2 | Chromium |

## XPF WebView（NativeWebView 和 NativeWebDialog）

`Avalonia.Xpf.Controls.WebView` NuGet 包提供两个控件：

- **NativeWebView**：可嵌入的 Web 控件，可在窗口内进行内联渲染。支持 Windows 和 macOS。
- **NativeWebDialog**：独立的浏览器对话框窗口。支持包括 Linux 在内的所有平台。

有关这些控件的 Avalonia 文档，请参阅[嵌入 Web 内容](/docs/app-development/embedding-web-content)。

### Linux 要求

Linux 上的 `NativeWebDialog` 需要 webkit2gtk 4.1 版本：

**Debian / Ubuntu:**
```bash
sudo apt install libwebkit2gtk-4.1-dev
```

**Fedora:**
```bash
sudo dnf install webkit2gtk4.1-devel
```

**RHEL / CentOS:**
```bash
sudo dnf install webkit2gtk4.1-devel
```

:::note
需要 webkit2gtk 4.1 版本。较旧的版本（4.0）不支持 `NativeWebDialog` 所需的全部功能。
:::

## CefSharp

`CefSharp.Wpf.NetCore` 仅在 Windows 上与 XPF 配合使用。它包含 Windows 原生的 Chromium 二进制文件，不能在 Linux 或 macOS 上运行。

如果 CefSharp 对 `CursorInteropHelper.Create()` 抛出 `NotImplementedException`，请升级到 XPF 1.6.0 或更高版本，该版本提供了回退方案。对于旧版本，可通过从 `ChromiumWebBrowser` 派生并重写 `OnCursorChange`，将 CefSharp 光标类型映射到 WPF 的 `Cursors` 作为临时解决方案。

## DotNetBrowser

TeamDev 的 DotNetBrowser 在 XPF 中支持所有平台。有关设置指南，请参阅 [XpfDotNetBrowserApp 示例](https://github.com/AvaloniaUI/Avalonia-XPF-Samples/tree/master/src/XpfDotNetBrowserApp)。

## 选择 Web 控件

- **仅部署到 Windows**：CefSharp 提供最完整的 Chromium 集成，具备完整的键盘、样式和 DevTools 支持。
- **Windows + macOS**：对于窗口内嵌入，请使用 `NativeWebView`；或者使用基于 Chromium 的 DotNetBrowser。
- **包括 Linux 在内的所有平台**：如果可以接受基于对话框的方式，请使用 `NativeWebDialog`；或者使用窗口内嵌入的 DotNetBrowser。