---
id: embedding-web-content
title: 嵌入 Web 内容
description: 在 Avalonia 应用中使用 NativeWebView、NativeWebDialog 和 WebAuthenticationBroker 嵌入 Web 内容。
doc-type: how-to
tags:
  - xpf
---

## 概览

Avalonia WebView 组件为你的 Avalonia 应用提供了原生网页浏览能力。与那些需要打包 Chromium 的嵌入式 WebView 方案不同，这种实现直接利用平台原生的网页渲染能力，因此应用体积更小、性能也更好。

WebView 组件主要包含以下三个 API：

- [`NativeWebView`](/controls/web/nativewebview) - 用于将 Web 内容直接嵌入应用 UI 的控件
- [`NativeWebDialog`](/controls/web/nativewebdialog) - 用于承载 Web 内容的独立对话框窗口
- [`WebAuthenticationBroker`](/controls/web/webauthenticationbroker) - 用于处理 OAuth 和基于 Web 的身份验证流程的工具

WebView 组件同时适用于 Avalonia 和 [Avalonia XPF](/xpf)。有关 XPF 专属的安装和使用方式，请参阅下方的 [XPF](#xpf) 一节。

## 安装

将 WebView 包添加到你的项目中：

```bash
dotnet add package Avalonia.Controls.WebView
```

## 基本用法

### NativeWebView

:::note
在 Linux 上，`NativeWebView` 使用 [WPE WebKit](https://wpewebkit.org) 并以离屏方式渲染。请确保目标系统已安装 WPE 运行库，详见 [Linux 前置条件](#linux)。如果目标系统上没有 WPE，请改用 [`NativeWebDialog`](/controls/web/nativewebdialog)。
:::

```xml
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">

    <NativeWebView Source="https://avaloniaui.net/"
                   NavigationCompleted="WebView_NavigationCompleted" />
</Window>
```

```csharp
private void WebView_NavigationCompleted(object? sender, WebViewNavigationCompletedEventArgs args)
{
    if (args.IsSuccess)
    {
        // 导航已成功完成
    }
}
```

#### 双向 JavaScript 执行

在某些场景下，你需要从 WebView 控件中执行任意 JavaScript 代码。
`NativeWebView` 提供了 [`InvokeScript` async method](/controls/web/nativewebview#invokescript)：

```csharp
webView.InvokeScript("console.log('Hello World')");
```

如果你还需要从 JavaScript（网页端）接收数据并在 C# 侧处理，可以结合使用 `NativeWebView.WebMessageReceived` 事件和 `invokeCSharpAction` 辅助 JS 方法。

完整的双向通信示例如下：
```csharp
private async void NativeWebView_OnNavigationCompleted(object? sender, WebViewNavigationCompletedEventArgs e)
{
    await ((NativeWebView)sender!).InvokeScript(""" invokeCSharpAction("{'key': 10}") """);
}

private void NativeWebView_OnWebMessageReceived(object? sender, WebMessageReceivedEventArgs e)
{
    var message = e.Body;
    // message == "{'key': 10}"
}
```

![NativeWebView control displaying web content in an Avalonia window](/img/webview.png)

### NativeWebDialog

```csharp
var dialog = new NativeWebDialog
{
    Title = "Avalonia Docs",
    CanUserResize = true,
    Source = new Uri("https://docs.avaloniaui.net/")
};

dialog.NavigationCompleted += (s, e) =>
{
    if (e.IsSuccess)
    {
        // 导航已成功完成
    }
};

dialog.Show();
```

### WebAuthenticationBroker

```csharp
var authOptions = new WebAuthenticatorOptions(
    RequestUri: new Uri("https://accounts.google.com/o/oauth2/auth?response_type=code&client_id=YOUR_CLIENT_ID&redirect_uri=http://localhost&scope=openid"),
    RedirectUri: new Uri("http://localhost")
);

var result = await WebAuthenticationBroker.AuthenticateAsync(mainWindow, authOptions);

if (result.CallbackUri != null)
{
    // 处理认证结果
    var code = HttpUtility.ParseQueryString(result.CallbackUri.Query)["code"];
}
```

请将 `YOUR_CLIENT_ID` 替换为你自己应用的客户端 ID。

## 平台前置条件

WebView 组件依赖目标设备上可用的原生网页渲染实现。

#### 汇总

| 组件 | Windows | macOS | Linux | iOS | Android | Browser |
|-----------|---------|-------|-------|-----|---------|---------|
| NativeWebView | ✓ | ✓ | ✓* | ✓ | ✓ | ✗ |
| NativeWebDialog | ✓ | ✓ | ✓ | ✗ | ✗ | ✗ |
| WebAuthenticationBroker | ✓** | ✓ | ✓** | ✓ | ✓*** | ✓**** |

\* 在 Linux 上，`NativeWebView` 使用 WPE WebKit。如果目标系统上没有 WPE，请回退到 `NativeWebDialog`（它使用 WebKitGTK）。
\** 使用 NativeWebDialog 实现
\*** Android 支持仍属实验性
\**** 需要为重定向页面配置 CORS，同时在浏览器中运行该库还需要 .NET 10。

#### Windows

使用 Microsoft Edge WebView2，其特点如下：

- Windows 11 预装
- 在 Windows 10 上可能需要额外安装

对于 Windows 10 用户，你可以将 WebView2 运行时与安装程序一并分发：

- [WebView2 Runtime Download](https://developer.microsoft.com/en-us/microsoft-edge/webview2?form=MA13LH#download)
- [Distribution Guide](https://learn.microsoft.com/en-us/microsoft-edge/webview2/concepts/distribution?tabs=dotnetcsharp)

#### macOS/iOS

使用 `WKWebView`，它在所有现代 macOS/iOS 设备上都已预装。

- 不需要额外配置
- 对于 WebAuthenticationBroker：要求 macOS 10.15+ 或 iOS 12.0+

#### Linux

Linux 会根据你使用的 API 采用两种不同的后端：

- **`NativeWebView`** 使用 [WPE WebKit](https://wpewebkit.org) 进行离屏（SHM）渲染。WPE 不强依赖 GTK，因此这个嵌入控件既可以在 X11 会话中工作，也可以在 Wayland 会话中工作，并且能够被组合进 Avalonia 的视觉树中。
- **`NativeWebDialog`** 在一个独立的 GTK 窗口中使用 WebKitGTK。当 WPE 不可用，或者你更希望使用独立浏览器窗口时，可以把它作为回退方案。

如果使用 `NativeWebView`，请安装 WPE 运行时库：

Debian/Ubuntu (24.04+):

```bash
sudo apt install libwpewebkit-2.0-1
```

Fedora:

```bash
sudo dnf install dnf-plugins-core
sudo dnf copr enable philn/wpewebkit
sudo dnf install wpewebkit
```

Arch:

```bash
sudo pacman -S wpewebkit
```

如果使用 `NativeWebDialog`，请安装 GTK 3.0 和 WebKitGTK 4.1：

Debian/Ubuntu:

```bash
apt install libgtk-3-0 libwebkit2gtk-4.1-0
```

Fedora:

```bash
dnf install gtk3 webkit2gtk4.1
```

:::note
对于较旧的 Ubuntu 发行版，`NativeWebDialog` 也支持 `libwebkit2gtk-4.0` 和 `soup-2.4`。不过更推荐使用 `libwebkit2gtk-4.1`。
:::

:::note
如果系统没有打包 WPE，请改用 `NativeWebDialog`，或者设置 [`LinuxWpeWebViewEnvironmentRequestedEventArgs.PreferWebKitGtkInstead`](/controls/web/webview-environment#linux-wpe-webkit) 以回退到 WebKitGTK 适配器。
:::

#### Android

要求 Android API 21 或更高版本。

## 原生浏览器互操作

Avalonia WebView 组件通过使用平台原生 WebView，提供了跨平台的网页内容渲染能力。
不过，在某些情况下，你仍然需要访问那些没有通过 Avalonia WebView 抽象层暴露出来的平台专属 API。

本节将说明如何获取原生句柄，并与各个受支持平台上的底层浏览器实现进行互操作。

### 获取句柄

要访问原生浏览器功能，第一步是从 WebView 控件中获取平台专属句柄。

#### 对于 WebView 控件

在 WebView 实例上使用 `TryGetPlatformHandle()` 方法：

```csharp
if (myWebView.TryGetPlatformHandle() is IWindowsWebView2PlatformHandle handle)
{
    // 转换为平台专属接口后使用
}
```

#### 对于 WebView 对话框

在 WebView 对话框实例上使用 `TryGetWebViewPlatformHandle()` 方法：

```csharp
if (myWebViewDialog.TryGetWebViewPlatformHandle() is IWindowsWebView2PlatformHandle handle)
{
    // 转换为平台专属接口后使用
}
```

### 互操作

#### Windows

Avalonia 在 Windows 上的 WebView 支持两种适配器：

- **WebView2**：现代的 Chromium 内核 Edge（推荐）
- **WebView1**：旧版 Edge（适用于未安装 WebView2 的旧 Windows 10 设备的回退方案）

这两种适配器都通过传统 COM 互操作工作。
*IDL* 定义文件可以在 `Microsoft.Web.WebView2` NuGet 包中（对应 WebView2）、Windows SDK 中（对应 WebView1）或互联网上找到。

**推荐方式**：使用新的 [`[GeneratedComInterface]`](https://learn.microsoft.com/en-us/dotnet/standard/native-interop/comwrappers-source-generation) 特性，以获得更快、且对 trimmer/AOT 更友好的 COM 互操作。

**替代方案**：

- [CsWin32 generators](https://github.com/microsoft/CsWin32)
- [Legacy `[ComImport]`](https://learn.microsoft.com/en-us/dotnet/standard/native-interop/cominterop)

```csharp
public interface IWindowsWebView2PlatformHandle : IPlatformHandle
{
    /// Returns COM handle to the ICoreWebView2 [76ECEACB-0462-4D94-AC83-423A6793775E] COM interface
    IntPtr CoreWebView2 { get; }
    /// Returns COM handle to the ICoreWebView2 [4D00C0D1-9434-4EB6-8078-8697A560334F] COM interface
    IntPtr CoreWebView2Controller { get; }
}
```

```csharp
public interface IWindowsWebView1PlatformHandle : IPlatformHandle
{
    /// Returns COM handle to the IWebViewControl [3F921316-BC70-4BDA-9136-C94370899FAB] COM interface.
    IntPtr WebViewControl { get; }
}
```

#### MacOS/iOS

**推荐方式**：使用官方的 .NET Xamarin.Native macOS/iOS 绑定，以获得强类型包装。典型用法是 [NSObject.GetNSObject\<WKWebView\>(IntPtr, false)](https://learn.microsoft.com/en-us/dotnet/api/objcruntime.runtime.getnsobject?view=xamarin-ios-sdk-12#objcruntime-runtime-getnsobject-1(system-intptr-system-boolean))。

```csharp
var wkWebView = NSObject.GetNSObject<WKWebView>(handle.WKWebView, false);
```

**替代方式**：使用 `objc_msgSend` 的 P/Invoke 直接访问原生 API（控制力更强，但维护难度更高）。

```csharp
public interface IAppleWKWebViewPlatformHandle : IPlatformHandle
{
    IntPtr WKWebView { get; }
    IntPtr GetWKWebViewRetained();
}
```

#### Linux (WPE WebKit)

在 Linux 上，`NativeWebView` 由 WPE WebKit 提供支持。平台句柄会暴露 `WebKitWebView` GObject 以及底层的 `wpe_view_backend` 结构体，从而允许你直接对 [WPEWebKit API](https://github.com/WebPlatformForEmbedded/WPEWebKit) 进行 P/Invoke 调用。

```csharp
public interface ILinuxWpePlatformHandle : IPlatformHandle
{
    /// Pointer to the WebKitWebView GObject instance.
    IntPtr WebKitWebView { get; }

    /// Pointer to the wpe_view_backend native struct.
    IntPtr WpeViewBackend { get; }
}
```

#### Linux (WebKitGTK)

`NativeWebDialog`（以及被配置为优先使用 WebKitGTK 的 `NativeWebView`）会暴露一个 WebKitGTK 句柄。提供的 `WebKitWebView` IntPtr 可以直接配合 [官方 WebKitGTK 文档](https://webkitgtk.org/reference/webkit2gtk/stable/index.html) 中的 WebKit P/Invoke 使用。

```csharp
public interface IGtkWebViewPlatformHandle : IPlatformHandle
{
    IntPtr WebKitWebView { get; }
}
```

#### Android

为了最方便地获取托管包装访问，建议使用官方的 .NET Xamarin.Android 绑定。

有关用法细节，请参阅 [Android.Webkit.WebView 文档](https://learn.microsoft.com/en-us/dotnet/api/android.webkit.webview.-ctor?view=net-android-35.0#android-webkit-webview-ctor(system-intptr-android-runtime-jnihandleownership))。

```csharp
public interface IAndroidWebViewPlatformHandle : IPlatformHandle
{
    IntPtr WebKitWebView { get; }
}
```

## XPF

WebView 组件同样可用于 [Avalonia XPF](/xpf) 应用。上文所述的所有 WebView 功能、API 和平台前置条件同样适用于 XPF，只是下面这些地方有所不同。

### 安装

首先，请确保你已经按照 [说明](/xpf/version-info/versioning) 配置好了 XPF 的 NuGet 源。

配置好 NuGet 源后，安装 `Avalonia.Xpf.Controls.WebView` 包：

```xml
<PackageReference Include="Avalonia.Xpf.Controls.WebView" Version="11.3.9" />
```

:::note
如果有新版本，建议使用最新版本。你可以在 IDE 的 NuGet Packages 窗口中检查更新。

在 Windows 上，如果 WebView2 不可用，则会嵌入旧版 Internet Explorer。这在面向较老 Windows 版本时会很有用。
:::

### 使用方式

将 XPF 命名空间添加到你的 XAML 文件中，然后使用 `NativeWebView`：

```xml
<wpf:NativeWebView xmlns:wpf="clr-namespace:Avalonia.Xpf.Controls;assembly=Avalonia.Xpf.Controls.WebView"
                   Source="https://avaloniaui.net/" />
```

`Source` 属性支持绑定。其余 API（`NativeWebDialog`、`WebAuthenticationBroker`、JavaScript 互操作）都与上文描述的行为一致。

为了简化代码迁移，你也可以在 Windows 的原生 WPF 中直接使用 `NativeWebView` 控件，而不依赖 XPF。在这种场景下，API 成员和底层浏览器实现都是相同的，并且仍然使用同一个包。

## 另请参阅

- [NativeWebView](/controls/web/nativewebview)
- [NativeWebDialog](/controls/web/nativewebdialog)
- [WebAuthenticationBroker](/controls/web/webauthenticationbroker)
- [WebView environment options](/controls/web/webview-environment)
- [FAQ](/tools/faq#webview)
