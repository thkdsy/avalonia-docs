---
id: notification
title: WindowNotificationManager
description: 一个吐司风格的通知弹出系统，可在窗口中的可配置位置显示临时消息。
doc-type: reference
---

[`WindowNotificationManager`](/api/avalonia/controls/notifications/windownotificationmanager) 提供了一个内置的通知弹出系统。它会在窗口中的可配置位置显示吐司风格的消息。你可以用它在不阻塞其余 UI 交互的情况下，向用户提示操作完成、警告、错误或其他事件。

## 常用属性

| 属性 | 类型 | 说明 |
|---|---|---|
| `Position` | `NotificationPosition` | 通知显示的位置。可选：`TopLeft`、`TopCenter`、`TopRight`、`BottomLeft`、`BottomCenter`、`BottomRight`。默认值为 `TopRight`。 |
| `MaxItems` | `int` | 同一时间可见的通知最大数量。默认值为 `5`。 |

## 通知属性

内置的 `Notification` 类公开了以下属性：

| 属性 | 类型 | 说明 |
|---|---|---|
| `Title` | `string` | 通知标题文本。 |
| `Message` | `string` | 通知正文文本。 |
| `Type` | `NotificationType` | 严重级别：`Information`、`Success`、`Warning` 或 `Error`。 |
| `Expiration` | `TimeSpan` | 通知自动关闭前的持续时间。设置为 `TimeSpan.Zero` 表示需要手动关闭。 |

## 设置

在你的窗口中注册一个 `WindowNotificationManager`，通常是在 code-behind 中完成，或通过视图模型引用：

```csharp
public partial class MainWindow : Window
{
    private WindowNotificationManager _notificationManager;

    public MainWindow()
    {
        InitializeComponent();

        _notificationManager = new WindowNotificationManager(this)
        {
            Position = NotificationPosition.BottomRight,
            MaxItems = 3
        };
    }
}
```

如果你更偏好基于标记的方式，也可以在 XAML 中声明 `WindowNotificationManager`：

```xml
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        x:Class="MyApp.MainWindow">
    <Panel>
        <WindowNotificationManager x:Name="NotificationManager"
                                   Position="TopRight"
                                   MaxItems="5" />
        <!-- 你的其他内容写在这里 -->
    </Panel>
</Window>
```

然后你可以在 code-behind 中访问该管理器：

```csharp
public partial class MainWindow : Window
{
    public MainWindow()
    {
        InitializeComponent();
        var manager = this.FindControl<WindowNotificationManager>("NotificationManager");
    }
}
```

## 显示通知

传入一个 `Notification` 对象调用 `Show`：

```csharp
_notificationManager.Show(new Notification(
    "File saved",
    "Your document has been saved successfully.",
    NotificationType.Success,
    TimeSpan.FromSeconds(3)));
```

如果省略 `Expiration` 参数，通知会使用默认的过期时间。若要让通知一直保持显示，直到用户手动关闭它，请传入 `TimeSpan.Zero`：

```csharp
_notificationManager.Show(new Notification(
    "Action required",
    "Please review the pending changes before continuing.",
    NotificationType.Warning,
    TimeSpan.Zero));
```

## 通知类型

使用 `NotificationType` 枚举来表达严重级别。内置主题会为每种类型应用不同的颜色：

```csharp
// 信息（默认蓝色）
_notificationManager.Show(new Notification("Info", "Operation started.", NotificationType.Information));

// 成功（绿色）
_notificationManager.Show(new Notification("Done", "Upload complete.", NotificationType.Success));

// 警告（黄色/橙色）
_notificationManager.Show(new Notification("Warning", "Disk space is low.", NotificationType.Warning));

// 错误（红色）
_notificationManager.Show(new Notification("Error", "Connection failed.", NotificationType.Error));
```

## 以编程方式关闭通知

你可以在代码中关闭某个特定通知，或清除所有通知：

```csharp
var notification = new Notification("Processing", "Working...", NotificationType.Information, TimeSpan.Zero);
_notificationManager.Show(notification);

// 关闭某个特定通知
_notificationManager.Close(notification);

// 清除所有活动通知
_notificationManager.CloseAll();
```

当一个长时间运行的操作完成后，你想用结果通知替换进度通知时，这会很有用：

```csharp
var progressNotification = new Notification(
    "Uploading",
    "Sending files to the server...",
    NotificationType.Information,
    TimeSpan.Zero);

_notificationManager.Show(progressNotification);

// 稍后，当上传完成时：
_notificationManager.Close(progressNotification);
_notificationManager.Show(new Notification(
    "Upload complete",
    "All files have been sent successfully.",
    NotificationType.Success,
    TimeSpan.FromSeconds(3)));
```

## 自定义通知内容

实现 `INotification` 以提供自定义通知数据：

```csharp
public class CustomNotification : INotification
{
    public string Title { get; set; }
    public string Message { get; set; }
    public NotificationType Type { get; set; }
    public TimeSpan Expiration { get; set; }
    public string ActionText { get; set; }
    public Action OnAction { get; set; }

    public Action? OnClose { get; set; }
    public Action? OnClick { get; set; }
}
```

之后你可以用同样的方式显示自定义通知：

```csharp
_notificationManager.Show(new CustomNotification
{
    Title = "New message",
    Message = "You have a new message from support.",
    Type = NotificationType.Information,
    Expiration = TimeSpan.FromSeconds(5),
    ActionText = "View",
    OnAction = () => NavigateToMessages()
});
```

## 定位

通过设置 `Position` 控制通知出现的位置：

```csharp
// 右上角（默认）
_notificationManager.Position = NotificationPosition.TopRight;

// 底部居中（适合移动端风格的吐司提示）
_notificationManager.Position = NotificationPosition.BottomCenter;
```

可用的位置共有六种：

| 位置 | 说明 |
|---|---|
| `TopLeft` | 窗口左上角。 |
| `TopCenter` | 顶部边缘，水平居中。 |
| `TopRight` | 窗口右上角（默认）。 |
| `BottomLeft` | 窗口左下角。 |
| `BottomCenter` | 底部边缘，水平居中。 |
| `BottomRight` | 窗口右下角。 |

## MVVM 模式

通过服务暴露通知管理器，这样视图模型就能在不直接引用 UI 类型的情况下显示通知：

```csharp
public interface INotificationService
{
    void ShowInfo(string title, string message);
    void ShowSuccess(string title, string message);
    void ShowWarning(string title, string message);
    void ShowError(string title, string message);
}

public class NotificationService : INotificationService
{
    private readonly WindowNotificationManager _manager;

    public NotificationService(WindowNotificationManager manager)
    {
        _manager = manager;
    }

    public void ShowInfo(string title, string message) =>
        _manager.Show(new Notification(title, message, NotificationType.Information));

    public void ShowSuccess(string title, string message) =>
        _manager.Show(new Notification(title, message, NotificationType.Success));

    public void ShowWarning(string title, string message) =>
        _manager.Show(new Notification(title, message, NotificationType.Warning));

    public void ShowError(string title, string message) =>
        _manager.Show(new Notification(title, message, NotificationType.Error));
}
```

在应用启动时注册该服务，并将其注入到需要显示通知的视图模型中：

```csharp
public class MyViewModel
{
    private readonly INotificationService _notifications;

    public MyViewModel(INotificationService notifications)
    {
        _notifications = notifications;
    }

    public void SaveDocument()
    {
        // 执行保存逻辑...
        _notifications.ShowSuccess("Saved", "Your document has been saved.");
    }
}
```

## 另请参阅

- [How to show notifications and toasts](/docs/how-to/notifications-how-to)
- [Popup](/controls/feedback/popup)
- [ToolTip](/controls/feedback/tooltip)
