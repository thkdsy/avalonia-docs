---
id: troubleshooting
title: 故障排除
---

## 恢复 NuGet 包时遇到问题

如果你无法还原 XPF 和/或 Avalonia 包（例如报告缺少 `Xpf.Sdk` 或某个 Avalonia `cibuild` 包），请按以下步骤操作：

### 检查防火墙设置

尝试在浏览器中打开以下 URL：

:::tip
在提示时，使用用户名 `license`，并将你的许可证密钥作为密码。
:::

- https://xpf-nuget-feed.avaloniaui.net/
- https://xpf-nuget-feed.avaloniaui.net/v3/index.json
- https://nuget-feed-all.avaloniaui.net/v3/index.json

第一个 URL 应显示类似于 [nuget.org](https://www.nuget.org/packages) 上包列表的页面；后两个 URL 应显示一些 JSON。

如果你无法查看这些 URL 中的任何一个，请检查你的防火墙设置。

如果你无法使用许可证密钥登录，可能是它已过期。请向支持团队申请新的许可证密钥。

### 检查你是否已设置 NuGet.config

- 确保你已添加 [NuGet.config](/xpf/getting-started#step-2-add-a-nugetconfig) 文件，**并且它与**你正在加载的 `.sln` 文件位于同一目录中
- 确保你已在 `NuGet.config` 文件中添加了有效的许可证密钥

### 清除 NuGet HTTP 缓存

从命令行运行以下命令：

```bash
dotnet nuget locals http-cache --clear
dotnet restore
```

## Avalonia 版本冲突

如果你看到类似于以下的 `TypeLoadException`：

```text
Method 'SetDataAsync' in type 'Avalonia.Win32.ClipboardImpl' does not have an implementation
```

这通常是由显式引用的 Avalonia 包版本与 XPF 捆绑版本不匹配引起的。例如，引用 `Avalonia.Desktop` 11.3.8，而 XPF 捆绑的是 Avalonia 11.3.0。

**解决方案**：从项目中移除显式的 Avalonia 包引用。XPF SDK 会传递性地提供所有必需的 Avalonia 包。如果你需要直接引用某个 Avalonia 包，请使用 `$(XpfAvaloniaVersion)` MSBuild 属性来匹配 XPF 捆绑的版本：

```xml
<PackageReference Include="Avalonia.Headless.XUnit" Version="$(XpfAvaloniaVersion)" />
```

## 多项目解决方案中的程序集版本冲突

当混合使用 `Sdk="Xpf.Sdk"` 的项目与使用 `Sdk="Microsoft.NET.Sdk"` 且包含 `<UseWpf>true</UseWpf>` 的项目时，你可能会看到关于 `ReachFramework` 或 `System.Windows.Input.Manipulations` 版本冲突的生成警告。

这些警告可以安全忽略。运行时会使用 XPF 随附的这些程序集版本。

## ContextMenu 无法通过程序方式显示

如果设置 `ContextMenu.IsOpen = true` 后没有显示上下文菜单（而右键单击可以正常工作），请在打开之前显式设置 `PlacementTarget` 属性：

```csharp
myContextMenu.PlacementTarget = targetElement;
myContextMenu.IsOpen = true;
```

在 WPF 中，`PlacementTarget` 在某些情况下会被隐式设置，但 XPF 要求显式设置。

## 发布应用中返回空的应用程序路径

当运行单文件发布的应用程序时，`Assembly.GetEntryAssembly().Location` 会返回 null 或空值。这是 .NET 5+ 的行为，不是 XPF 特有的。

请改用 `AppDomain.CurrentDomain.BaseDirectory`：

```csharp
string appPath = AppDomain.CurrentDomain.BaseDirectory;
```

## 监听 XPF 日志

XPF 日志通过环境变量控制。
* `XPF_LOG_OUTPUT`：`console`、`trace`、`file=filePath`。支持使用 `;` 分隔多个值。
* `XPF_LOG_LEVEL`：`Verbose`、`Information`、`Debug`、`Warning`、`Error`、`Fatal`。

:::caution
较旧的文档可能会引用 `ATLANTIS_LOG_OUTPUTS` 和 `ATLANTIS_LOG_LEVEL`。正确的变量名是 `XPF_LOG_OUTPUT` 和 `XPF_LOG_LEVEL`。
:::

## 监听 Avalonia 日志

在某些情况下，收集 Avalonia 日志可能会很有用，因为 XPF 构建于 Avalonia 之上。在排查问题时，这会很有帮助。

### 在自定义 Avalonia 初始化中使用 .LogToTrace

1. 按照 [说明](/xpf/configuration/customizing-initialization) 设置自定义 Avalonia 初始化。
2. 然后你就可以在 AppBuilder 链中调用带可选严重级别参数的 `.LogToTrace()`，如下所示：
```diff
        AppBuilder.Configure<AvaloniaUI.Xpf.Helpers.DefaultXpfAvaloniaApplication>()
            .UsePlatformDetect()
+           .LogToTrace(LogEventLevel.Warning)
            .WithAvaloniaXpf()
```

这会将所有 Avalonia 日志重定向到 .NET `System.Diagnostics.Trace` 监听器。你可以在应用程序中添加自定义的 trace 监听器，将这些日志路由到文件、控制台或你自己的日志框架：

```csharp
// 将 trace 输出路由到文件
Trace.Listeners.Add(new TextWriterTraceListener("avalonia.log"));
Trace.AutoFlush = true;
```

### 覆盖 `Logger.Sink`

静态属性 `Logger.Sink` 提供公共 setter，可由自定义实现覆盖。
```csharp
public void Initialize()
{
    // 你可以在应用生命周期中的任何时刻覆盖 Logger.Sink 的值，
    // 但最好尽早进行，甚至在自定义 Avalonia 初始化中完成。
    Logger.Sink = new MyLogger();
}

public class MyLogger : ILogSink
{
    // 实现所有成员
}
```

## System.Resources.Extensions

如果你遇到以下异常：

```text
System.IO.FileNotFoundException: Could not load file or assembly 'System.Resources.Extensions, Version=4.0.0.0, Culture=neutral, PublicKeyToken=cc7b13ffcd2ddd51'. The system cannot find the file specified.
```

那么请通过 nuget 安装 `System.Resources.Extensions` 包：

```xml
<PackageReference Include="System.Resources.Extensions" Version="7.0.0" />
```

7.0.0 版本也应与 .NET 6 兼容。

## NullReferenceException 或 MissingMethodException

如果你在升级 XPF 后遇到 `NullReferenceException` 或 `MissingMethodException`，请尝试清理项目或删除 `bin`/`obj` 目录。

## Linux 上找不到 libSkiaSharp

如果你遇到：

```text
System.DllNotFoundException: Unable to load shared library 'libSkiaSharp' or one of its dependencies
```

这通常是由通过 Visual Studio 发布导致的，它可能会生成不完整的输出。请改为从命令行发布：

```bash
dotnet publish -r linux-x64 -c Release
```

更多详情请参见 [Linux: 发布](/xpf/platforms/linux#publishing-for-linux)。

## AssemblyLoadContext (ALC) 冲突

如果你的应用程序使用带有独立 `AssemblyLoadContext` 实例的自定义 .NET 主机或插件架构，XPF 初始化可能会因关于类型参数约束的 `VerificationException` 而失败。这是由同一个程序集被加载到多个 ALC 中引起的。

**解决方案**：
- 确保 XPF 程序集加载到 `AssemblyLoadContext.Default`
- 对于插件架构，请在你的 `.csproj` 中添加以下内容：
  ```xml
  <ItemGroup>
      <RuntimeHostConfigurationOption Include="AvaloniaUI.Xpf.EnableAlcSupport" Value="true" />
  </ItemGroup>
  ```
- 使用契约程序集模式在 ALC 之间进行通信

## .NET 版本兼容性

XPF 可与 .NET 6、7、8、9 和 10 一起使用。使用 XPF SDK 时，`net8.0-windows`（或类似）目标框架可在所有平台上工作。

.NET 6 之后版本新增的 WPF 功能（例如 .NET 9 中的 Fluent 主题）在 XPF 中可能不可用，但 .NET 8 中的功能（例如 `OpenFolderDialog`）是受支持的。

:::tip
当使用 XPF SDK 时，`-windows` 目标框架后缀（例如 `net8.0-windows`）可在 Linux 和 macOS 上工作。跨平台构建时你无需更改 TFM。使用不带 `-windows` 后缀的普通 `net8.0` TFM 可能会导致依赖 Windows 特定 API 的第三方库出现编译错误。
:::

## Xpf.Sdk 导入冲突

当混合使用 `Sdk="Xpf.Sdk"` 的项目与标准 `Microsoft.NET.Sdk` 项目时，你可能会遇到 MSBuild 导入冲突或重复类型警告。常见症状包括：

- `ReachFramework` 或 `System.Windows.Input.Manipulations` 版本冲突
- `WindowsDesktop` SDK 被导入两次

**解决方案**：
- 确保只有你的可执行项目使用 `Sdk="Xpf.Sdk"`。类库项目可以改用 `Microsoft.NET.Sdk` 并添加 `<EnableWindowsTargeting>true</EnableWindowsTargeting>`。
- 从使用 XPF SDK 的项目中移除显式的 `<UseWpf>true</UseWpf>`，因为该 SDK 会自动提供 WPF 支持。
- 如果在更改 SDK 后遇到 `Could not load file or assembly` 错误，请清理你的 `bin`/`obj` 目录。

## 许可证验证

XPF 使用两个标识符验证你的应用程序许可证：

1. **程序集名称**：通过 `Assembly.GetEntryAssembly().GetName().Name` 获取
2. **进程可执行文件名称**：正在运行的进程名称

两者都必须与许可证中配置的值匹配。如果许可证验证失败，请确认项目的 `AssemblyName` 与为许可证注册的名称一致。

:::note
如果你重命名了应用程序可执行文件或更改了 `.csproj` 中的 `AssemblyName`，你必须更新许可证以保持一致。请联系 Avalonia 团队更新你的许可证配置。
:::

## 创建依赖 XPF 的 NuGet 包

如果你希望将内部使用 XPF 的库作为公共 NuGet 包发布：

- 你的 NuGet 包使用者必须拥有自己的 XPF 许可证
- 不要将 XPF 程序集嵌入或重新分发到你的 NuGet 包中
- 将 XPF 包作为依赖项引用，以便它们从已授权的 NuGet 源解析
- 使用者的入口程序集名称必须与其许可证匹配，而不是你的库程序集名称

## 调度线程错误

如果你遇到“The calling thread cannot access this object because a different thread owns it”异常：

- 确保 UI 操作发生在主调度线程上：`Dispatcher.CurrentDispatcher.Invoke(() => { ... })`
- XPF 在 macOS 上仅支持单一 UI 线程。必须重构那些在单独线程上创建窗口的 WPF 模式，以使用主调度器。
- 某些第三方库（例如 Caliburn.Micro）可能会在初始化期间从后台线程访问窗口属性。请参见 [库兼容性：Caliburn.Micro](/xpf/third-party/compatibility#caliburnmicro) 获取特定于库的指导。