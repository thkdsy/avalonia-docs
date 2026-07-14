---
id: logs-tool
title: 日志工具
description: 在 Developer Tools 中查看和筛选 Avalonia 日志消息，集成 Microsoft.Extensions.Logging，并创建诸如 Serilog sink 之类的自定义日志源。
doc-type: reference
---

## 在工具中查看 Avalonia 日志

默认情况下，`Avalonia` 的 Warning 和 Error 日志会被 `Developer Tools` 自动记录。

主要功能包括：

1. 在数据表中显示组合后的消息。
2. 按详细级别、消息和参数进行筛选。
3. 独立显示每个参数。
4. 如果日志条目的 `Source` 是附加在元素树中的可视元素，可以点击它并在 `Developer Tools` 中导航到该元素。
5. 与第三方日志器集成。

![Logs Tool with Avalonia warnings](/img/tools/dev-tools/logs-avalonia-list.png)

## 启用 Microsoft.Extensions.Logging 集成

默认情况下，只有 `Avalonia` 日志会被重定向到 `Developer Tools` 进程。
`Diagnostics Support` 库内置了对 Microsoft logging 抽象的集成，并且可以轻松启用。

为此，需要像平常一样创建 `LoggerFactory`。返回对象可以传给 `DevToolsLoggerCollector.WithMicrosoftLogger(ILoggerFactory)` 方法。

```csharp
public override void Initialize()
{
    AvaloniaXamlLoader.Load(this);

    var loggerFactory = LoggerFactory.Create(b => b
        .SetMinimumLevel(LogLevel.Information)
        .AddConsole());

    this.AttachDeveloperTools(o =>
    {
        o.AddMicrosoftLoggerObservable(loggerFactory);
    });

    Logger = loggerFactory.CreateLogger<Application>();
}
```

对于使用 MS Dependency Injection 的方案，`ILoggerFactory` 接口可以存储在 `ServiceCollection` 中并从中获取。

有关 `DeveloperToolsOptions` 的更多细节，请参阅 [DeveloperToolsOptions 参考](/tools/developer-tools/options) 页面。

## 附加自定义日志源

![Logs Tool with custom Serilog events](/img/tools/dev-tools/logs-custom-serilog.png)

下面以创建一个 `Serilog` sink 为例，并将其配置为把日志重定向到 `Developer Tools`。

根据 `Serilog` 的 [Developing a sink](https://github.com/serilog/serilog/wiki/Developing-a-sink) 文档，需要实现一个简单的 `ILogEventSink` 接口。同时还需要实现 `ILoggerObservable`，以便将它连接到 `Developer Tools`：

```csharp
public class DevToolsSerilogSink(string logArea = "Serilog") : ILogEventSink, ILoggerObservable
{
}
```

首先实现 `ILoggerObservable.Subscribe`，并记录一组观察者列表。`ILoggerObserver` 只有两个方法：`IsEnabled` 和 `Log`，本示例都会用到。返回值是一个 disposable，它会在 DevTools 断开连接时被调用。

```csharp
private readonly LinkedList<ILoggerObserver> _observers = [];

public IDisposable Subscribe(ILoggerObserver observer)
{
    _observers.AddLast(observer);
    return Disposable.Create(() => _observers.Remove(observer));
}
```

而 `ILogEventSink.Emit` 的实现需要把 Serilog 日志事件转换为与 `ILoggerObserver` 兼容的参数：

```csharp
public void Emit(LogEvent logEvent)
{
    var logLevel = logEvent.Level switch
    {
        LogEventLevel.Verbose => LogEntryVerbosity.Verbose,
        LogEventLevel.Debug => LogEntryVerbosity.Debug,
        LogEventLevel.Information => LogEntryVerbosity.Information,
        LogEventLevel.Warning => LogEntryVerbosity.Warning,
        LogEventLevel.Error => LogEntryVerbosity.Error,
        LogEventLevel.Fatal => LogEntryVerbosity.Fatal,
        _ => throw new ArgumentOutOfRangeException()
    };

    // Map each parameter into a strings array:
    var parameters = new string[logEvent.Properties.Count];
    var paramIndex = 0;
    foreach (var value in logEvent.Properties.Values)
    {
        parameters[paramIndex++] = value.ToString(null, formatProvider);
    }

    foreach (var observer in _observers)
    {
        // `Developer Tools` might disable specific logging areas, so we need to check them first.
        if (observer.IsEnabled(logLevel, logArea))
        {
            // Queue log entry with our parameters.
            observer.Log(logLevel, logArea, null, logEvent.MessageTemplate.Text, logEvent.Exception, parameters);
        }
    }
}
```

完成这两个接口后，就可以在 `Application.Initialize` 方法中同时配置 `Serilog` 和 `Developer Tools`：

```csharp
public override void Initialize()
{
    AvaloniaXamlLoader.Load(this);

    var sink = new SerilogSink();

    Logger = new LoggerConfiguration()
        .MinimumLevel.Information()
        .WriteTo.Sink(sink)
        .CreateLogger();

    this.AttachDeveloperTools(o =>
    {
        o.AddLoggerObservable(sink);
    });
}
```

随后就可以在代码中的任意位置使用它：

```csharp
private int _clickTimes = 0;
private void Button_OnClick(object? sender, RoutedEventArgs e)
{
    _clickTimes++;
    App.Logger!.Information("Button was clicked {Times} times", _clickTimes);
}
```

<details>
  <summary>DevToolsSerilogSink 类完整代码</summary>
  
```csharp
public class DevToolsSerilogSink(string logArea = "Serilog", IFormatProvider? formatProvider = null)
    : ILogEventSink, ILoggerObservable
{
    private readonly LinkedList<ILoggerObserver> _observers = [];

    public IDisposable Subscribe(ILoggerObserver observer)
    {
        _observers.AddLast(observer);
        return Disposable.Create(() => _observers.Remove(observer));
    }

    public void Emit(LogEvent logEvent)
    {
        var logLevel = logEvent.Level switch
        {
            LogEventLevel.Verbose => LogEntryVerbosity.Verbose,
            LogEventLevel.Debug => LogEntryVerbosity.Debug,
            LogEventLevel.Information => LogEntryVerbosity.Information,
            LogEventLevel.Warning => LogEntryVerbosity.Warning,
            LogEventLevel.Error => LogEntryVerbosity.Error,
            LogEventLevel.Fatal => LogEntryVerbosity.Fatal,
            _ => throw new ArgumentOutOfRangeException()
        };

        var parameters = new string[logEvent.Properties.Count];
        var paramIndex = 0;
        foreach (var value in logEvent.Properties.Values)
        {
            parameters[paramIndex++] = value.ToString(null, formatProvider);
        }

        foreach (var observer in _observers)
        {
            if (observer.IsEnabled(logLevel, logArea))
            {
                observer.Log(logLevel, logArea, null, logEvent.MessageTemplate.Text, logEvent.Exception, parameters);
            }
        }
    }
}
```

</details>

## 另请参阅

- [Developer tools 选项](/tools/developer-tools/options)
- [Developer tools 安装](/tools/developer-tools/installation)
