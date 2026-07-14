---
id: macos
title: macOS
---

## 应用名称

默认情况下，macOS 菜单栏和系统对话框会将“Avalonia Application”显示为应用名称。要设置你自己的应用名称，需要配置 Avalonia `Application` 对象。

### 使用自定义 Avalonia 应用

按照 [自定义初始化](/xpf/configuration/customizing-initialization#optional-define-a-custom-avalonia-application) 中的步骤创建一个自定义 Avalonia Application 类，然后在你的 AXAML 中设置 `Name` 属性：

```xml title="MyAvaloniaApp.axaml"
<Application xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             x:Class="MyXpfApp.MyAvaloniaApp"
             Name="My App Name">
  <Application.Styles>
    <SimpleTheme/>
  </Application.Styles>
</Application>
```

### 使用 DefaultXpfAvaloniaApplication

如果你不需要完整自定义的 Application 类，可以继承 `DefaultXpfAvaloniaApplication` 并设置 `Name` 属性：

```csharp
public class MyAvaloniaApp : AvaloniaUI.Xpf.Helpers.DefaultXpfAvaloniaApplication
{
    public MyAvaloniaApp()
    {
        Name = "My App Name";
    }
}
```

然后在你的 `AppBuilder` 配置中引用此类：

```csharp
AppBuilder.Configure<MyAvaloniaApp>()
    .UsePlatformDetect()
    .WithAvaloniaXpf()
    .SetupWithLifetime(new ClassicDesktopStyleApplicationLifetime
    {
        ShutdownMode = ShutdownMode.OnExplicitShutdown
    });
```

## 原生菜单

macOS 应用在屏幕顶部使用全局菜单栏。XPF 通过 Avalonia 的 `NativeMenu` API 支持这一点。

### 以编程方式设置原生菜单

在你的 WPF 窗口的 `Loaded` 事件中，访问底层 Avalonia 窗口并设置菜单：

```csharp
using Atlantis;
using Avalonia.Controls;

private void Window_Loaded(object sender, RoutedEventArgs e)
{
    var avaloniaWindow = XpfWpfAbstraction.GetAvaloniaWindowForWindow(this);

    var menu = new NativeMenu();

    var fileMenu = new NativeMenuItem("File");
    var fileSubMenu = new NativeMenu();
    fileSubMenu.Add(new NativeMenuItem("Open") { Command = /* your command */ });
    fileSubMenu.Add(new NativeMenuItemSeparator());
    fileSubMenu.Add(new NativeMenuItem("Exit") { Command = /* your command */ });
    fileMenu.Menu = fileSubMenu;

    menu.Add(fileMenu);
    NativeMenu.SetMenu(avaloniaWindow, menu);
}
```

### 跨平台菜单回退

在不支持全局菜单栏的平台（Windows 和大多数 Linux 桌面环境）上，你可以通过 `AvaloniaHost` 在 XPF 窗口中嵌入一个 `NativeMenuBar` 控件。该控件仅在不支持原生全局菜单的平台上渲染传统菜单栏，并且在 macOS 上会隐藏（因为此时使用全局菜单）。

有关承载 Avalonia 控件的详细信息，请参见 [在 XPF 中嵌入 Avalonia](/xpf/interop/embedding-avalonia-in-xpf)。

## Dock 可见性

要控制你的应用是否显示在 macOS Dock 中，请在 [自定义初始化](/xpf/configuration/customizing-initialization) 中使用 `MacOSPlatformOptions`：

```csharp
AppBuilder.Configure<MyAvaloniaApp>()
    .UsePlatformDetect()
    .With(new MacOSPlatformOptions { ShowInDock = false })
    .WithAvaloniaXpf()
    .SetupWithLifetime(new ClassicDesktopStyleApplicationLifetime
    {
        ShutdownMode = ShutdownMode.OnExplicitShutdown
    });
```

### Info.plist 交互

`ShowInDock` 选项会与 macOS 的 `Info.plist` 设置产生交互：

| 配置 | 行为 |
|---|---|
| `ShowInDock = false` | 应用不显示在 Dock 中。等同于 `LSUIElement = true`。 |
| `Info.plist` 中 `LSUIElement = true` | 应用不显示在 Dock 或 Cmd+Tab 切换器中。应用没有菜单栏。 |
| `Info.plist` 中 `LSBackgroundOnly = true` | 应用作为后台进程运行，没有 UI 显示。对于带窗口的 XPF 应用不适用。 |

如果你同时在代码中设置了 `ShowInDock = false`，并在 `Info.plist` 中设置了 `LSUIElement`，请使用 XPF 1.6.0 或更高版本，以避免启动时 Dock 图标短暂闪烁。

对于仅使用托盘图标的应用，请使用 `ShowInDock = false`，并提供系统托盘图标供用户交互。

## 启动与模态对话框

在 macOS 上，在窗口激活阶段显示模态对话框（通过 `ShowDialog`）可能会导致应用冻结。这是因为首次绘制通知会在渲染管线尚未完全初始化之前触发启动代码。此时调用 `ShowDialog` 会启动一个嵌套的调度器循环，从而阻塞绘制完成。

### 推荐解决方案

在你的 `.csproj` 中添加以下内容，以将启动代码延后到安全的调度器优先级：

```xml
<ItemGroup>
    <RuntimeHostConfigurationOption Include="AvaloniaUI.Xpf.AllowBlockingCallsOnStartup" Value="true" />
</ItemGroup>
```

或者，避免在窗口构造函数或激活处理程序中调用 `ShowDialog`。改为从 `Application.Startup` 事件触发对话框，或使用调度器回调：

```csharp
Dispatcher.CurrentDispatcher.BeginInvoke(DispatcherPriority.Loaded, () =>
{
    var dialog = new MyDialog();
    dialog.ShowDialog();
});
```

:::caution
在 macOS 上，避免对启动嵌套消息循环的操作（例如 `ShowDialog`）使用 `DispatcherPriority.Normal` 或 `DispatcherPriority.Send`。应使用 `DispatcherPriority.Loaded` 或更低优先级。
:::

## DPI 和渲染缩放

WPF API `VisualTreeHelper.GetDpi()` 在 macOS 上可能不会返回准确的值。要获取正确的渲染缩放因子，请访问底层 Avalonia 窗口：

```csharp
using Atlantis;

var avaloniaTopLevel = XpfWpfAbstraction.GetAvaloniaTopLevelForWindow(myWpfWindow);
double scaling = avaloniaTopLevel.RenderScaling;
```

## 触控板手势

macOS 触控板手势（缩放、旋转）无法通过 XPF 中的 WPF 操作 API 使用。要处理这些手势，请在底层 Avalonia 窗口上使用 Avalonia 的手势事件：

```csharp
using Atlantis;
using Avalonia.Input;

var avaloniaWindow = XpfWpfAbstraction.GetAvaloniaWindowForWindow(this);

// 双指缩放
avaloniaWindow.AddHandler(Gestures.PointerTouchPadGestureMagnifyEvent, (sender, e) =>
{
    double scale = e.Scale;
    // 处理缩放
}, handledEventsToo: true);
```

要区分触控板滚动和鼠标滚轮事件，在处理 `PointerWheelChanged` 时检查 `PointerDeltaEventArgs` 属性。

## GDI+ 和 System.Drawing.Common

`System.Drawing.Common`（GDI+）在非 Windows 平台上已被弃用，并且会在 macOS 的 XPF 中抛出异常。这通常会影响依赖 GDI+ 进行渲染或打印的第三方控件（例如某些 DevExpress 控件）。

如果某个第三方控件提供了基于 Skia 的渲染选项，请在非 Windows 构建中启用它。有关非 GDI 渲染后端的指导，请联系你的控件供应商。

更多详情请参见 [库兼容性](/xpf/third-party/compatibility)。

## 打包与部署

macOS 应用必须打包为 `.app` bundle 才能分发。XPF 应用的关键注意事项：

- **不要** 将 `IncludeNativeLibrariesForSelfExtract` 设置为 `true`。这与 macOS 不兼容。
- 在开发机器之外分发时，请使用 `SelfContained` 发布。
- 进行代码签名时，请对单个文件签名，而不要使用 `--deep` 标志。
- 为获得可靠输出，请从命令行而不是 Visual Studio 发布：

```bash
dotnet publish -r osx-arm64 -c Release --self-contained
```

:::tip
Avalonia **Parcel** 工具可以自动化 XPF 应用的 macOS 打包、代码签名和公证流程。请联系 Avalonia 团队获取访问权限。
:::

## 键位映射

macOS 的修饰键与 Windows 和 Linux 不同。默认情况下，修饰键映射如下：

- Control -> `Key.LeftCtrl` / `Key.RightCtrl` / `ModifierKeys.Control`
- Option -> `Key.LeftAlt` / `Key.RightAlt` / `ModifierKeys.Alt`
- Command -> `Key.LWin` / `Key.RWin` / `ModifierKeys.Windows`

但这种映射存在一些问题：

1. macOS 应用通常在 Windows 和 Linux 上会使用 Control 键的位置使用 Command 键。例如，“复制”在 macOS 上是 Command-C，而不是 Control+C
2. `ModifierKeys.Windows` 根据 WPF 的设计不会包含在 `Keyboard.Modifiers` 中，因此无法通过标准 WPF 修饰键检查来检测 Command 键
3. 文本框等常见控件在 macOS 上预期具有不同的键盘快捷键，例如“将插入点移动到前一个单词的开头”在 macOS 上是 Option+Left Arrow，而不是 Control+Left Arrow

### 自动 macOS 键位映射

为修复这些问题，可以在启动时调用 `XpfKeyboard.MapMacOSKeys()` 方法。这通常会在与 [XPF WinAPI shim 设置](/xpf/third-party/win32-api-shims) 相同的位置完成；也就是在你的 `App` 类构造函数或 `Program.Main` 中：

```csharp
using System.Windows;
using Atlantis;

namespace XpfKeyboardMappingExample;

public partial class App : Application
{
    public App()
    {
        XpfKeyboard.MapMacOSKeys();
    }
}
```

在 macOS 上调用此方法会：

- 将 Command 键映射为 Control 键
- 将一些常见的文本框键盘快捷键映射为它们的 XPF 等效项
  - Command+Left -> Home
  - Command+Right -> End
  - Option+Left Arrow-> Ctrl+Left Arrow
  - Option+Left Arrow -> Ctrl+Left Arrow

### macOS 自定义键位映射

如需更灵活的键位映射，你可以[添加自定义键位映射](/xpf/migration/key-mapping)。

## 上下文菜单

在 macOS 上，右键单击和 Ctrl+单击都可以打开上下文菜单。你可以在启动时设置 `XpfMouse.ShowContextMenuOnMacOSCtrlClick` 来启用此功能。这通常会在与 [XPF WinAPI shim 设置](/xpf/third-party/win32-api-shims) 相同的位置完成；也就是在你的 `App` 类构造函数或 `Program.Main` 中：

```csharp
using System.Windows;
using Atlantis;

namespace XpfKeyboardMappingExample;

public partial class App : Application
{
    public App()
    {
        XpfMouse.ShowContextMenuOnMacOSCtrlClick = true;
    }
}
```

启用此功能后，可以通过处理 `ContextMenuOpening` 事件，并检查 `Keyboard.Modifiers` 和/或 `Mouse.LeftButton` 来判断上下文菜单是如何被打开的，从而在单个控件级别禁用它：

```csharp
private void OnContextMenuOpening(object sender, ContextMenuEventArgs e)
{
    // 为此特定控件抑制 Ctrl+Click 上下文菜单
    if (Keyboard.Modifiers.HasFlag(ModifierKeys.Control) && Mouse.LeftButton == MouseButtonState.Pressed)
    {
        e.Handled = true;
    }
}
```

## 原生 API 互操作

要从 XPF 应用访问 macOS 特定 API（例如 Keychain 或原生 cookie），你有以下几种选择：

- 使用 `MonoMac.NetStandard` NuGet 包访问常见的 macOS API
- 使用 C 风格的 `DllImport` 直接调用 macOS 框架
- 对于 WebView cookie 访问，使用 XPF 提供的 `NativeWebViewCookieManager` API

:::note
MAUI Essentials 不支持 macOS（仅支持 Mac Catalyst）。它不能与 macOS 上的 XPF 一起使用。
:::

## 已知限制

- **多个 UI 线程**：macOS 只允许一个 UI 线程。依赖多个调度器的 WPF 模式（例如在单独线程上的启动画面）将无法工作。请重构这些模式，改为使用主调度器和 `DispatcherPriority.Background` 来处理延迟工作。
- **透明窗口点击穿透**：XPF 不支持按像素的命中透明度（点击穿透窗口中的透明区域）。请考虑将内容嵌入单个窗口中，而不是使用透明覆盖层。
- **SystemSounds.Beep**：`System.Media.SystemSounds.Beep` 在 macOS 上不受支持，并将抛出 `PlatformNotSupportedException`。请通过平台检查保护这些调用，或在跨平台构建中移除它们。
- **工具提示抢占焦点**：在某些 macOS 版本上，显示工具提示可能会导致应用短暂地从其他应用抢占焦点。这是 XPF 团队正在跟踪的一个已知问题。