---
id: notifications-how-to
title: "如何：显示通知和 Toast"
description: 学习如何在 Avalonia 应用程序中跨桌面和移动平台显示覆盖通知、Toast 消息、状态栏和系统托盘通知。
doc-type: how-to
---

本指南介绍如何在 Avalonia 应用程序中显示通知、Toast 消息和状态栏。由于 Avalonia 本身不提供内置通知控件，因此通常需要基于 [`ItemsControl`](/api/avalonia/controls/itemscontrol)、`Panel`、`Border` 等标准布局控件自行构建。

## 覆盖式通知面板

一种常见模式，是让通知面板从窗口角落滑入。你可以使用 `ItemsControl` 和过渡动画来实现它。

### 通知模型

```csharp
public partial class NotificationViewModel : ObservableObject
{
    public string Message { get; }
    public string Type { get; } // "info", "success", "error", "warning"

    public NotificationViewModel(string message, string type = "info")
    {
        Message = message;
        Type = type;
    }
}
```

### 通知服务

下面这个服务用于管理当前活动通知集合，并处理自动关闭。需要注意，之所以要用 `Dispatcher.UIThread.Post`，是因为 `Task.Delay(...).ContinueWith(...)` 会在线程池线程上继续执行，而集合变更必须切回 UI 线程。

```csharp
public partial class NotificationService : ObservableObject
{
    public ObservableCollection<NotificationViewModel> Notifications { get; } = new();

    public void Show(string message, string type = "info", int durationMs = 3000)
    {
        var notification = new NotificationViewModel(message, type);
        Notifications.Add(notification);

        // 在指定时间后自动关闭
        if (durationMs > 0)
        {
            _ = Task.Delay(durationMs).ContinueWith(_ =>
                Dispatcher.UIThread.Post(() => Notifications.Remove(notification)));
        }
    }

    [RelayCommand]
    private void Dismiss(NotificationViewModel notification)
    {
        Notifications.Remove(notification);
    }
}
```

:::tip
如果把 `durationMs` 设为 `0`，通知就会一直保留，直到用户手动关闭它。这对那些需要用户明确注意的错误信息特别有用。
:::

### XAML 覆盖层

把下面这段内容放到主窗口根节点的 `Panel` 中，并且作为最后一个子元素，这样它就会覆盖在其他内容之上：

```xml
<Panel>
    <!-- 主内容 -->
    <ContentControl Content="{Binding CurrentView}" />

    <!-- 通知覆盖层 -->
    <ItemsControl ItemsSource="{Binding Notifications.Notifications}"
                  HorizontalAlignment="Right" VerticalAlignment="Top"
                  Margin="16" MaxWidth="360">
        <ItemsControl.ItemTemplate>
            <DataTemplate>
                <Border Background="#1E1E2E" CornerRadius="8" Padding="12,8"
                        Margin="0,0,0,8" BorderBrush="#333" BorderThickness="1">
                    <Grid ColumnDefinitions="*,Auto">
                        <TextBlock Text="{Binding Message}" TextWrapping="Wrap"
                                   Foreground="White" VerticalAlignment="Center" />
                        <Button Grid.Column="1" Content="x" FontSize="10"
                                Background="Transparent" Foreground="Gray"
                                Padding="4,2" Margin="8,0,0,0"
                                Command="{Binding $parent[ItemsControl].((vm:NotificationService)DataContext).DismissCommand}"
                                CommandParameter="{Binding}" />
                    </Grid>
                </Border>
            </DataTemplate>
        </ItemsControl.ItemTemplate>
    </ItemsControl>
</Panel>
```

:::note
在移动平台（Android 和 iOS）上，建议把通知放在屏幕顶部，以避免遮挡软导航按钮。同时还应调整 `VerticalAlignment` 和 `Margin`，以适配刘海屏或圆角屏幕的安全区域边距。
:::

### 在视图模型中使用

```csharp
public partial class MainViewModel : ObservableObject
{
    public NotificationService Notifications { get; } = new();

    [RelayCommand]
    private async Task SaveAsync()
    {
        await _repository.SaveAsync();
        Notifications.Show("Changes saved successfully.", "success");
    }
}
```

## 简单状态栏

如果你希望反馈方式更克制一些，可以在窗口底部使用状态栏：

```xml
<DockPanel>
    <Border DockPanel.Dock="Bottom" Background="#F3F4F6" Padding="8,4">
        <TextBlock Text="{Binding StatusMessage}" FontSize="12" Foreground="Gray" />
    </Border>

    <!-- Main content -->
    <ContentControl Content="{Binding CurrentView}" />
</DockPanel>
```

```csharp
[ObservableProperty]
private string _statusMessage = "Ready";

[RelayCommand]
private async Task LoadDataAsync()
{
    StatusMessage = "Loading...";
    await _dataService.LoadAsync();
    StatusMessage = "Loaded 42 items.";

    // 延迟后清除
    await Task.Delay(3000);
    StatusMessage = "Ready";
}
```

:::warning
如果用户在短时间内多次触发 `LoadDataAsync`，较早一次调用中的 `Task.Delay` 可能会在较新的操作尚未完成时，错误地把状态消息重置掉。为了避免这种情况，可以使用 `CancellationTokenSource`，并在每次重新进入该方法时取消前一次延迟。
:::

## 确认提示横幅

对于重要消息，可以在页面顶部显示一个提示横幅：

```xml
<StackPanel>
    <!-- 横幅 -->
    <Border Background="#FEF3C7" Padding="12,8"
            IsVisible="{Binding ShowBanner}">
        <Grid ColumnDefinitions="*,Auto">
            <TextBlock Text="{Binding BannerMessage}" Foreground="#92400E"
                       VerticalAlignment="Center" />
            <Button Grid.Column="1" Content="Dismiss"
                    Command="{Binding DismissBannerCommand}"
                    Background="Transparent" Foreground="#92400E" />
        </Grid>
    </Border>

    <!-- 页面内容 -->
    <ContentControl Content="{Binding CurrentPage}" />
</StackPanel>
```

## 托盘图标通知（仅桌面端）

在桌面平台（Windows、macOS、Linux）上，你可以使用 `TrayIcon` 控件与系统托盘集成：

```xml
<TrayIcon.Icons>
    <TrayIcons>
        <TrayIcon Icon="/Assets/app-icon.ico"
                  ToolTipText="My Application"
                  Command="{Binding ShowWindowCommand}">
            <TrayIcon.Menu>
                <NativeMenu>
                    <NativeMenuItem Header="Show" Command="{Binding ShowWindowCommand}" />
                    <NativeMenuItem Header="Exit" Command="{Binding ExitCommand}" />
                </NativeMenu>
            </TrayIcon.Menu>
        </TrayIcon>
    </TrayIcons>
</TrayIcon.Icons>
```

### 平台注意事项

| 平台 | 说明 |
|----------|-------|
| **Windows** | 完整支持托盘图标和气泡通知。图标建议使用 `.ico` 格式。 |
| **macOS** | 会显示在菜单栏中。按照 macOS 规范，建议菜单栏图标使用模板图像（单色 PNG）。 |
| **Linux** | 支持程度取决于桌面环境。GNOME、KDE、XFCE 通常会通过 `libappindicator` 或 `StatusNotifierItem` 协议支持托盘图标。 |
| **Android / iOS / Browser** | 这些平台不支持 `TrayIcon`。请改用应用内覆盖通知。 |

## 按类型区分颜色的通知

你可以基于通知类型，使用类选择器给它们应用不同样式：

```xml
<ItemsControl.ItemTemplate>
    <DataTemplate>
        <Border CornerRadius="8" Padding="12,8" Margin="0,0,0,8"
                Classes.info="{Binding Type, Converter={StaticResource EqualConverter}, ConverterParameter=info}"
                Classes.success="{Binding Type, Converter={StaticResource EqualConverter}, ConverterParameter=success}"
                Classes.error="{Binding Type, Converter={StaticResource EqualConverter}, ConverterParameter=error}">
            <TextBlock Text="{Binding Message}" Foreground="White" />
        </Border>
    </DataTemplate>
</ItemsControl.ItemTemplate>

<ItemsControl.Styles>
    <Style Selector="Border.info">
        <Setter Property="Background" Value="#3B82F6" />
    </Style>
    <Style Selector="Border.success">
        <Setter Property="Background" Value="#22C55E" />
    </Style>
    <Style Selector="Border.error">
        <Setter Property="Background" Value="#EF4444" />
    </Style>
</ItemsControl.Styles>
```

:::tip
建议除了 `info`、`success` 和 `error` 之外，再增加一个 `warning` 样式（例如 `#F59E0B`），这样就能覆盖四种常见通知级别。
:::

## 另请参阅

- [Threading](/docs/app-development/threading): Understand UI thread marshalling with `Dispatcher.UIThread`.
- [TrayIcon](/controls/navigation/trayicon): System tray integration for desktop platforms.
- [Flyout](/controls/layout/containers/flyout): Popup content attached to controls.
- [ToolTip](/controls/feedback/tooltip): Hover tooltips for controls.
- [ItemsControl](/docs/how-to/itemscontrol-how-to): Working with `ItemsControl` for dynamic lists.
- [Data templates](/docs/data-templates/introduction-to-data-templates): Customize how notification items render.
