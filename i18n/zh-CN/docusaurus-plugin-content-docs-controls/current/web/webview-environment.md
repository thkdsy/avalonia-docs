---
id: webview-environment
title: WebView environment
---

## 概述

WebView 环境选项允许你在底层浏览器引擎初始化之前对其进行自定义。这对于配置开发者工具、隐私浏览、用户数据目录以及其他必须在创建阶段设置的平台专属特性来说非常重要。

`EnvironmentRequested` 事件会在 WebView 适配器创建之前触发，使你能够根据应用需求修改这些设置。

## 基本用法

```csharp
var webView = new WebView();
webView.EnvironmentRequested += (sender, args) =>
{
    // 为所有平台启用开发者工具
    args.EnableDevTools = true;
    
    // 平台专属配置
    switch (args)
    {
        case WindowsWebView2EnvironmentRequestedEventArgs webView2Args:
            webView2Args.IsInPrivateModeEnabled = true;
            break;
        case AppleWKWebViewEnvironmentRequestedEventArgs appleArgs:
            appleArgs.NonPersistentDataStore = true;
            break;
        case GtkWebViewEnvironmentRequestedEventArgs gtkArgs:
            gtkArgs.EphemeralDataManager = true;
            break;
    }
};
```

## 基类属性

### WebViewEnvironmentRequestedEventArgs

**属性：**

- `EnableDevTools` (bool)：控制用户是否可以通过上下文菜单或键盘快捷键打开开发者工具。所有平台均可用。

## 平台专属选项

### Windows WebView2

**关键属性：**

- `ExplicitEnvironment`：使用现有的 ICoreWebView2Environment COM 句柄
- `ProfileName`：设置自定义浏览器配置文件名称
- `BrowserExecutableFolder`：指定 Edge 浏览器可执行文件位置
- `UserDataFolder`：定义用户数据的存储位置
- `AdditionalBrowserArguments`：传递自定义 Chromium 命令行参数
- `Language`：设置浏览器 UI 语言（BCP 47 格式）
- `IsInPrivateModeEnabled`：启用隐私浏览模式

**示例：**

```csharp
webView.EnvironmentRequested += (sender, args) =>
{
    if (args is WindowsWebView2EnvironmentRequestedEventArgs webView2)
    {
        webView2.ProfileName = "AvaloniaUser";
        webView2.UserDataFolder = Path.Combine(AppContext.BaseDirectory, "webview");
    }
};
```

### macOS/iOS（WKWebView）

**关键属性：**

- `NonPersistentDataStore`：使用仅保存在内存中的数据存储
- `DataStoreIdentifier`：为持久化数据设置唯一标识符
- `ApplicationNameForUserAgent`：自定义用户代理中的应用名称
- `UpgradeKnownHostsToHTTPS`：自动将 HTTP 升级为 HTTPS
- `LimitsNavigationsToAppBoundDomains`：限制导航只能访问应用绑定域名

**示例：**

```csharp
webView.EnvironmentRequested += (sender, args) =>
{
    if (args is AppleWKWebViewEnvironmentRequestedEventArgs wkWebView)
    {
        wkWebView.NonPersistentDataStore = true;
        wkWebView.ApplicationNameForUserAgent = "Avalonia WebView Sample";
    }
};
```

### Linux（WPE WebKit）

`NativeWebView` 在 Linux 上的默认后端是 [WPE WebKit](https://wpewebkit.org)，它采用离屏渲染，并将结果合成到 Avalonia 可视树中。

**关键属性：**

- `DataDirectory`：用于持久化网站数据的目录。为 `null` 时使用默认 WPE 数据目录。
- `CacheDirectory`：用于网站缓存的目录。为 `null` 时使用默认 WPE 缓存目录。
- `RenderingMode`：选择 WPE 渲染后端（`WpeRenderingMode`）。默认值 `Auto` 当前会映射为 `Shm`（软件渲染，不需要 GPU）。`Egl` 和 `DmaBuf` 预留给未来使用，若当前选择会抛出 `NotImplementedException`。该选项是进程级全局设置，会影响所有 `NativeWebView` 实例。
- `PreferWebKitGtkInstead`：当为 `true` 时，即便 WPE 可用，也会回退到 WebKitGTK 适配器。

**示例：**

```csharp
webView.EnvironmentRequested += (sender, args) =>
{
    if (args is LinuxWpeWebViewEnvironmentRequestedEventArgs wpeArgs)
    {
        wpeArgs.DataDirectory = Path.Combine(AppContext.BaseDirectory, "wpe-data");
        wpeArgs.CacheDirectory = Path.Combine(AppContext.BaseDirectory, "wpe-cache");
    }
};
```

### Linux（GTK WebKit）

该后端由 `NativeWebDialog` 使用；如果设置了 `PreferWebKitGtkInstead`，或者 WPE 不可用，`NativeWebView` 也会使用它。

**关键属性：**

- `ApplicationNameForUserAgent`：自定义用户代理中的应用名称
- `ExperimentalOffscreen`：启用实验性的离屏渲染
- `EphemeralDataManager`：使用非持久化数据存储
- `BaseDataDirectory`：设置网站数据的基础目录
- `BaseCacheDirectory`：设置缓存的基础目录
- `SharedProcessModel`：为所有 WebView 实例使用共享进程模型
- `DisableCache`：完全禁用缓存，以优化内存占用

**示例：**

```csharp
webView.EnvironmentRequested += (sender, args) =>
{
    if (args is GtkWebViewEnvironmentRequestedEventArgs gtkArgs)
    {
        gtkArgs.EphemeralDataManager = true;
        gtkArgs.EnableDevTools = true;
    }
};
```

## 另请参阅

- [NativeWebView](/controls/web/nativewebview)
- [NativeWebDialog](/controls/web/nativewebdialog)
- [WebAuthenticationBroker](/controls/web/webauthenticationbroker)
- [嵌入 Web 内容](/docs/app-development/embedding-web-content)
- [常见问题](/tools/faq#webview)
