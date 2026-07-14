---
id: windows
title: Windows
description: Avalonia 在 Windows 上的平台特性，包括透明效果、Mica、自定义标题栏、深色模式、DPI 缩放与 Win32 互操作。
doc-type: overview
---

## Avalonia 如何在 Windows 上运行

Avalonia 在 Windows 上直接使用 Win32 API。除了 .NET SDK 之外，不需要额外的 workload 或依赖。你的 target framework 只需使用 `net9.0` 或 `net10.0`，而不是某个特定平台版本。

渲染使用基于 Direct3D 的 Skia；当 GPU 加速不可用时（例如远程桌面会话中，或未开启 GPU 直通的虚拟机中），会自动退回到软件渲染。输入、窗口系统、剪贴板、文件对话框、拖放和无障碍能力也都通过标准 Win32 API 提供。

由于不依赖 Windows 专属的 .NET workload，你可以在 macOS 或 Linux 上交叉编译 Windows 版本。生成的二进制文件可在任意受支持且已安装 .NET 运行时的 Windows 版本上运行。

## 窗口透明效果与 Mica

Windows 支持全部 `TransparencyLevelHint` 取值，因此它也是唯一一个完整支持透明效果的平台。macOS 仅支持 `Transparent`，而 Linux 是否支持则取决于合成器。

| 级别 | 效果 | 最低版本 |
|---|---|---|
| `Transparent` | 完全透明的窗口背景 | Windows 7+ |
| `AcrylicBlur` | 模糊、半透明背景 | Windows 10 1803+ |
| `Mica` | 由 DWM 绘制的系统着色材质 | Windows 11 |

要启用 Mica 背景：

```xml
<Window xmlns="https://github.com/avaloniaui"
        TransparencyLevelHint="Mica"
        Background="Transparent">
    <!-- 你的内容 -->
</Window>
```

如果 Mica 不可用（例如在 Windows 10 上），窗口会按照列表顺序依次回退。你可以在运行时检查当前实际启用的是哪一级：

```csharp
var actual = myWindow.ActualTransparencyLevel;
```

要让任何透明级别生效，都需要把窗口的 `Background` 设为 `Transparent`。不透明背景会完全遮盖透明效果。

更多细节请参阅 [窗口管理](/docs/app-development/window-management)。

## 自定义标题栏

Windows 支持把客户端区域扩展到标题栏区域，从而实现自定义窗口外观。可以通过设置 `ExtendClientAreaToDecorationsHint` 把内容推入标题栏区域，并使用 `WindowDecorationProperties.ElementRole` 将某个区域标记为可拖动标题栏：

```xml
<Window ExtendClientAreaToDecorationsHint="True"
        WindowDecorations="None">
    <Grid RowDefinitions="32,*">
        <Border Grid.Row="0" Background="#2D2D2D"
                WindowDecorationProperties.ElementRole="TitleBar">
            <TextBlock Text="My App" Foreground="White"
                       VerticalAlignment="Center" Margin="12,0" />
        </Border>
        <Border Grid.Row="1">
            <TextBlock Text="Window content" />
        </Border>
    </Grid>
</Window>
```

被标记的区域支持原生窗口拖动和双击最大化。放在标题栏区域中的交互控件仍然可以正常接收输入。

将 `WindowDecorations="None"` 设为无，会移除所有系统窗口边框，让你完全控制窗口外观；而使用 `WindowDecorations="Full"` 时，系统的最小化、最大化和关闭按钮仍会和你的自定义标题栏内容一起显示。在 Windows 上，被禁用的标题栏按钮会直接隐藏，而不是显示为灰色。

更多细节请参阅 [窗口管理](/docs/app-development/window-management#custom-title-bar)。

## 深色模式与系统主题检测

Avalonia 通过 `PlatformSettings` 检测 Windows 系统主题和强调色。内置的 `FluentTheme` 会自动在浅色和深色变体之间切换，以匹配系统偏好。

若要读取当前主题并响应变化：

```csharp
var settings = myControl.GetPlatformSettings();
var colors = settings?.GetColorValues();

// 检查当前主题
if (colors?.ThemeVariant == ThemeVariant.Dark)
{
    // 系统当前处于深色模式
}

// 实时响应主题变化
if (settings is not null)
{
    settings.ColorValuesChanged += (sender, values) =>
    {
        // values 中包含更新后的主题和强调色
    };
}
```

这对于 `FluentTheme` 自动处理范围之外的自定义主题逻辑非常有用。例如，你可以根据系统当前是浅色还是深色模式，调整图表颜色或图像叠加层。

:::note
在 Windows 11 上，Avalonia 会自动更新原生标题栏，使其与应用的 `RequestedThemeVariant` 保持一致。而在 Windows 10 上，标题栏不会变暗，因为平台没有为此提供官方 API。如果你在 Windows 10 上需要深色标题栏，可以使用[自定义标题栏](#custom-title-bars)，或使用 [Windows 故障排查](/troubleshooting/platform-specific-issues/windows#title-bar-stays-light-when-switching-to-dark-theme-on-windows-10) 中提到的未文档化 `DwmSetWindowAttribute` 变通方案。
:::

更多细节请参阅 [平台设置](/docs/services/platform-settings)。

## 高 DPI 与按显示器缩放

在 Windows 上，Avalonia 默认具备按显示器感知 DPI 的能力。每个显示器都可以报告自己的缩放因子，而 Avalonia 会在窗口跨显示器移动时自动调整布局和渲染。

`Screen.Scaling` 属性会报告每个显示器的 DPI 缩放因子：

| 缩放值 | DPI | 常见用途 |
|---|---|---|
| `1.0` | 96 | 标准密度（100%） |
| `1.25` | 120 | 125% 缩放 |
| `1.5` | 144 | 150% 缩放 |
| `2.0` | 192 | HiDPI / 200% 缩放 |

Avalonia 中的所有布局都使用设备无关像素，因此你不需要手动进行 DPI 换算。若要读取当前屏幕的缩放因子：

```csharp
var screen = myWindow.Screens.ScreenFromWindow(myWindow);
var scaling = screen?.Scaling ?? 1.0;
```

对于位图资源，当显示缩放为 2.0 或更高时，Avalonia 会自动选择 `@2x` 变体。

更多细节请参阅 [窗口管理](/docs/app-development/window-management#working-with-screens)。

## 嵌入原生 Win32 控件

你可以使用 `NativeControlHost` 将基于 HWND 的 Win32 控件嵌入到 Avalonia 布局中。重写 `CreateNativeControlCore` 以创建并返回原生控件句柄：

```csharp
public class NativeTextEditor : NativeControlHost
{
    protected override IPlatformHandle CreateNativeControlCore(
        IPlatformHandle parent)
    {
        if (OperatingSystem.IsWindows())
        {
            var hwnd = CreateWindowEx(0, "EDIT", "",
                WS_CHILD | WS_VISIBLE | ES_MULTILINE,
                0, 0, 100, 100,
                parent.Handle, IntPtr.Zero, IntPtr.Zero, IntPtr.Zero);
            return new PlatformHandle(hwnd, "HWND");
        }

        return base.CreateNativeControlCore(parent);
    }

    protected override void DestroyNativeControlCore(
        IPlatformHandle control)
    {
        if (OperatingSystem.IsWindows())
            DestroyWindow(control.Handle);
        else
            base.DestroyNativeControlCore(control);
    }
}
```

你还可以获取任意 Avalonia 窗口的 HWND，以便与现有 Win32 代码互操作：

```csharp
var handle = TopLevel.GetTopLevel(myControl)?.TryGetPlatformHandle();
// handle.Handle 中包含 HWND（类型为 IntPtr）
// handle.HandleDescriptor 的值为 "HWND"
```

通过 `NativeControlHost` 渲染的原生控件总是位于 Avalonia 内容的上层或下层，而不会参与常规视觉树的 z 轴排序。它们不支持透明或渲染变换（旋转、缩放），裁剪能力也仅限于宿主控件的边界。

更多细节请参阅 [原生平台互操作](/docs/app-development/native-interop)。

## 在 Windows Forms 中嵌入 Avalonia

你可以使用 `WinFormsAvaloniaControlHost` 将 Avalonia 控件托管到 Windows Forms 应用中。这使得现有 Windows Forms 应用能够渐进式迁移到 Avalonia，而不必一次性全部重写。

一个典型的配置至少需要两个项目：

1. **YourApp**：一个跨平台类库，包含 Avalonia 控件和 ViewModel。
2. **YourApp.WinForms**：你现有的 Windows Forms 应用。
3. **YourApp.Desktop**（可选）：一个独立的 Avalonia 可执行项目，仅当你想使用 Visual Studio 的 XAML 预览器时才需要。

由于 Windows Forms 只能运行在 Windows 上，因此把 Avalonia 控件嵌入 WinForms 应用并不会让它变成跨平台应用。如果你需要真正的跨平台支持，仍应完整迁移到 Avalonia 桌面项目。

### Setup

以下说明默认你使用的是安装了 Avalonia 扩展的 Visual Studio 2022。如果你使用 VS Code 或 Rider，可以跳过可选的 `YourApp.Desktop` 项目。

1. 使用 **Avalonia C# Project** 模板向解决方案添加一个新项目，并至少选择 **Desktop** 作为目标平台。这样会创建 `YourApp` 和 `YourApp.Desktop`。

2. 向 Windows Forms 项目添加以下引用：
   - 包引用：`Avalonia.Desktop`
   - 包引用：`Avalonia.Win32.Interoperability`
   - 项目引用：`YourApp.csproj`

3. 在 WinForms 的 `Program.cs` 中，在调用 `Application.Run()` 之前初始化 Avalonia：

```csharp
AppBuilder.Configure<App>()
    .UsePlatformDetect()
    .SetupWithoutStarting();
```

4. 向窗体中添加 `WinFormsAvaloniaControlHost` 控件（添加包引用后可在 Toolbox 中看到）。

5. 在窗体构造函数中、`InitializeComponent()` 之后，为它设置内容：

```csharp
winFormsAvaloniaControlHost1.Content = new MainView
{
    DataContext = new MainViewModel()
};
```

现在你应该就能在 Windows Forms 应用中看到 Avalonia 的默认视图已经被渲染出来。

## 系统托盘集成

Windows 对 `TrayIcon` 提供完整支持。托盘图标会显示在 Windows 通知区域，并在右键时提供 `NativeMenu` 上下文菜单。

```xml
<TrayIcon Icon="/Assets/app-icon.ico"
          ToolTipText="My Application">
    <TrayIcon.Menu>
        <NativeMenu>
            <NativeMenuItem Header="Show Window" Click="ShowWindow_OnClick" />
            <NativeMenuItemSeparator />
            <NativeMenuItem Header="Exit" Click="Exit_OnClick" />
        </NativeMenu>
    </TrayIcon.Menu>
</TrayIcon>
```

Windows 要求托盘图标使用 `.ico` 格式。请在 `.csproj` 中将图标文件作为 Avalonia 资源引入：

```xml
<ItemGroup>
    <AvaloniaResource Include="Assets/app-icon.ico" />
</ItemGroup>
```

如果你希望应用最小化到托盘而不是任务栏，请在最小化时将窗口的 `ShowInTaskbar` 设为 `False`，并在托盘图标的点击处理程序中恢复窗口。

更多细节请参阅 [TrayIcon](/controls/navigation/trayicon)。

## Accessibility

Avalonia 通过 UI Automation（UIA）框架，在 Windows 上将 UI 元素暴露给辅助技术。Narrator 和 NVDA 等屏幕阅读器都可以读取并与 Avalonia 应用交互。

内置控件会自动提供 automation peer，因此按钮、文本框、列表项等标准控件开箱即支持屏幕阅读器。对于自定义控件，请设置 `AutomationProperties.Name` 以提供有意义的标签：

```xml
<MyCustomControl AutomationProperties.Name="Star rating" />
```

对于更高级的场景，可以在控件中重写 `OnCreateAutomationPeer`，返回一个自定义 automation peer，用于上报正确的控件类型和状态。

若要支持 Narrator 的地标导航，请将 `AutomationProperties.AccessibilityView` 至少设置为 `"Control"`，这样 Narrator 的相关快捷方式才会生效。

更多细节请参阅 [无障碍](/docs/app-development/accessibility)。

## Windows 上的平台特定服务

Avalonia 的平台服务在 Windows 上依赖 Win32 API。以下是常见服务行为的摘要：

| 服务 | Windows 上的行为 |
|---|---|
| Clipboard | 通过 Win32 剪贴板 API 支持文本、HTML、RTF 和文件列表。 |
| File dialogs | 使用 Win32 通用文件对话框（IFileDialog），支持文件类型筛选、初始目录和多选。 |
| Drag and drop | 支持从资源管理器拖放文件，以及通过 OLE 在应用之间传输数据。 |
| Launcher | `Launcher.LaunchUriAsync` 会在默认浏览器中打开 URL，`Launcher.LaunchFileAsync` 会使用关联应用打开文件。 |

## 另请参阅

- [Windows 打包](/tools/parcel/packaging-for-windows)（NSIS 安装包、代码签名）
- [WPF 迁移指南](/docs/migration/wpf)
- [原生平台互操作](/docs/app-development/native-interop)
- [窗口管理](/docs/app-development/window-management)
- [平台设置](/docs/services/platform-settings)
- [键盘与快捷键](/docs/input-interaction/keyboard-and-hotkeys)
- [无障碍](/docs/app-development/accessibility)
