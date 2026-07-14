---
id: main-window
title: 主窗口
description: 在桌面、移动端和浏览器平台上设置并访问主窗口或主视图。
doc-type: explanation
---

主窗口是在 `App.axaml.cs` 文件的 `OnFrameworkInitializationCompleted` 方法中，赋值给 `ApplicationLifetime.MainWindow` 的那个窗口：

```csharp
public override void OnFrameworkInitializationCompleted()
{
    if (ApplicationLifetime is IClassicDesktopStyleApplicationLifetime desktopLifetime)
    {
        desktopLifetime.MainWindow = new MainWindow();
    }
}
```

你可以在任意时刻把 `Application.Current.ApplicationLifetime` 转换为 `IClassicDesktopStyleApplicationLifetime`，从而获取它。

需要注意的是，使用静态全局变量并在应用程序任意位置访问 `MainWindow` 可能存在风险，也有时会带来不佳的用户体验。所有与顶级对象（窗口）相关的 API，最好都从最具体的那个顶级对象发起调用，通常也就是当前最新激活的那个窗口。这样可以避免把用户对话框错误地从别的窗口弹出。

:::caution
在 Avalonia 中，移动端和浏览器平台并没有 `Window` 这一概念。在 iOS 和浏览器平台上，你需要通过 `ISingleViewApplicationLifetime` 设置 `MainView` 控件；而在 Android 上，则需要通过 `IActivityApplicationLifetime` 设置 `MainViewFactory`，因为 Android 可能会多次重建主 Activity。详情请参阅 [Application lifetimes](/docs/fundamentals/application-lifetimes)。
:::

## 另请参阅

- [Top level](/docs/fundamentals/top-level)
- [Application lifetimes](/docs/fundamentals/application-lifetimes)
