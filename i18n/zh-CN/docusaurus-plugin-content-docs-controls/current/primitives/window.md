---
id: window
title: Window
description: 一个顶级内容控件，用于表示带有标题栏、图标以及关闭/最小化/最大化窗口装饰的操作系统窗口。
doc-type: reference
---

[`Window`](/api/avalonia/controls/window) 是一个顶级 [`ContentControl`](/controls/data-display/contentcontrol)，表示一个操作系统窗口。它提供了桌面应用用户所期望的标题栏、图标以及系统窗口装饰（关闭、最小化、最大化按钮）。

通常你不会直接创建 `Window` 实例，而是会为应用所需的每一种窗口类型派生一个 `Window` 子类。

:::tip
`Window` 仅在桌面平台（Windows、macOS、Linux）上可用。如果你的目标平台是移动端或浏览器，请改用 [`UserControl`](/controls/primitives/usercontrol) 并配合导航框架。
:::

## 常用属性

| 属性 | 类型 | 说明 |
| :--- | :--- | :--- |
| `Title` | `string` | 显示在标题栏中的文本。 |
| `Icon` | `WindowIcon` | 显示在标题栏和任务栏中的图标。 |
| `SizeToContent` | `SizeToContent` | 控制窗口是否自动调整尺寸，以在水平方向、垂直方向或两个方向上适应其内容。 |
| `WindowState` | `WindowState` | 获取或设置窗口状态：`Normal`、`Minimized`、`Maximized` 或 `FullScreen`。 |
| `CanResize` | `bool` | 获取或设置用户是否可以调整窗口大小。 |
| `ShowInTaskbar` | `bool` | 获取或设置窗口是否出现在操作系统任务栏中。 |
| `Topmost` | `bool` | 获取或设置窗口是否始终位于其他窗口之上。 |
| `WindowDecorations` | `WindowDecorations` | 控制窗口装饰（标题栏和边框）。设为 `None` 时可创建无边框窗口。 |
| `ExtendClientAreaToDecorationsHint` | `bool` | 当为 `true` 时，你的内容会延伸到标题栏区域，从而支持自定义窗口边框。 |

## 显示、隐藏和关闭窗口

你可以通过调用 `Show` 方法来显示一个窗口：

```csharp
var window = new MyWindow();
window.Show();
```

你也可以通过调用 `Close` 来关闭窗口。这与用户点击窗口关闭按钮的效果相同：

```csharp
window.Close();
```

:::warning
窗口一旦被关闭，就不能再次显示。对已关闭的窗口调用 `Show` 会抛出异常。如果你之后还需要再次显示同一个窗口，请使用 `Hide` 而不是 `Close`。
:::

```csharp
window.Hide();

// 之后还可以再次显示该窗口。
window.Show();
```

另请参阅[阻止窗口关闭](#阻止窗口关闭)。

## 将窗口显示为对话框

你可以通过调用 `ShowDialog` 将窗口显示为模态对话框。此方法要求你传入一个所有者窗口，以便系统知道该对话框隶属于哪个窗口：

```csharp
// “this” 是当前 Window 实例。
// 你也可以通过将 Application.ApplicationLifetime
// 转换为 IClassicDesktopStyleApplicationLifetime 来获取主窗口。
var ownerWindow = this;
var dialog = new MyWindow();
dialog.ShowDialog(ownerWindow);
```

`ShowDialog` 会返回一个 `Task`，因此你可以通过 `await` 等待对话框关闭：

```csharp
var dialog = new MyWindow();
await dialog.ShowDialog(ownerWindow);
```

### 从对话框返回结果

对话框可以通过向 `Close` 方法传入一个值来返回结果。调用方可通过泛型重载 `ShowDialog<T>` 读取该结果：

```csharp
public class MyDialog : Window
{
    public MyDialog()
    {
        InitializeComponent();
    }

    private void OkButton_Click(object? sender, EventArgs e)
    {
        Close("OK Clicked!");
    }
}
```

```csharp
var dialog = new MyDialog();

// 返回结果是字符串，因此使用 ShowDialog<string>。
var result = await dialog.ShowDialog<string>(ownerWindow);
```

## 阻止窗口关闭

你可以通过处理 `Closing` 事件并设置 `e.Cancel = true` 来阻止窗口关闭：

```csharp
window.Closing += (s, e) =>
{
    e.Cancel = true;
};
```

一种常见模式是隐藏窗口而不是关闭它，这样你之后还可以再次显示它：

```csharp
window.Closing += (s, e) =>
{
    ((Window)s!).Hide();
    e.Cancel = true;
};
```

## 实用说明

- **启动窗口。** 应用程序的主窗口通常在 `App.axaml.cs` 中，通过给 `IClassicDesktopStyleApplicationLifetime` 的 `MainWindow` 赋值来设置。详情请参阅 [主窗口](/docs/fundamentals/main-window)。
- **多个窗口。** 你可以根据需要创建并调用 `Show` 来打开任意数量的窗口。每个窗口都在同一个应用中独立运行。
- **定位。** 可使用 `Position` 属性（类型为 `PixelPoint`）设置窗口在屏幕上的坐标，或将 `WindowStartupLocation` 设为 `CenterScreen` 或 `CenterOwner`。
- **关闭行为。** 默认情况下，当最后一个窗口关闭时，应用程序会退出。你可以通过设置应用生命周期中的 `ShutdownMode` 来更改此行为。

## 另请参阅

- [主窗口](/docs/fundamentals/main-window)
- [窗口管理](/docs/app-development/window-management)
- [如何：使用窗口](/docs/how-to/window-how-to)
- [`ContentControl`](/controls/data-display/contentcontrol)
- [`UserControl`](/controls/primitives/usercontrol)
- [`Window` GitHub 源代码](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/Window.cs)
- [`Window` API 参考](/api/avalonia/controls/window)
