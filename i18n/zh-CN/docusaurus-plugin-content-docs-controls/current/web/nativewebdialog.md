---
id: nativewebdialog
title: NativeWebDialog
---

`NativeWebDialog` 是一个承载原生 Web 浏览器的对话框窗口。在 Linux 等无法使用嵌入式 `NativeWebView` 控件的平台上，它尤其有用；当你希望在单独窗口中显示 Web 内容，而不是将其嵌入布局时，它也很适合。

## 常用属性

| 属性 | 类型 | 说明 |
|---|---|---|
| `Title` | `string?` | 对话框窗口标题。 |
| `CanUserResize` | `bool` | 用户是否可以调整对话框大小。 |
| `Source` | `Uri` | WebView 中显示页面的 URI。设置此属性等价于调用 `Navigate()`。默认值：`about:blank`。 |
| `CanGoBack` | `bool` | 只读。当 WebView 可以在历史记录中后退时为 `true`。 |
| `CanGoForward` | `bool` | 只读。当 WebView 可以在历史记录中前进时为 `true`。 |

## 基本示例

创建一个对话框，导航到指定 URL，并等待它关闭：

```csharp
var dialog = new NativeWebDialog
{
    Title = "Avalonia Docs",
    CanUserResize = false,
    Source = new Uri("https://docs.avaloniaui.net/")
};

var tcs = new TaskCompletionSource();
dialog.Closing += (s, e) => tcs.SetResult();

dialog.Show(mainWindow);

await tcs.Task;
```

你也可以直接加载 HTML：

```csharp
var dialog = new NativeWebDialog { Title = "Preview" };
dialog.Show();
dialog.NavigateToString("<h1>Hello from Avalonia</h1>");
```

## 显示对话框

调用 `Show()` 可将对话框作为独立窗口打开，或使用 `Show(IPlatformHandle)` 将其附加到一个所有者窗口：

```csharp
// 独立窗口
dialog.Show();

// 指定所有者窗口
dialog.Show(mainWindow);
```

可以通过 `Close()` 以编程方式关闭它。`Closing` 事件会在对话框关闭前触发，方便你执行清理逻辑。

## 导航

| 方法 | 说明 |
|---|---|
| `Navigate(Uri)` | 导航到指定 URI。 |
| `NavigateToString(string)` | 将 HTML 字符串渲染为页面内容。 |
| `GoBack()` | 后退导航。若没有历史记录则返回 `false`。 |
| `GoForward()` | 前进导航。若没有历史记录则返回 `false`。 |
| `Refresh()` | 重新加载当前页面。 |
| `Stop()` | 停止任何正在进行的导航。 |

## 执行 JavaScript

在已加载的页面中执行 JavaScript，并接收返回结果：

```csharp
var result = await dialog.InvokeScript("document.title");
```

若要接收来自 JavaScript 的消息，可订阅 `WebMessageReceived`。Web 内容可通过调用 `invokeCSharpAction(body)` 发送消息：

```csharp
dialog.WebMessageReceived += (sender, e) =>
{
    var message = e.Body;
    // 处理来自 JavaScript 的消息
};
```

## 打印

| 方法 | 说明 |
|---|---|
| `ShowPrintUI()` | 打开平台打印对话框。 |
| `PrintToPdfStreamAsync()` | 将当前页面作为 PDF 流返回。 |

:::note
`PrintToPdfStreamAsync` 不支持边距、方向等扩展打印选项。为了获得更广泛的平台支持，建议使用 CSS 规则，例如 [@media print](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/At-rules/@media#print) 和 [@page](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/At-rules/@page)。
:::

## 拦截请求

当 WebView 发起 URL 请求时，会触发 `WebResourceRequested` 事件，这使你能够检查或修改请求头：

```csharp
dialog.WebResourceRequested += (sender, e) =>
{
    // 检查 e.Request 和 e.Headers
};
```

:::note
请求头字典可能会因为请求类型或平台不同而变为只读。请始终检查 `TrySet` 和 `TryRemove` 方法的返回结果。
:::

## 环境选项

`EnvironmentRequested` 事件会在 WebView 适配器创建之前触发，使你能够自定义诸如隐私模式、开发者工具等选项：

```csharp
dialog.EnvironmentRequested += (sender, e) =>
{
    // 在初始化前配置 WebView 环境
};
```

详情请参阅 [WebView 环境选项](/controls/web/webview-environment)。该事件的参数类型取决于平台。

## 窗口尺寸与位置

| 方法 | 说明 |
|---|---|
| `Resize(int, int)` | 将对话框调整为指定宽度和高度。 |
| `Move(int, int)` | 将对话框移动到指定屏幕坐标。 |

## 高级功能

| 方法 | 说明 |
|---|---|
| `TryGetCommandManager()` | 如果支持，则返回用于键盘命令（复制、粘贴等）的 `NativeWebViewCommandManager`。 |
| `TryGetCookieManager()` | 如果支持，则返回用于管理 Cookie 的 `NativeWebViewCookieManager`。 |
| `TryGetWebViewPlatformHandle()` | 返回托管 WebView 的平台句柄。详见 [嵌入 Web 内容](/docs/app-development/embedding-web-content)。 |
| `TryGetPlatformHandle()` | 返回对话框窗口本身的平台句柄。 |

## 事件

| 事件 | 说明 |
|---|---|
| `Closing` | 在对话框关闭前触发。 |
| `AdapterCreated` | 在 WebView 适配器初始化完成后触发。 |
| `AdapterDestroyed` | 在 WebView 适配器销毁后触发。 |
| `EnvironmentRequested` | 在 WebView 适配器创建前触发，用于配置环境选项。 |
| `NavigationStarted` | 在新的导航开始前触发。 |
| `NavigationCompleted` | 在导航完成后触发（无论成功与否）。 |
| `NewWindowRequested` | 当 WebView 请求打开新窗口时触发（例如来自 `window.open()`）。 |
| `WebMessageReceived` | 当 Web 内容调用 `invokeCSharpAction(body)` 时触发。 |
| `WebResourceRequested` | 当 WebView 发起 URL 请求时触发。 |

## 平台支持

| Feature | Windows | macOS | Linux | iOS | Android | Browser |
|---|---|---|---|---|---|---|
| `Show` | Yes | Yes | Yes | No | No | No |
| `Show(Window)` | Yes | Yes | Yes* | No | No | No |
| `WebMessageReceived` | Yes | Yes | No | No | No | No |
| `ShowPrintUI` | Yes | Yes | Yes | No | No | No |
| `PrintToPdfStreamAsync` | Yes | Yes** | Yes | No | No | No |

\* Linux 支持情况可能因窗口管理器不同而有所差异。

\** macOS 不支持扩展的 `PrintToPdfStreamAsync` 打印选项。请改用 CSS `@media print` 和 `@page` 规则。

## 另请参阅

- [NativeWebView](/controls/web/nativewebview)：可嵌入布局中的 WebView 控件。
- [WebAuthenticationBroker](/controls/web/webauthenticationbroker)：OAuth 和基于 Web 的身份验证流程。
- [WebView 环境选项](/controls/web/webview-environment)：配置 WebView 环境。
- [嵌入 Web 内容](/docs/app-development/embedding-web-content)：在 Avalonia 应用中承载 Web 内容。
- [常见问题](/tools/faq#webview)：常见 WebView 问题。
