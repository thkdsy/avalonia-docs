---
id: developer-tools
title: 开发者工具问题
sidebar_label: 开发者工具
description: 排查常见的开发者工具问题，包括连接失败、日志缺失和诊断配置。
doc-type: troubleshooting
tags:
  - avalonia plus
  - avalonia pro
  - avalonia enterprise
---

## 报告问题

Developer Tools 使用 GitHub 仓库来跟踪错误和功能请求：[AvaloniaUI/AvaloniaUI.DeveloperTools](https://github.com/AvaloniaUI/AvaloniaUI.DeveloperTools/issues)。

在提交问题之前，至少请收集以下信息：

1. 复现该问题的步骤。
2. 你的操作系统及其版本。
3. 你的应用所针对的 Avalonia 版本。
4. 你配置的任何非默认 `DeveloperToolsOptions` 值。
5. Developer Tools 和 Diagnostics Support 日志（见下文）。

## Developer Tools 无法启动或附加

如果 Developer Tools 无法打开或无法附加到你的应用，请检查以下内容：

- **确认已安装 NuGet 包。** 你的项目必须引用 `Avalonia.Diagnostics` 包（或者，如果是独立工具，请确保你已单独安装 Developer Tools）。请验证包版本与你的 Avalonia 版本匹配。
- **检查是否调用了 `AttachDeveloperTools`。** 在你的 `App.axaml.cs` 或启动代码中，确保调用了 `application.AttachDeveloperTools()`。如果没有这一步，诊断支持库将永远不会初始化。
- **验证进程未被防火墙或杀毒软件阻止。** 独立的 Developer Tools 进程通过本地连接与你的应用通信。安全软件有时会阻止这类流量。
- **检查端口冲突。** 如果其他进程正在使用同一个端口，连接可能会静默失败。请查看下面描述的 Diagnostics Support 日志以获取连接错误。
- **重启两个进程。** 如果你更新了 Avalonia 或 Developer Tools 包，请先关闭你的应用和 Developer Tools 进程，然后重新启动两者。

## 获取 Developer Tools 日志

Developer Tools 进程会收集日志、进行批处理并将其保存到磁盘。日志目录因平台而异。

**Windows:**

```text
%LocalAppData%\AvaloniaUI\com.AvaloniaUI.Net.DeveloperTools\Logs\
```

**Linux:**

```text
~/.local/share/AvaloniaUI/com.AvaloniaUI.Net.DeveloperTools/Logs/
```

**macOS:**

```text
~/Library/Application Support/AvaloniaUI/com.AvaloniaUI.Net.DeveloperTools/Logs/
```

如果日志目录不存在，Developer Tools 可能没有成功运行。请尝试手动启动它，并检查终端或控制台输出中的错误。

### 日志文件为空或缺失

- 日志是批量写入的，因此一次很短的会话可能不会产生输出。请在关闭 Developer Tools 之前至少保持打开几秒钟。
- 在 Linux 上，请确认你的用户帐户对 `~/.local/share` 目录具有写入权限。
- 在 macOS 上，请确认 `~/Library/Application Support` 目录未被系统隐私设置限制。

## 获取 Diagnostics Support 日志

Diagnostics Support 是运行在你的应用进程内并与 Developer Tools 进程建立连接的集成库。

默认情况下，它不会写入任何日志。要启用日志，请在附加时配置 `DeveloperToolsOptions`：

```csharp
application.AttachDeveloperTools(o =>
{
    // CreateConsole 返回一个内置实现，它会写入 Console.Out 和 Console.Error。
    o.DiagnosticLogger = DiagnosticLogger.CreateConsole(LogEntryVerbosity.Verbose);
});
```

启用后，诊断消息会出现在你的应用标准输出中。如果你从 IDE 运行，请查看 **Output** 或 **Debug Console** 窗口。

### 自定义日志记录器实现

如果控制台输出不太方便（例如在生产环境的诊断场景中），你可以创建一个使用 `CreateTextWriter` 工厂方法写入文件的 `DiagnosticLogger`：

```csharp
var writer = new StreamWriter("diagnostics.log", append: true);
```

然后在选项中传入它：

```csharp
application.AttachDeveloperTools(o =>
{
    o.DiagnosticLogger = DiagnosticLogger.CreateTextWriter(writer, LogEntryVerbosity.Verbose);
});
```

## 常见问题

| 症状 | 可能原因 | 解决方案 |
|---|---|---|
| Developer Tools 窗口打开了，但没有显示视觉树 | 应用尚未完成初始化 | 等待主窗口出现，或在 `OnFrameworkInitializationCompleted` 之后调用 `AttachDeveloperTools` |
| 诊断日志中显示 “Connection refused” | 端口冲突或防火墙阻止了本地流量 | 检查端口冲突和防火墙规则 |
| 日志目录存在，但没有最近的文件 | Developer Tools 在刷新日志批次之前崩溃了 | 重现问题，并在关闭前让 Developer Tools 保持更久时间打开 |
| 断点或属性编辑不生效 | `Avalonia.Diagnostics` 与你的 Avalonia 运行时版本不匹配 | 确保所有 Avalonia 包使用相同版本 |

## 另请参阅

- [Developer Tools 安装](/tools/developer-tools/installation)
- [附加应用](/tools/developer-tools/attaching-applications)
- [Developer Tools 选项](/tools/developer-tools/options)
- [元素工具](/tools/developer-tools/elements-tool)