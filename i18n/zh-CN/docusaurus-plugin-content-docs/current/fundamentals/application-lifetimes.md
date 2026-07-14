---
id: application-lifetimes
title: 应用程序生命周期
description: 为桌面、移动端和浏览器平台选择并配置应用程序生命周期模型。
doc-type: explanation
---

并非所有平台都完全一样。例如，你在 Windows Forms 或 WPF 中熟悉的生命周期管理方式，只能运行在桌面类平台上。Avalonia 是一个跨平台框架；为了让你的应用具有可移植性，它为应用程序提供了多种不同的生命周期模型；如果目标平台允许，你也可以完全手动控制整个生命周期。

## 生命周期如何工作？

对于桌面应用程序，你通常会这样初始化：

```csharp
class Program
{
  // This method is needed for IDE previewer infrastructure
  public static AppBuilder BuildAvaloniaApp() 
    => AppBuilder.Configure<App>().UsePlatformDetect();

  // The entry point. Things aren't ready yet, so at this point
  // you shouldn't use any Avalonia types or anything that expects
  // a SynchronizationContext to be ready
  public static int Main(string[] args) 
    => BuildAvaloniaApp().StartWithClassicDesktopLifetime(args);
}
```

然后在 `Application` 类中创建主窗口：

```csharp
public override void OnFrameworkInitializationCompleted()
{
    if (ApplicationLifetime
            is IClassicDesktopStyleApplicationLifetime desktop)
        desktop.MainWindow = new MainWindow();
    else if (ApplicationLifetime
            is IActivityApplicationLifetime activityLifetime)
        activityLifetime.MainViewFactory = () => new MainView();
    else if (ApplicationLifetime
            is ISingleViewApplicationLifetime singleView)
        singleView.MainView = new MainView();
    base.OnFrameworkInitializationCompleted();
}
```

该方法会在框架初始化完成后调用，此时 `ApplicationLifetime` 属性会包含当前所选择的生命周期对象（如果有）。

:::info
如果你以设计模式运行应用程序（即使用 IDE 预览器进程），那么 `ApplicationLifetime` 会是 null。
:::

## 生命周期接口

Avalonia 提供了一组接口，让你可以选择适合自己应用程序的控制粒度。这些接口由 `BuildAvaloniaApp().Start[Something]` 这一系列方法提供。

### IControlledApplicationLifetime

由以下方法提供：

* `StartWithClassicDesktopLifetime`
* `StartLinuxFramebuffer`

它允许你订阅 `Startup` 和 `Exit` 事件，并且可以通过调用 `Shutdown` 方法显式关闭应用程序。这个接口让你能够控制应用程序的退出流程。

### IClassicDesktopStyleApplicationLifetime

继承自：`IControlledApplicationLifetime`

由以下方法提供：

* `StartWithClassicDesktopLifetime`

它允许你以类似 Windows Forms 或 WPF 应用程序的方式来控制应用生命周期。这个接口提供了访问当前已打开窗口列表、设置主窗口的能力，并支持三种关闭模式：

* `OnLastWindowClose` - 当最后一个窗口关闭时关闭应用程序
* `OnMainWindowClose` - 当主窗口关闭时关闭应用程序（如果已设置主窗口）
* `OnExplicitShutdown` - 禁用应用程序自动关闭，你需要在代码中手动调用 `Shutdown` 方法

### ISingleViewApplicationLifetime

由以下方式提供：

* `StartLinuxFramebuffer`
* iOS
* web platform (WebAssembly/WASM)

某些平台并没有桌面主窗口这一概念，并且设备屏幕同一时间只允许显示一个视图。对于这些平台，这种生命周期模型允许你改为设置和切换主视图类（`MainView`）。

:::info
如果你想在这类平台上实现导航栈（即只有一个主视图），可以使用路由控件或导航框架。常见做法是维护一个视图模型栈，在用户导航时进行入栈和出栈，再由宿主控件自动显示对应的视图。
:::

### IActivityApplicationLifetime

由以下方式提供：

* Android

在应用程序生命周期内，Android 可能会多次创建你的主 Activity 实例（例如用户点击通知，或者从其他应用返回时）。单个 `MainView` 实例无法在这些 Activity 重建之间安全复用，因此 Android 改为使用工厂函数。

将 `MainViewFactory` 属性设置为一个函数，使其在每次 Activity 启动时都创建新的视图：

```csharp
public override void OnFrameworkInitializationCompleted()
{
    if (ApplicationLifetime is IClassicDesktopStyleApplicationLifetime desktop)
        desktop.MainWindow = new MainWindow();
    else if (ApplicationLifetime is IActivityApplicationLifetime activityLifetime)
        activityLifetime.MainViewFactory = () => new MainView();
    else if (ApplicationLifetime is ISingleViewApplicationLifetime singleView)
        singleView.MainView = new MainView();
    base.OnFrameworkInitializationCompleted();
}
```

每当创建新的 Activity 实例时，这个工厂都会被调用，从而生成一个拥有独立状态的新视图。这样可以避免在多次 Activity 启动之间复用同一个视图实例所导致的崩溃问题。

## 手动管理生命周期

如果有需要，你也可以完全接管应用程序的生命周期管理。例如在桌面平台上，你可以把一个委托 `AppMain` 传给 `BuildAvaloniaApp.Start` 方法，然后从那里开始手动管理：

```csharp
class Program
{
  // This method is needed for IDE previewer infrastructure
  public static AppBuilder BuildAvaloniaApp() 
    => AppBuilder.Configure<App>().UsePlatformDetect();

  // The entry point. Things aren't ready yet, so at this point
  // you shouldn't use any Avalonia types or anything that expects
  // a SynchronizationContext to be ready
  public static int Main(string[] args) 
    => BuildAvaloniaApp().Start(AppMain, args);

  // Application entry point. Avalonia is completely initialized.
  static void AppMain(Application app, string[] args)
  {
     // A cancellation token source that will be 
     // used to stop the main loop
     var cts = new CancellationTokenSource();
     
     // Do your startup code here
     new Window().Show();

     // Start the main loop
     app.Run(cts.Token);
  }
}
```

## 另请参阅

- [Main window](/docs/fundamentals/main-window)
- [The MVVM pattern](/docs/fundamentals/the-mvvm-pattern)
