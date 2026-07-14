---
id: window-management
title: 窗口管理
description: 在 Avalonia 桌面应用中创建、配置和管理窗口与对话框。
doc-type: overview
---

Avalonia 提供了一个灵活的窗口系统，可用于创建单窗口和多窗口桌面应用。本页介绍常见的窗口管理模式。

## 创建窗口

窗口通常在 XAML 中定义，并配合一个代码后置类：

```xml title="SecondWindow.axaml"
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        x:Class="MyApp.SecondWindow"
        Title="Second Window"
        Width="400" Height="300">
    <TextBlock Text="Hello from the second window" />
</Window>
```

```csharp title="SecondWindow.axaml.cs"
public partial class SecondWindow : Window
{
    public SecondWindow()
    {
        InitializeComponent();
    }
}
```

### 打开一个窗口

```csharp
var window = new SecondWindow();
window.Show(); // Non-modal: both windows remain interactive
```

### 打开模态对话框

```csharp
var dialog = new SecondWindow();
var result = await dialog.ShowDialog<string>(parentWindow);
// Execution resumes here after the dialog closes
```

当对话框打开时，父窗口会被禁用。请将拥有者窗口作为参数传给 `ShowDialog`。

### 带结果关闭

在对话框中，你可以通过向 `Close` 传入一个值来设置返回结果：

```csharp
// Inside the dialog
Close("user clicked OK");
```

这个值会作为父窗口中 `ShowDialog<T>` 调用的返回结果。

## 窗口属性

| 属性 | 说明 |
|---|---|
| `Title` | 显示在窗口标题栏中的文本。 |
| `Width`, `Height` | 初始尺寸。 |
| `MinWidth`, `MinHeight` | 允许的最小尺寸。 |
| `MaxWidth`, `MaxHeight` | 允许的最大尺寸。 |
| `WindowStartupLocation` | 窗口出现的位置：`Manual`、`CenterScreen`、`CenterOwner`。 |
| `Position` | 窗口在屏幕坐标中的位置（当 `WindowStartupLocation` 为 `Manual` 时）。 |
| `CanResize` | 用户是否可以调整窗口大小。 |
| `CanMinimize` | 最小化按钮是否启用。默认值为 `true`。 |
| `CanMaximize` | 最大化按钮是否启用。默认值为 `true`。当 `CanResize` 为 `false` 时会自动变为 `false`。 |
| `IsDialog` | 只读。使用 `ShowDialog` 打开时为 `true`，使用 `Show` 打开时为 `false`。 |
| `ShowInTaskbar` | 窗口是否出现在操作系统任务栏中。 |
| `Topmost` | 窗口是否始终保持在其他窗口之上。 |
| `WindowState` | 当前窗口状态：`Normal`、`Minimized`、`Maximized`、`FullScreen`。 |
| `SystemDecorations` | 标题栏和边框样式：`Full`、`BorderOnly`、`None`。 |
| `ExtendClientAreaToDecorationsHint` | 将客户区扩展到标题栏区域，以实现自定义窗口外观。应用绘制装饰时会配合 `WindowDrawnDecorations` 使用。 |
| `Icon` | 显示在标题栏和任务栏中的窗口图标。 |
| `TransparencyLevelHint` | 启用窗口透明效果：`None`、`Transparent`、`AcrylicBlur`、`Mica`。覆盖层模式可参考 [Transparent click-through window](/docs/how-to/window-how-to#transparent-click-through-window)。 |
| `ClosingBehavior` | 控制拥有者窗口关闭时子窗口的行为：`OwnerAndChildWindows`（默认，先关闭子窗口，且可取消）或 `OwnerWindowOnly`（只检查拥有者窗口的 `Closing` 事件）。 |

## 窗口尺寸控制

### 按内容自动定尺寸

设置 `SizeToContent` 后，窗口会根据内容自动调整尺寸：

```xml
<Window SizeToContent="WidthAndHeight">
    <StackPanel Margin="20">
        <TextBlock Text="The window will size to fit this content." />
        <Button Content="OK" HorizontalAlignment="Right" Margin="0,10,0,0" />
    </StackPanel>
</Window>
```

| 值 | 行为 |
|---|---|
| `Manual` | 窗口显式使用 `Width` 和 `Height`（默认值）。 |
| `Width` | 宽度根据内容自动调整，高度显式指定。 |
| `Height` | 高度根据内容自动调整，宽度显式指定。 |
| `WidthAndHeight` | 宽高都根据内容自动调整。 |

### 保存与恢复窗口位置

```csharp
protected override void OnOpened(EventArgs e)
{
    base.OnOpened(e);

    // Restore saved position
    if (Settings.WindowLeft >= 0 && Settings.WindowTop >= 0)
    {
        Position = new PixelPoint(Settings.WindowLeft, Settings.WindowTop);
        Width = Settings.WindowWidth;
        Height = Settings.WindowHeight;
    }
}

protected override void OnClosing(WindowClosingEventArgs e)
{
    base.OnClosing(e);

    // Save position
    Settings.WindowLeft = Position.X;
    Settings.WindowTop = Position.Y;
    Settings.WindowWidth = Width;
    Settings.WindowHeight = Height;
}
```

## 多窗口模式

### 跟踪已打开窗口

```csharp
public static class WindowManager
{
    private static readonly List<Window> _openWindows = new();

    public static IReadOnlyList<Window> OpenWindows => _openWindows;

    public static void Register(Window window)
    {
        _openWindows.Add(window);
        window.Closed += (_, _) => _openWindows.Remove(window);
    }

    public static void CloseAll()
    {
        foreach (var window in _openWindows.ToList())
            window.Close();
    }
}
```

### 从控件查找父窗口

```csharp
var topLevel = TopLevel.GetTopLevel(myControl);
if (topLevel is Window window)
{
    // Use window
}
```

或者使用扩展方法：

```csharp
var window = myControl.FindAncestorOfType<Window>();
```

## 阻止窗口关闭

你可以处理 `Closing` 事件来拦截关闭操作。将 `e.Cancel = true` 设为真即可阻止窗口关闭：

```csharp
protected override void OnClosing(WindowClosingEventArgs e)
{
    base.OnClosing(e);

    if (HasUnsavedChanges)
    {
        e.Cancel = true;
        // Show a save confirmation dialog instead
        _ = ShowSavePromptAsync();
    }
}
```

## 自定义标题栏

如果要创建自定义标题栏，可以先把客户区扩展到窗口装饰区域，再使用 `WindowDecorationProperties.ElementRole` 将某个区域标记为标题栏：

```xml
<Window ExtendClientAreaToDecorationsHint="True"
        SystemDecorations="None">
    <Grid RowDefinitions="32,*">
        <!-- Custom title bar -->
        <Border Grid.Row="0" Background="#2D2D2D"
                WindowDecorationProperties.ElementRole="TitleBar">
            <TextBlock Text="My App" Foreground="White"
                       VerticalAlignment="Center" Margin="12,0" />
        </Border>
        <!-- Content -->
        <Border Grid.Row="1">
            <TextBlock Text="Window content" />
        </Border>
    </Grid>
</Window>
```

被标记为 `WindowDecorationProperties.ElementRole="TitleBar"` 的元素支持原生窗口拖动，以及双击最大化。放在标题栏区域内的交互控件（如按钮、文本框）仍然会正常接收输入，而不会触发拖动行为。

有关 `ElementRole` 各取值的更多说明，请参阅 [custom title bar how-to](/docs/how-to/window-how-to#custom-title-bar-with-drag-region)。

## 窗口事件

| 事件 | 触发时机 |
|---|---|
| `Opened` | 窗口第一次显示后触发。 |
| `Closing` | 窗口即将关闭时触发，可取消。 |
| `Closed` | 窗口已经关闭后触发。 |
| `Activated` | 窗口获得焦点时触发。 |
| `Deactivated` | 窗口失去焦点时触发。 |
| `PositionChanged` | 窗口被移动时触发。 |
| `Resized` | 窗口尺寸变化时触发。 |

## 使用屏幕信息

`Screens` API 提供有关已连接显示器的信息。你可以从任意 `TopLevel` 访问它：

```csharp
var screens = TopLevel.GetTopLevel(this)?.Screens;
```

### 查询屏幕

```csharp
// All connected screens
var allScreens = screens.All;

// Primary monitor
var primary = screens.Primary;

// Screen containing a specific window
var currentScreen = screens.ScreenFromWindow(this);

// Screen at a point
var screenAtPoint = screens.ScreenFromPoint(new PixelPoint(500, 300));
```

### 屏幕属性

每个 `Screen` 对象都提供以下属性：

| 属性 | 类型 | 说明 |
|---|---|---|
| `Bounds` | `PixelRect` | 屏幕完整边界，单位为像素。 |
| `WorkingArea` | `PixelRect` | 可用区域，不包含任务栏和停靠栏。 |
| `Scaling` | `double` | DPI 缩放系数（例如 96 DPI 为 1.0，144 DPI 为 1.5）。 |
| `IsPrimary` | `bool` | 是否为主显示器。 |
| `DisplayName` | `string?` | 操作系统报告的显示器名称。 |
| `CurrentOrientation` | `ScreenOrientation` | 当前屏幕方向（如横向、纵向等）。 |

### 响应屏幕变化

订阅 `Changed` 事件，可以检测显示器何时被添加、移除或重新配置：

```csharp
screens.Changed += (sender, args) =>
{
    // Re-evaluate window placement or layout
    var count = screens.All.Count;
};
```

## 平台差异

| 特性 | Windows | macOS | Linux |
|---|---|---|---|
| `Topmost` | 支持 | 支持 | 支持 |
| `TransparencyLevelHint` | 支持全部级别 | 仅支持 `Transparent` | 取决于合成器 |
| `SystemDecorations.None` | 支持 | 支持 | 支持 |
| `ExtendClientAreaToDecorationsHint` | 支持 | 支持 | 支持有限 |
| 模态对话框 | 会阻塞父窗口 | 在 macOS 上表现为 sheet 样式 | 会阻塞父窗口 |

## 另请参阅

- [Main Window](/docs/fundamentals/main-window)：应用的主窗口。
- [Application Lifetimes](/docs/fundamentals/application-lifetimes)：应用生命周期如何管理窗口。
