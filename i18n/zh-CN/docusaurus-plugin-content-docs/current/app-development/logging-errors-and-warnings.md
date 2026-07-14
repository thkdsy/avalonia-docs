---
id: logging-errors-and-warnings
title: 记录错误和警告日志
description: 使用 LogToTrace 方法和日志区域启用并配置 Avalonia 诊断日志。
doc-type: how-to
---

import LogToTraceOutputScreenshot from '/img/guides/app-development/log-to-trace-output.png';

本指南演示如何在 Avalonia 中使用标准 `System.Diagnostics.Trace` 组件记录警告和错误日志。

## 启用日志

如果你使用的是 Avalonia 解决方案模板，那么用于启用日志的代码通常已经自动添加到项目中了。

如果你要手动启用，或者只是检查日志是否已启用，请按以下步骤操作：

- 找到应用中的 **Program.cs** 文件。
- 检查 `BuildAvaloniaApp` 方法中是否调用了 `LogToTrace`，例如：

```csharp
public static AppBuilder BuildAvaloniaApp()
    => AppBuilder.Configure<App>()
        .UsePlatformDetect()
        .LogToTrace();
```

不带参数时，`LogToTrace` 会记录严重级别为 `Warning` 或更高的消息。你可以通过向 `LogToTrace` 传入 `LogLevel` 参数来改用其他级别。例如：

```csharp
using Avalonia.Logging;
...
public static AppBuilder BuildAvaloniaApp()
    => AppBuilder.Configure<App>()
        .UsePlatformDetect()
        .LogToTrace(LogEventLevel.Verbose)
```

:::info
完整 API 文档请参阅 [`LogEventLevel` enum reference](/api/avalonia/logging/logeventlevel)。
:::

之后，日志消息会显示在 IDE 的 **Output** 窗口中的 **Debug** 视图里。例如，在启用 verbose 日志后：

<Image light={LogToTraceOutputScreenshot} alt="Verbose log output in the IDE Debug Output window" position="center" maxWidth={400} cornerRadius="true"/>

如果你想把这些消息重新路由到其他位置，可以使用 `System.Diagnostics.Trace` 组件上的相关方法。

## 日志区域

Avalonia 发出的每条日志消息都会被归类到一个区域，你可以利用这个区域来过滤日志。这些区域由 `Avalonia.Logging.LogArea` 静态类中的成员表示：

* `Property`
* `Binding`
* `Animations`
* `Visual`
* `Layout`
* `Control`

你可以在 `LogToTrace` 调用中，在 `LogEventLevel` 参数后面继续传入 `Avalonia.Logging.LogArea` 类型参数，从而将日志限制在某个或某几个特定区域。例如，下面的代码只记录 property 和 layout 相关消息：

```csharp
public static AppBuilder BuildAvaloniaApp()
    => AppBuilder.Configure<App>()
        .UsePlatformDetect()
        .LogToTrace(LogEventLevel.Debug, LogArea.Property, LogArea.Layout);
```

## 其他日志目标

### LogToDelegate

将日志消息路由到自定义回调函数：

```csharp
public static AppBuilder BuildAvaloniaApp()
    => AppBuilder.Configure<App>()
        .UsePlatformDetect()
        .LogToDelegate((level, area, source, messageTemplate, propertyValues) =>
        {
            Console.WriteLine($"[{area}] {messageTemplate}");
        });
```

### LogToTextWriter

将日志消息写入任意 `TextWriter`，例如文件或 `Console.Out`：

```csharp
public static AppBuilder BuildAvaloniaApp()
    => AppBuilder.Configure<App>()
        .UsePlatformDetect()
        .LogToTextWriter(File.CreateText("avalonia.log"));
```

## 日志 sink

`LogToTrace` 扩展方法使用的是 `StringLogSink`。Avalonia 支持通过实现 `ILogSink` 来创建自定义 sink。只要把你的自定义 sink 赋值给 `Avalonia.Logging.Logger.Sink`，Avalonia 就会使用它。

```csharp title='Extension method to assign Logger.Sink'
using Avalonia.Controls;
using Avalonia.Logging;

namespace MyNamespace;
public static class MyLogExtensions
{
    public static AppBuilder LogToMySink(this AppBuilder builder, 
        LogEventLevel level = LogEventLevel.Warning, 
        params string[] areas)
    {
        Logger.Sink = new MyLogSink(level, areas);
        return builder;
    }
}
```

```csharp title='Startup with custom sink'
public static AppBuilder BuildAvaloniaApp()
    => AppBuilder.Configure<App>()
        .UsePlatformDetect()
        .LogToMySink();
```

:::info
可以在 _GitHub_ 上查看源代码：[`StringLogSink.cs`](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Base/Logging/StringLogSink.cs)
:::

## 另请参阅

- [Setting Unhandled Exceptions](/docs/app-development/setting-unhandled-exceptions)：处理应用中的未捕获异常。
- [LogEventLevel API reference](/api/avalonia/logging/logeventlevel)：可用的日志严重级别。
