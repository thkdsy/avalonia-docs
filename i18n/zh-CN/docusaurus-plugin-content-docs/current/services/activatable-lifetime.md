---
id: activatable-lifetime
title: 可激活生命周期
description: IActivatableLifetime 的 API 参考，该服务公开应用激活与停用事件以及后台状态相关方法。
doc-type: reference
---

[`IActivatableLifetime`](/api/avalonia/controls/applicationlifetimes/iactivatablelifetime) 服务定义了一组与应用激活和停用生命周期相关的方法与事件。`IActivatableLifetime` 是一个全局的应用级服务，可通过应用实例上的 `TryGetFeature` 方法访问：

```csharp
Application.Current.TryGetFeature<IActivatableLifetime>();
```

## 事件

### Activated

当应用因 `ActivationKind` 枚举中描述的各种原因被 `Activated` 时触发的事件。

### Deactivated

当应用因 `ActivationKind` 枚举中描述的各种原因被 `Deactivated` 时触发的事件。

## 方法

### TryLeaveBackground

通知应用尝试离开后台状态。

如果当前平台支持该操作，则返回 `true`；否则返回 `false`。

**示例：** macOS 上的 `[NSApp unhide]`。

### TryEnterBackground

通知应用尝试进入后台状态。

如果当前平台支持该操作，则返回 `true`；否则返回 `false`。

**示例：** macOS 上的 `[NSApp hide]`。

## 提前订阅启动事件

如果你希望接收应用启动时发生的激活事件（例如用户双击某个关联文件来启动应用），请在 `OnFrameworkInitializationCompleted` 中、任何 `await` 调用之前订阅 `Activated`。如果处理程序附加得太晚，启动阶段的激活事件可能已经触发，从而被错过。

```csharp
public override void OnFrameworkInitializationCompleted()
{
    if (ApplicationLifetime is IClassicDesktopStyleApplicationLifetime desktopLifetime)
    {
        desktopLifetime.MainWindow = new MainWindow();

        if (this.TryGetFeature<IActivatableLifetime>() is { } activatableLifetime)
        {
            activatableLifetime.Activated += (_, e) =>
            {
                if (e is ProtocolActivatedEventArgs protocolArgs)
                {
                    Console.WriteLine($"Protocol: {protocolArgs.Uri}");
                }
                else if (e is FileActivatedEventArgs fileArgs)
                {
                    Console.WriteLine($"File: {fileArgs.Files.FirstOrDefault()?.Path}");
                }
            };
        }
    }

    base.OnFrameworkInitializationCompleted();
}
```

:::caution
请为每种激活类型使用正确的事件参数类型。`ProtocolActivatedEventArgs` 用于处理 URI/深链接激活（`ActivationKind.OpenUri`）。`FileActivatedEventArgs` 用于处理文件关联激活（`ActivationKind.File`）。如果在打开文件时去检查 `ProtocolActivatedEventArgs`，将不会匹配成功，表现出来就像事件没有触发一样。
:::

## 示例

### 进入与退出后台状态

你可能希望应用在进入后台时暂停或停止某些代码处理，例如暂停多媒体播放，或者停止周期性 HTTP 请求。

```csharp
if (Application.Current?.TryGetFeature<IActivatableLifetime>() is { } activatableLifetime)
{
    activatableLifetime.Activated += (sender, args) =>
    {
        if (args.Kind == ActivationKind.Background)
        {
            Console.WriteLine($"App exited background");
        }
    };
    activatableLifetime.Deactivated += (sender, args) =>
    {
        if (args.Kind == ActivationKind.Background)
        {
            Console.WriteLine($"App entered background");
        }
    };
}
```

### 处理 URI 激活

你的应用可能需要支持协议激活，也就是更常说的深链接。链接 scheme 必须先在系统中注册并与应用关联。注册完成后，操作系统就可以将这些链接重定向到你的应用。

典型用例包括跳转到特定页面，或在 OAuth 流程中创建[重定向 URL](https://www.oauth.com/oauth2-servers/oauth-native-apps/redirect-urls-for-native-apps/)。

```csharp
if (Application.Current?.TryGetFeature<IActivatableLifetime>() is { } activatableLifetime)
{
    activatableLifetime.Activated += (s, a) =>
   {
        if (a is ProtocolActivatedEventArgs protocolArgs && protocolArgs.Kind == ActivationKind.OpenUri)
        {
            Console.WriteLine($"App activated via Uri: {protocolArgs.Uri}");
        }
   };
}
```

:::note
某些平台需要额外更新清单文件，才能启用协议处理。

**macOS 和 iOS：** 在你的 `Info.plist` 中添加带有 `CFBundleURLSchemes` 段的 `CFBundleURLTypes`。可参阅 [Creating an app custom URL scheme](https://rderik.com/blog/creating-app-custom-url-scheme/)（忽略其中的 Swift 部分，这部分由 `IActivatableLifetime` 处理）。

**Android：** 在 `AndroidManifest.xml` 中添加带特定 `android:scheme` 的 `intent-filter`。详情请参阅 [Deep linking on Android](https://developer.android.com/training/app-links/deep-linking)（忽略 Kotlin/Java 部分，这部分由 `IActivatableLifetime` 处理）。
:::

### 处理文件激活

你的应用可能需要处理文件激活，即当操作系统启动你的应用或将其切换到前台时发生的激活（通常是因为用户打开了与你的应用关联的文件）。与链接 scheme 一样，文件类型关联也必须先在系统中注册并链接到你的应用。注册完成后，打开关联文件时也会通过此事件打开你的应用。

典型用例包括打开文档、导入文件，或处理来自操作系统 Shell 传入的文件。

```csharp
if (Application.Current?.TryGetFeature<IActivatableLifetime>() is { } activatableLifetime)
{
    activatableLifetime.Activated += (s, a) =>
    {
        if (a is FileActivatedEventArgs fileArgs && fileArgs.Kind == ActivationKind.File)
        {
            foreach (var file in fileArgs.Files)
            {
                Console.WriteLine($"App activated via file: {file.Name}");
            }
        }
    };
}
```

:::note
某些平台需要额外更新清单文件，才能启用文件类型关联。

**macOS 和 iOS：** 在 `Info.plist` 中添加 `CFBundleDocumentTypes`，用于声明应用可处理的文件类型。详情请参阅 [Apple 文档](https://developer.apple.com/documentation/bundleresources/information_property_list/cfbundledocumenttypes)。

**Android：** 在 `AndroidManifest.xml` 中添加带有 `action.VIEW` 以及适当 `data` MIME 类型或文件扩展名的 `intent-filter`。详情请参阅 [Android 文档](https://developer.android.com/training/data-storage/shared/documents-files)（忽略 Kotlin/Java 部分，因为这些由 `IActivatableLifetime` 处理）。
:::

## 平台兼容性

| 功能 | Windows | macOS | Linux | Browser | Android | iOS |
|---------------|-------|-------|-------|-------|-------|-------|
| `ActivationKind.Background` | ✖ | ✔ | ✖ | ✔ | ✔ | ✔ |
| `ActivationKind.File` | ✖ | ✔ | ✖ | ✖ | ✔ | ✔ |
| `ActivationKind.OpenUri` | ✖ | ✔ | ✖ | ✖ | ✔ | ✔ |
| `ActivationKind.Reopen` | ✖ | ✔ | ✖ | ✖ | ✖ | ✖ |
| `TryLeaveBackground`  | ✖ | ✔ | ✖ | ✖ | ✖ | ✖ |
| `TryEnterBackground` | ✖ | ✔ | ✖ | ✖ | ✔ | ✖ |

## 另请参阅

- [IActivatableLifetime issue and discussion (#15316)](https://github.com/AvaloniaUI/Avalonia/issues/15316)
