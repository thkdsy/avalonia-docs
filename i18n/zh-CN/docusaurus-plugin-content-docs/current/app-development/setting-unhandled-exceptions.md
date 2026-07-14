---
id: setting-unhandled-exceptions
title: 处理未捕获异常
description: 在 Avalonia 应用中处理来自 UI 线程、后台线程和任务的未捕获异常。
doc-type: how-to
---

生产环境中的应用需要一套策略来捕获那些逃逸出常规错误处理流程的异常。Avalonia 提供了多种机制，用于拦截 UI 线程以及后台任务中的未捕获异常。

## UI 线程异常

### Dispatcher.UnhandledException

当 UI 线程上的异常没有被应用代码捕获时，会触发 `Dispatcher.UIThread.UnhandledException` 事件。你可以在这里记录错误，并根据需要将其标记为已处理：

```csharp
public override void OnFrameworkInitializationCompleted()
{
    Dispatcher.UIThread.UnhandledException += OnUnhandledException;

    base.OnFrameworkInitializationCompleted();
}

private void OnUnhandledException(object sender, DispatcherUnhandledExceptionEventArgs e)
{
    // Log the exception
    Log.Error(e.Exception, "Unhandled UI thread exception");

    // Optionally prevent the application from crashing
    e.Handled = true;
}
```

:::caution
将 `e.Handled = true` 设为真，会抑制异常并让应用继续运行。请谨慎使用。如果异常已经让应用进入不一致状态（例如数据损坏、操作只完成了一部分），继续运行可能会引发更多问题。只要可能，优先在局部捕获并恢复异常。
:::

### Dispatcher.UnhandledExceptionFilter

`UnhandledExceptionFilter` 会在 `UnhandledException` 之前触发，并允许你决定某一类异常是否应该继续传递给 `UnhandledException` 处理器：

```csharp
Dispatcher.UIThread.UnhandledExceptionFilter += (sender, e) =>
{
    // Prevent certain exceptions from reaching UnhandledException
    if (e.Exception is TaskCanceledException)
    {
        e.RequestCatch = false;
    }
};
```

### 在 Main 中进行全局 try-catch

将应用入口点包裹在一个 try-catch 块中，可以捕获所有导致 UI 线程终止的异常，包括那些没有被标记为已处理的异常：

```csharp
public static void Main(string[] args)
{
    try
    {
        BuildAvaloniaApp()
            .StartWithClassicDesktopLifetime(args);
    }
    catch (Exception e)
    {
        Log.Fatal(e, "Application terminated unexpectedly");
    }
    finally
    {
        Log.CloseAndFlush();
    }
}
```

这是你的最后一道防线。当异常抵达这个代码块时，Avalonia 应用通常已经关闭。这里适合用来做日志记录和清理工作，而不是恢复应用运行。

## 后台线程异常

### 未观察到的任务异常

在 `Task.Run` 或其他异步操作中抛出的异常，如果从未被 `await` 或显式观察到，就会变成未观察到的任务异常。默认情况下，这类异常在 .NET 中通常会被静默吞掉。你可以订阅 `TaskScheduler.UnobservedTaskException` 来检测它们：

```csharp
TaskScheduler.UnobservedTaskException += (sender, e) =>
{
    Log.Error(e.Exception, "Unobserved task exception");

    // Prevent the exception from terminating the process
    e.SetObserved();
};
```

:::info
未观察到的任务异常是在终结器线程上引发的，而不是在异常实际发生时立即引发。因此事件触发可能会有延迟。为了获得更可靠的错误处理，请始终 `await` 你的任务，或使用 `ContinueWith` 显式观察异常。
:::

### AppDomain.UnhandledException

对于非 UI 线程上、且不属于任务体系的异常，可以使用 .NET 的 `AppDomain.UnhandledException` 事件：

```csharp
AppDomain.CurrentDomain.UnhandledException += (sender, e) =>
{
    var exception = e.ExceptionObject as Exception;
    Log.Fatal(exception, "Unhandled domain exception (terminating: {IsTerminating})",
        e.IsTerminating);
};
```

这个事件仅用于通知。若 `IsTerminating` 为 `true`，你无法阻止应用终止。

## 推荐策略

一套可靠的异常处理方案通常会组合多个层次：

```csharp title="Program.cs"
public static void Main(string[] args)
{
    // Background thread exceptions
    AppDomain.CurrentDomain.UnhandledException += (s, e) =>
        Log.Fatal(e.ExceptionObject as Exception, "Unhandled domain exception");

    TaskScheduler.UnobservedTaskException += (s, e) =>
    {
        Log.Error(e.Exception, "Unobserved task exception");
        e.SetObserved();
    };

    try
    {
        BuildAvaloniaApp()
            .StartWithClassicDesktopLifetime(args);
    }
    catch (Exception e)
    {
        Log.Fatal(e, "Application crashed");
    }
    finally
    {
        Log.CloseAndFlush();
    }
}
```

```csharp title="App.axaml.cs"
public override void OnFrameworkInitializationCompleted()
{
    // UI thread exceptions
    Dispatcher.UIThread.UnhandledException += (s, e) =>
    {
        Log.Error(e.Exception, "Unhandled UI exception");
        e.Handled = true; // Only if safe to continue
    };

    base.OnFrameworkInitializationCompleted();
}
```

### 日志记录

建议使用结构化日志库，例如 [Serilog](https://serilog.net) 或 [NLog](https://nlog-project.org)，把异常记录到文件、控制台或外部服务中。至少应记录异常类型、消息和堆栈跟踪，以便你根据生产环境报告进行诊断。

## 另请参阅

- [Application Lifetimes](/docs/fundamentals/application-lifetimes)：桌面端与移动端的生命周期模型。
- [TaskScheduler.UnobservedTaskException](https://learn.microsoft.com/dotnet/api/system.threading.tasks.taskscheduler.unobservedtaskexception)：.NET 文档。
- [AppDomain.UnhandledException](https://learn.microsoft.com/dotnet/api/system.appdomain.unhandledexception)：.NET 文档。
