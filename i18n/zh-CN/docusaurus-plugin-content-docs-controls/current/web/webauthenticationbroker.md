---
id: webauthenticationbroker
title: WebAuthenticationBroker
---

## 概述

`WebAuthenticationBroker` 是一个工具类，通过为桌面应用提供安全的 Web 身份验证处理方式，来简化 OAuth 和其他基于 Web 的身份验证流程。

## 静态方法

### AuthenticateAsync

```csharp
public static Task<WebAuthenticationResult> AuthenticateAsync(
    TopLevel topLevel, WebAuthenticatorOptions options)
```

通过导航到指定的起始 URI 并监视是否导航到结束 URI，来启动一个身份验证流程。

#### 参数

- `topLevel`：所属的顶级对象，在桌面平台上通常是一个窗口。
- `options`：控制 broker 行为的身份验证选项。

#### 返回值

一个包含身份验证结果的 `Task<WebAuthenticationResult>`。

## WebAuthenticationOptions

### 属性

```csharp
public Uri RequestUri { get; init; }
```

启动身份验证流程的初始 URI。

```csharp
public Uri RedirectUri { get; init; }
```

表示身份验证流程完成的 URI。

```csharp
public bool NonPersistent { get; init; }
```

提示平台实现不要持久化存储任何会话数据。

```csharp
public bool PreferNativeWebDialog { get; init; }
```

如果为 true，WebAuthenticationBroker 将避免使用平台特定实现，而改为使用基于 [NativeWebDialog](/controls/web/nativewebdialog) 的实现。

```csharp
public Func<NativeWebDialog?> NativeWebDialogFactory { get; init; }
```

当 WebAuthenticationBroker 使用对话框实现而非系统身份验证 API 时，可使用此回调覆盖 [NativeWebDialog](/controls/web/nativewebdialog) 的创建过程。

## WebAuthenticationResult

### 属性

```csharp
public Uri? CallbackUri { get; }
```

包含身份验证数据的响应 URI。

## 使用示例

本示例使用 Google OAuth。

最低配置要求如下：

1. 创建 Google 凭据（类型：Windows/macOS/Linux 使用 Desktop，或使用 iOS）。请参阅 [Console Credentials](https://console.cloud.google.com/apis/credentials)。此步骤中会创建 client ID 和 redirect URI。
2. 阅读 Google 的 [OAuth 2.0](https://developers.google.com/identity/protocols/oauth2/web-server#httprest) 文档以了解基本原理。

```csharp
var googleAuthRedirectUri = "http://localhost";
var googleAuthRequestUri = "https://accounts.google.com/o/oauth2/auth?response_type=code&access_type=offline&scope=openid";
googleAuthRequestUri += "&client_id=" + /* YOUR CLIENT ID */;
googleAuthRequestUri += "&redirect_uri=" + googleAuthRedirectUri;

var result = await WebAuthenticationBroker.AuthenticateAsync(
    mainWindow,
    new WebAuthenticatorOptions(
        RequestUri: new Uri(googleAuthRequestUri),
        RedirectUri: new Uri(googleAuthRedirectUri)));
```

同样的方式也适用于 [Microsoft identity](https://learn.microsoft.com/en-us/entra/identity-platform/v2-oauth2-auth-code-flow)、[Facebook Login](https://developers.facebook.com/docs/facebook-login/) 以及其他兼容 OAuth2 标准的方案。

## 平台支持
| 功能 | Windows | macOS (10.15+) | Linux | iOS (iOS 12.0+) | Android | Browser |
|-----------------------------|---------|-------|-------|-----|-----------|-----------|
| Platform Implementation | ✗ | ✓* | ✗ | ✓* | ✓** | ✓*** |
| NativeWebDialog | ✓ | ✓ | ✓ | ✗ | ✗ | ✗ |

\* Apple 平台使用 ASWebAuthenticationSession 实现。  
\** Android 使用 CustomTabsIntent 实现，但该支持仍属实验性，未来可能发生变化。  
\*** 浏览器方案要求配置 CORS，以允许访问重定向页面。此外，在浏览器中运行此库还需要 .NET 10。

## 另请参阅

- [NativeWebView](/controls/web/nativewebview)
- [NativeWebDialog](/controls/web/nativewebdialog)
- [WebView environment options](/controls/web/webview-environment)
- [Embedding web content](/docs/app-development/embedding-web-content)
- [FAQ](/tools/faq#webview)
