---
id: nativewebview
title: NativeWebView
---

## 概述

`NativeWebView` 是一个为 Avalonia 和 WPF 应用提供原生 Web 浏览器实现的控件。它封装了平台专属的 Web 控件，并为 Web 浏览功能提供统一 API。

## 属性

### Source

```csharp
public Uri Source { get; set; }
```

这是 WebView 中显示的顶层文档 URI。设置该属性等价于调用 `Navigate()`。

默认值：`about:blank`

### CanGoBack

```csharp
public bool CanGoBack { get; }
```

表示 WebView 是否可以在导航历史中返回上一页。

### CanGoForward

```csharp
public bool CanGoForward { get; }
```

表示 WebView 是否可以在导航历史中前进到下一页。

## 事件

### AdapterCreated

```csharp
public event EventHandler<WebViewAdapterEventArgs>? AdapterCreated;
```

在底层 WebView 适配器初始化完成后触发。

### AdapterDestroyed

```csharp
public event EventHandler<WebViewNavigationCompletedEventArgs>? AdapterDestroyed;
```

在底层 WebView 适配器销毁后触发。

### EnvironmentRequested

```csharp
public event EventHandler<WebViewEnvironmentRequestedEventArgs>? EnvironmentRequested;
```

会在底层 WebView 适配器创建之前触发，使你能够自定义 WebView 环境。
你可以使用此事件在 WebView 初始化前修改环境选项（例如启用隐私模式或开发者工具）。
事件参数类型取决于平台。

详情请参阅[环境选项](/controls/web/webview-environment)页面。

### NavigationCompleted

```csharp
public event EventHandler<WebViewNavigationCompletedEventArgs>? NavigationCompleted;
```

在顶层文档的导航渲染完成后触发，无论成功还是失败。

### NavigationStarted

```csharp
public event EventHandler<WebViewNavigationStartingEventArgs>? NavigationStarted;
```

在顶层文档开始新一轮导航之前触发。

### NewWindowRequested

```csharp
public event EventHandler<WebViewNewWindowRequestedEventArgs>? NewWindowRequested;
```

在请求为顶层文档打开新窗口之前触发。

### WebMessageReceived

```csharp
public event EventHandler<WebMessageReceivedEventArgs>? WebMessageReceived;
```

当 Web 内容通过 `invokeCSharpAction(body)` 向宿主应用发送消息后触发。

### WebResourceRequested

```csharp
public event EventHandler<WebResourceRequestedEventArgs>? WebResourceRequested;
```

当 WebView 对匹配的 URL 发起请求时触发。
参数中包含请求信息以及请求头字典。

:::note
请求头字典可能会因为请求类型或平台不同而变为只读。
请始终检查 `TrySet` 和 `TryRemove` 方法的返回结果。
:::

#### 使用示例

双向 JS&lt;-&gt;C# 通信示例：

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

## 方法

### Navigate

```csharp
public void Navigate(Uri url)
```

将 WebView 导航到指定 URI。

### NavigateToString

```csharp
public void NavigateToString(string text)
```

将提供的 HTML 字符串渲染为顶层文档。

### InvokeScript

```csharp
public Task<string?> InvokeScript(string scriptName)
```

在顶层文档中执行给定的 JavaScript。

#### 使用示例

```xml
<NativeWebView Source="https://avaloniaui.net/" NavigationCompleted="WebView_NavigationCompleted" />
```

```csharp
private async void WebView_NavigationCompleted(object? sender, WebViewNavigationCompletedEventArgs args)
{
    // 执行 JavaScript
    await webView.InvokeScript("alert('Hello World')");
}
```

### GoBack

```csharp
public bool GoBack()
```

在导航历史中返回上一页。如果无法导航，则返回 `false`。

### GoForward

```csharp
public bool GoForward()
```

在导航历史中前进到下一页。如果无法导航，则返回 `false`。

### Refresh

```csharp
public bool Refresh()
```

重新加载当前页面。

### Stop

```csharp
public bool Stop()
```

停止任何正在进行的导航。

### ShowPrintUI

```csharp
void ShowPrintUI();
```

打开打印对话框以打印当前网页。

### PrintToPdfStreamAsync

```csharp
Task<Stream> PrintToPdfStreamAsync();
```

异步获取当前网页的 PDF 数据。

:::note

该 API 不接受边距、方向等扩展打印选项。
为了获得更广泛的平台支持，建议使用自定义 CSS 规则，例如 [@media print](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/At-rules/@media#print) 和 [@page](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/At-rules/@page)。

:::

### TryGetCommandManager

```csharp
public NativeWebViewCommandManager? TryGetCommandManager()
```

如果平台支持，则返回一个 `NativeWebViewCommandManager` 实例，用于执行常见键盘命令。

#### 使用示例

```csharp
var commandManager = webView.TryGetCommandManager();
if (commandManager != null)
{
    // 复制选中内容
    commandManager.Copy();
}
```

### TryGetCookieManager

```csharp
public NativeWebViewCookieManager? TryGetCookieManager()
```

如果平台支持，则返回一个 `NativeWebViewCookieManager` 实例，用于管理 Cookie。

#### 使用示例

```csharp
var cookieManager = webView.TryGetCookieManager();
if (cookieManager != null)
{
    // 获取所有 Cookie
    var cookies = await cookieManager.GetCookiesAsync();
}
```

### TryGetPlatformHandle

```csharp
public IPlatformHandle? TryGetPlatformHandle()
```

Returns a platform handle of the native control for accessing platform-specific APIs.
See the page on [embedding web content](/docs/app-development/embedding-web-content) for details.

### BeginReparenting

```csharp
public IDisposable BeginReparenting(bool yieldOnLayoutBeforeExiting = true)
```

Delays destruction of the native control during parent changes.

### BeginReparentingAsync

```csharp
public IAsyncDisposable BeginReparentingAsync()
```

Asynchronously delays destruction of the native control during parent changes.

## Platform support

| Feature                | Windows WebView2-Edge | macOS/iOS WKWebView | Linux WPE WebKit | Android | Browser |
|------------------------|-----------------------|---------------------|------------------|---------|---------|
| `NativeWebView`        | ✓                     | ✓                   | ✓                | ✓       | ✗*      |
| `TryGetCommandManager` | ✓                     | ✓                   | ✗*               | ✓       | ✗*      |
| `TryGetCookieManager`  | ✓                     | ✓                   | ✓                | ✓       | ✗*      |
| `ShowPrintUI`          | ✓                     | ✓                   | ✗*               | ✗*      | ✗*      |
| `PrintToPdfStreamAsync`| ✓                     | ✓**                 | ✗*               | ✗*      | ✗*      |

\* Not yet implemented while possible. If this is a blocker for your project, please open an issue.

\** macOS does not allow extended PrintToPdfStreamAsync print options. Use custom CSS rules instead - [@media print](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/At-rules/@media#print) and [@page](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/At-rules/@page).

:::note

On Linux, `NativeWebView` is rendered through [WPE WebKit](https://wpewebkit.org) using offscreen (SHM) rendering. Install the `libwpewebkit-2.0`, `libwpe-1.0`, and `libWPEBackend-fdo-1.0` runtime libraries — see the [Linux prerequisites](/docs/app-development/embedding-web-content#linux). If WPE is unavailable, you can opt in to the WebKitGTK adapter via [`LinuxWpeWebViewEnvironmentRequestedEventArgs.PreferWebKitGtkInstead`](/controls/web/webview-environment#linux-wpe-webkit), or fall back to [`NativeWebDialog`](/controls/web/nativewebdialog).

:::

## See also

- [NativeWebDialog](/controls/web/nativewebdialog)
- [WebAuthenticationBroker](/controls/web/webauthenticationbroker)
- [WebView environment options](/controls/web/webview-environment)
- [Embedding web content](/docs/app-development/embedding-web-content)
- [FAQ](/tools/faq#webview)
