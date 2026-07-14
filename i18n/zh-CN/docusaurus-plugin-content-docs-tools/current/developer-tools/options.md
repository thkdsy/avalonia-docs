---
id: options
title: Developer tools 选项
sidebar_label: 选项
doc-type: reference
---

## DeveloperToolsOptions.Gesture

定义用于启动并连接到 `Developer Tools` 进程的手势。
默认值：<kbd>F12</kbd>。

## DeveloperToolsOptions.ApplicationName

可选的应用显示名称。
如果未设置，则使用 `Application.Name` 或入口程序集名称。

## DeveloperToolsOptions.ConnectOnStartup

定义应用在启动时是否应自动连接到 dev tools。
默认值：在 iOS 和 Android 上为 `true`，其他平台均为 `false`。

## DeveloperToolsOptions.AutoConnectFromDesignMode

定义设计模式下的应用是否应连接到 dev tools。
默认值为 `false`。

## DeveloperToolsOptions.Runner

默认情况下，如果当前尚未运行 `DevTools` 实例，`DiagnosticsSupport` 包会在需要时尝试启动全局 `avdt` .NET 工具。

不过，你也可以通过修改 `DeveloperToolsOptions.Runner` 的值来重新定义这一行为：

```csharp
this.AttachDeveloperTools(o =>
{
    o.Runner = DeveloperToolsRunner.DotNetTool;
});
```

可选项包括：

1. `DeveloperToolsRunner.DotNetTool` - 使用全局 .NET 工具。
2. `DeveloperToolsOptions.AppleBundle` - 通过 macOS bundle ID 运行程序。要让它生效，你至少需要直接运行一次 `Developer Tools` 进程。
3. `DeveloperToolsOptions.NoOp` - 不执行任何操作。该选项假定 `Developer Tools` 应用已由用户手动启动。
4. `DeveloperToolsRunner.CreateFromExecutable(string)` - 通过完整路径运行可执行文件。除非你偏好自定义安装方式，否则不推荐此选项。
5. 默认值：`DeveloperToolsRunner.GetDefaultForPlatform()` - 在桌面平台上返回 `DotNetTool`，在移动端/浏览器上返回 `NoOp`。

## DeveloperToolsOptions.Protocol

`DiagnosticsSupport` 使用两种传输协议之一，在用户应用与 `Developer Tools` 进程之间进行通信：HTTP 或 Named Pipes。

```csharp
this.AttachDeveloperTools(o =>
{
    o.Protocol = DeveloperToolsProtocol.DefaultHttp;
});
```

可选项包括：

1. `DeveloperToolsProtocol.DefaultHttp` - 使用默认 HTTP 连接，端口为 `29414`，连接超时为 5 秒。
2. `DeveloperToolsProtocol.CreateHttp(Uri, TimeSpan)` - 使用给定参数创建 HTTP 连接。注意：你还需要根据 [Settings 页面](/tools/developer-tools/settings) 单独重新配置 `Developer Tools` 的监听端口。
3. `DeveloperToolsProtocol.CreateHttp(IpAddress, int? port, TimeSpan)` - 使用给定参数创建 HTTP 连接。当端口未设置时，默认使用 `29414`。
4. `DeveloperToolsProtocol.CreateNamedPipe(string)` - 创建 Named Pipe 连接。该选项仅兼容桌面平台；如果本机存在连接问题，可能更适合使用它。Named Pipe 名称会自动传递给 `Developer Tools` 实例。
5. 默认值：`DeveloperToolsProtocol.GetDefaultForPlatform()` - 当前在所有平台上都返回 `DefaultHttp`。

## DeveloperToolsOptions.DiagnosticLogger

定义所有 `AvaloniaUI.DiagnosticsSupport` 日志写入的目标 sink。
默认情况下，此选项被设置为 `AvaloniaDiagnosticLogger`，会将日志重定向到 `Avalonia.Logger.TryGet`。

可选项包括：

1. `DiagnosticLogger.CreateConsole(LogEntryVerbosity)`.
2. `DiagnosticLogger.CreateDebug(LogEntryVerbosity)`.
3. 任何用户自定义的 `DiagnosticLogger` 抽象接口实现。

:::note
如需进一步了解 `Developer Tools` 的日志记录方式，请参阅 [报告问题](/troubleshooting/tools/developer-tools)。
:::

## DeveloperToolsOptions.LoggerCollector

定义一个收集器，用于监听并收集要显示在 `Developer Tools` 中的日志。

默认情况下，`Developer Tools` 只会监听 Avalonia 日志，并将其显示在 [日志工具](/tools/developer-tools/logs-tool) 中。

你可以通过以下选项重新定义这一行为：

1. `DeveloperToolsOptions.AddAvaloniaLoggerObservable()` - 默认启用。
2. `DeveloperToolsOptions.AddMicrosoftLoggerObservable(ILoggerFactory, LogLevel)` - 允许将 devtools 作为日志提供程序连接到 Microsoft `ILoggerFactory`。
3. `DeveloperToolsOptions.AddLoggerObservable(ILoggerObservable)` - 自定义 `ILoggerObservable` 接口实现。如果你希望 DevTools 显示 Serilog 等第三方日志提供程序的日志，请使用此选项。
4. `DeveloperToolsOptions.ClearLoggerObservables()` - 清空所有 observables。

## 另请参阅

- [Developer tools 设置](/tools/developer-tools/settings)
- [Developer tools 安装](/tools/developer-tools/installation)
