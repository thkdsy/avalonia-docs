---
id: window-how-to
title: "如何：使用窗口"
description: 了解窗口大小、位置、对话框、多窗口应用、启动行为以及系统窗口装饰选项。
doc-type: how-to
---

本指南介绍常见的 Window 使用场景：大小控制、位置设置、对话框、多窗口应用、启动行为以及系统窗口装饰。

## 设置窗口大小和位置

### 启动时固定大小

```xml
<Window Width="800" Height="600"
        WindowStartupLocation="CenterScreen">
```

### 最小和最大尺寸

```xml
<Window MinWidth="400" MinHeight="300"
        MaxWidth="1920" MaxHeight="1080">
```

### 启动位置

| 值 | 说明 |
|---|---|
| `Manual` | 由 `Position` 属性指定位置。 |
| `CenterScreen` | 在主屏幕中央显示。 |
| `CenterOwner` | 在所有者窗口中央显示（常用于对话框）。 |

## 显示对话框窗口

使用 `ShowDialog<T>` 打开模态对话框并获取返回结果：

```csharp
var dialog = new SettingsWindow();
var result = await dialog.ShowDialog<bool>(this);
if (result)
{
    // 用户确认了操作
}
```

通过带参数调用 `Close` 来返回结果值：

```csharp
// 在对话框窗口中
private void OnOkClick(object sender, RoutedEventArgs e)
{
    Close(true);
}

private void OnCancelClick(object sender, RoutedEventArgs e)
{
    Close(false);
}
```

## 获取父窗口

在任意控件中，都可以使用 `TopLevel.GetTopLevel`：

```csharp
var topLevel = TopLevel.GetTopLevel(this);
if (topLevel is Window window)
{
    await new MyDialog().ShowDialog<bool>(window);
}
```

## 阻止窗口关闭

处理 `Closing` 事件以拦截关闭操作：

```csharp
protected override void OnClosing(WindowClosingEventArgs e)
{
    if (HasUnsavedChanges)
    {
        e.Cancel = true;
        // 显示保存提示
    }
    base.OnClosing(e);
}
```

## 窗口状态（最小化、最大化、恢复）

```csharp
// 通过代码控制窗口状态
window.WindowState = WindowState.Maximized;
window.WindowState = WindowState.Minimized;
window.WindowState = WindowState.Normal;
```

```xml
<Button Content="Maximize" Command="{Binding MaximizeCommand}" />
```

```csharp
[RelayCommand]
private void Maximize()
{
    if (Application.Current?.ApplicationLifetime is IClassicDesktopStyleApplicationLifetime desktop)
    {
        var window = desktop.MainWindow;
        window.WindowState = window.WindowState == WindowState.Maximized
            ? WindowState.Normal
            : WindowState.Maximized;
    }
}
```

## 禁用最小化和最大化按钮

使用 `CanMinimize` 和 `CanMaximize` 控制标题栏按钮是否可用：

```xml
<Window CanMinimize="False" CanMaximize="False">
```

当 `CanResize` 为 `false` 时，`CanMaximize` 会自动变为 `false`。

:::note
不同平台的行为有所差异。在 Windows 上，被禁用的按钮通常会隐藏；在 macOS 上，它们通常会显示为灰色；在 Linux 上，具体行为取决于窗口管理器。
:::

## 隐藏标题栏（无边框窗口）

通过禁用系统窗口装饰来创建无边框窗口：

```xml
<Window SystemDecorations="None"
        ExtendClientAreaToDecorationsHint="True"
        Background="Transparent"
        TransparencyLevelHint="AcrylicBlur">
```

### 带拖拽区域的自定义标题栏

使用附加属性 `WindowDecorationProperties.ElementRole` 将某个元素标记为标题栏拖拽区域。这样操作系统就会自动处理拖动和双击最大化等行为：

```xml
<Grid RowDefinitions="32,*">
    <!-- 自定义标题栏 -->
    <Border Grid.Row="0" Background="#1E1E2E"
            WindowDecorationProperties.ElementRole="TitleBar">
        <DockPanel Margin="8,0">
            <TextBlock Text="My App" VerticalAlignment="Center" Foreground="White" />
            <StackPanel DockPanel.Dock="Right" Orientation="Horizontal"
                        HorizontalAlignment="Right">
                <Button Content="_" Command="{Binding MinimizeCommand}" />
                <Button Content="□" Command="{Binding MaximizeCommand}" />
                <Button Content="✕" Command="{Binding CloseCommand}" />
            </StackPanel>
        </DockPanel>
    </Border>

    <!-- 内容 -->
    <ContentControl Grid.Row="1" Content="{Binding CurrentView}" />
</Grid>
```

`ElementRole` 属性支持以下取值：

| 值 | 行为 |
|---|---|
| `None` | 不启用任何特殊窗口装饰行为（默认）。 |
| `TitleBar` | 作为可拖动的标题栏区域。 |
| `ResizeN`, `ResizeS`, `ResizeE`, `ResizeW` | 对应边缘的缩放拖拽区域。 |
| `ResizeNE`, `ResizeNW`, `ResizeSE`, `ResizeSW` | 对应角落的缩放拖拽区域。 |

位于 `TitleBar` 区域内的交互控件（例如按钮）仍然会正常接收输入，不会触发窗口拖动。

## 多窗口应用程序

从主窗口中打开额外窗口：

```csharp
[RelayCommand]
private void OpenNewWindow()
{
    var window = new SecondaryWindow
    {
        DataContext = new SecondaryViewModel()
    };
    window.Show();
}
```

如果你想打开一个保持在所有者窗口之上的非模态窗口：

```csharp
var toolWindow = new ToolWindow();
toolWindow.Show(ownerWindow); // 保持显示在所有者窗口之上
```

## 保存和恢复窗口位置

```csharp
protected override void OnOpened(EventArgs e)
{
    base.OnOpened(e);
    var settings = LoadSettings();
    if (settings.WindowWidth > 0)
    {
        Width = settings.WindowWidth;
        Height = settings.WindowHeight;
    }
}

protected override void OnClosing(WindowClosingEventArgs e)
{
    SaveSettings(new AppSettings
    {
        WindowWidth = Width,
        WindowHeight = Height,
        WindowState = WindowState
    });
    base.OnClosing(e);
}
```

## 窗口透明效果

```xml
<!-- 亚克力模糊（依赖平台支持） -->
<Window TransparencyLevelHint="AcrylicBlur"
        Background="Transparent">
    <Panel>
        <ExperimentalAcrylicBorder Material="{DynamicResource AcrylicMaterial}" />
        <!-- 位于亚克力背景之上的内容 -->
    </Panel>
</Window>
```

可以在运行时检查当前支持哪些透明级别：

```csharp
var supported = this.ActualTransparencyLevel;
```

### 可穿透点击的透明窗口

如果你想创建一个透明覆盖窗口，并让鼠标点击穿过空白区域传递到底下的应用，请设置 `TransparencyLevelHint="Transparent"`，同时将窗口的 `Background` 设为 `{x:Null}` 来移除背景。放置在窗口中的交互控件仍然可以正常点击。

```xml
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        TransparencyLevelHint="Transparent"
        Background="{x:Null}"
        SystemDecorations="None"
        Topmost="True"
        WindowState="Maximized">
    <Grid>
        <!-- This button is clickable; empty areas pass input through -->
        <Button Content="Click Me"
                HorizontalAlignment="Center"
                VerticalAlignment="Center" />
    </Grid>
</Window>
```

`Background="{x:Null}"` 与 `Background="Transparent"` 之间的关键区别如下：

| Background value | Visual result | Hit testing |
|---|---|---|
| `{x:Null}` | Transparent | Empty areas pass clicks through to windows behind |
| `Transparent` | Transparent | Empty areas block clicks (the window captures all input) |

这种区别在 Avalonia 的各个层级都成立，从单独的面板到整个窗口本身都是如此。更多细节请参阅 [Background and hit testing](/docs/graphics-animation/hit-testing#background-and-hit-testing)。

:::tip[Migrating from WPF]
在 WPF 中，如果在 `Window` 上设置 `AllowsTransparency="True"` 并将 `Background="Transparent"`，默认情况下点击会穿过透明区域。Avalonia 的行为不同：`Background="Transparent"` 依然会捕获输入。若要获得类似 WPF 的点击穿透效果，请改用 `Background="{x:Null}"`。
:::

## 窗口图标

```xml
<Window Icon="/Assets/app-icon.ico">
```

或者通过代码设置：

```csharp
Icon = new WindowIcon(AssetLoader.Open(new Uri("avares://MyApp/Assets/app-icon.ico")));
```

## 关键属性

| 属性 | 类型 | 说明 |
|---|---|---|
| `Title` | `string` | 窗口标题栏文本。 |
| `WindowState` | `WindowState` | `Normal`、`Minimized`、`Maximized`、`FullScreen`。 |
| `WindowStartupLocation` | `WindowStartupLocation` | `Manual`、`CenterScreen`、`CenterOwner`。 |
| `SystemDecorations` | `SystemDecorations` | `Full`、`BorderOnly`、`None`。 |
| `CanResize` | `bool` | 用户是否可以调整窗口大小。 |
| `CanMinimize` | `bool` | 最小化按钮是否启用，默认是 `true`。 |
| `CanMaximize` | `bool` | 最大化按钮是否启用，默认是 `true`。当 `CanResize` 为 `false` 时会自动变成 `false`。 |
| `Topmost` | `bool` | 是否始终保持在其他窗口之上。 |
| `ShowInTaskbar` | `bool` | 是否在操作系统任务栏中显示。 |
| `Icon` | `WindowIcon` | 标题栏和任务栏使用的窗口图标。 |
| `TransparencyLevelHint` | `WindowTransparencyLevel` | 请求的透明效果：`None`、`Transparent`、`Blur`、`AcrylicBlur`、`Mica`。 |

## 另请参阅

- [Window Control Reference](/controls/primitives/window)：属性表说明。
- [Dialogs How-To](/docs/how-to/dialogs-how-to)：对话框模式与文件选择器。
- [Window Management](/docs/app-development/window-management)：窗口生命周期管理。
- [Application Lifetimes](/docs/fundamentals/application-lifetimes)：桌面端与移动端生命周期模型。
