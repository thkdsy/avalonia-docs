---
id: customizing-initialization
title: 自定义初始化
description: 在通过禁用自动初始化的 XPF 应用程序中如何访问并自定义 Avalonia AppBuilder API。
---

Avalonia 提供了 [`AppBuilder`](/docs/fundamentals/application-lifetimes) API，用于自定义框架的各个方面。 

由于 XPF 基于 Avalonia，因此在 XPF 应用程序中访问此 API 会很有用。

## 第 1 步：禁用自动 XPF 初始化

在你的项目文件中将 `DisableAutomaticXpfInit` 属性设置为 `true`：

```xml
<PropertyGroup>
  <DisableAutomaticXpfInit>true</DisableAutomaticXpfInit>
</PropertyGroup>
```

## 第 2 步：添加主入口点

添加一个包含 `Main` 入口点的 `Program.cs` 文件：

```csharp
using Avalonia;
using Avalonia.Controls;
using Avalonia.Controls.ApplicationLifetimes;
using AvaloniaUI.Xpf;

namespace MyXpfApp;

internal class Program
{
    public static void Main(string[] args)
    {
        AppBuilder.Configure<AvaloniaUI.Xpf.Helpers.DefaultXpfAvaloniaApplication>()
            .UsePlatformDetect()
            .WithAvaloniaXpf()
            .SetupWithLifetime(new ClassicDesktopStyleApplicationLifetime
            { 
                ShutdownMode = ShutdownMode.OnExplicitShutdown 
            });

        App.Main();
    }
}
```

:::tip
将上面示例中的命名空间更改为与你的应用程序命名空间一致。
:::

在上面的示例中，`App` 是你在 `App.xaml.cs` 中定义的 XPF `Application` 类。

## 第 3 步：设置 `StartupObject`

通过在你的 `.csproj` 中添加以下内容，配置项目使用新的 `Main` 方法：

```xml
 <PropertyGroup>
     <StartupObject>MyXpfApp.Program</StartupObject>
 </PropertyGroup>
```

:::tip
将上面示例中的命名空间更改为 `Program.cs` 中定义的命名空间。
:::

## AssemblyLoadContext (ALC) 支持

如果你的应用程序使用自定义 .NET 宿主或带有独立 `AssemblyLoadContext` 实例的插件架构，请通过在你的 `.csproj` 中添加以下内容来启用 ALC 支持：

```xml
<ItemGroup>
    <RuntimeHostConfigurationOption Include="AvaloniaUI.Xpf.EnableAlcSupport" Value="true" />
</ItemGroup>
```

这可以防止因同一个程序集被加载到多个 ALC 中而导致的 `VerificationException` 错误。你在以下情况下需要此设置：

- 你的应用程序使用会将程序集加载到隔离 ALC 中的插件系统
- 你在另一个使用自定义程序集加载的应用程序框架中托管 XPF
- 在 XPF 初始化期间，你看到有关类型参数约束的错误

## 自定义程序集加载

如果你有自定义机制来加载托管程序集，你可能会发现使用 `AvaloniaUI.Xpf.WinApiShim.WinApiShimSetup.AutoEnable` 函数会导致应用程序卡死。如果发生这种情况，可以尝试使用延迟添加程序集的方式，即使用 `AvaloniaUI.Xpf.WinApiShim.WinApiShimSetup.AddLibrary`，如下所示：

```csharp
using Avalonia;
using Avalonia.Controls;
using Avalonia.Controls.ApplicationLifetimes;
using AvaloniaUI.Xpf;
using AvaloniaUI.Xpf.WinApiShim;

namespace MyXpfApp;

internal class Program
{
    public void CalledFromYourCustomAssemblyLoading(Assembly targetAssembly)
    {
        // 此调用会将该程序集添加到将通过 XPF 的 Win32 Shim 系统
        // 拦截的程序集列表中。
        WinApiShimSetup.AddLibrary(targetAssembly);
    }
}
```

## 调度器约束

XPF 在所有平台上都支持单一 UI 调度器。在 macOS 上，这是由操作系统强制执行的（只允许一个 UI 线程）。在 Windows 和 Linux 上，有限的多调度器支持是存在的，但不建议使用。

如果你的 WPF 应用程序会在单独的线程上创建窗口（例如启动画面或进度对话框），请将这些模式重构为使用主调度器：

```csharp
// 不要为启动画面创建新线程：
Dispatcher.CurrentDispatcher.BeginInvoke(DispatcherPriority.Background, () =>
{
    var splash = new SplashWindow();
    splash.Show();
});
```

有关更多详细信息，请参阅[缺失的功能：多个 UI 线程](/xpf/version-info/missing-features)。

## 可选：定义自定义 Avalonia 应用程序

在某些情况下，你可能希望使用自定义的 Avalonia `Application` 类；这种场景的一些用例包括：

- 提供应用程序范围的 Avalonia 样式和资源
- 提供应用程序 `NativeMenu`

为此，首先为你的应用程序类添加 `.cs` 和 `.axaml` 文件：

```csharp title="MyAvaloniaApp.axaml.cs"
using Avalonia;
using Avalonia.Markup.Xaml;
using Avalonia.Styling;

namespace MyXpfApp;

public class MyAvaloniaApp : Application
{
    public MyAvaloniaApp()
    {
        RequestedThemeVariant = ThemeVariant.Light;
        AvaloniaXamlLoader.Load(this);
    }
}
```

```xml title="MyAvaloniaApp.axaml"
<Application xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             x:Class="MyXpfApp.MyAvaloniaApp">
  <Application.Styles>
    <SimpleTheme/>
  </Application.Styles>
</Application>
```

然后在第 2 步中添加的 `AppBuilder` 配置里引用这个自定义 `Application`：

```csharp
// highlight-next-line
AppBuilder.Configure<MyAvaloniaApp>()
    .UsePlatformDetect()
    .WithAvaloniaXpf()
    .SetupWithLifetime(new ClassicDesktopStyleApplicationLifetime
    { 
        ShutdownMode = ShutdownMode.OnExplicitShutdown 
    });
```