---
id: messagebox
title: 消息框
description: 了解为什么 Avalonia 不包含内置的 MessageBox，以及如何使用第三方库添加消息框功能。
doc-type: troubleshooting
---

Avalonia 不包含内置的 `MessageBox` 控件。这是一个经过深思熟虑的设计选择，因为 Avalonia 面向多个平台（桌面、移动端、浏览器），而传统的模态消息框并不总是适用。该功能正在考虑纳入未来开发计划。

如需更新和讨论，请参阅 GitHub 上的 [MessageBox 功能请求](https://github.com/AvaloniaUI/Avalonia/issues/670)。

## 为你的应用添加消息框功能

由于没有原生的 `MessageBox` API，你需要使用第三方库或自行构建对话框。按照以下步骤开始使用社区包：

### 第 1 步：选择一个库

查看下面列出的选项，并选择一个适合你项目的库。其中一些是免费且开源的，另一些是商业产品（用 `$` 标记）。

### 第 2 步：安装 NuGet 包

使用 .NET CLI 或你 IDE 中的 NuGet 包管理器安装该包。例如，要安装 `MessageBox.Avalonia`：

```bash
dotnet add package MessageBox.Avalonia
```

### 第 3 步：显示消息框

每个库都有自己的 API。以下示例使用 `MessageBox.Avalonia` 显示一个简单的信息对话框：

```csharp
using MsBox.Avalonia;
using MsBox.Avalonia.Enums;

var box = MessageBoxManager
    .GetMessageBoxStandard("Title", "Hello from Avalonia!", ButtonEnum.Ok);

await box.ShowAsync();
```

如果你需要获取用户的响应（例如，OK 还是 Cancel），请保存返回值：

```csharp
var result = await box.ShowAsync();

if (result == ButtonResult.Ok)
{
    // 处理确认
}
```

### 第 4 步：处理平台差异

请注意以下边界情况：

- **浏览器和移动端目标**：模态对话框的表现可能与桌面端不同。有些库会以内嵌方式渲染对话框，而不是作为单独窗口。请在你计划支持的每个平台上测试所选库。
- **单视图应用**：如果你的应用使用 `SingleViewApplicationLifetime`（移动端和浏览器中常见），你无法创建新的 `Window` 来承载对话框。请使用支持覆盖层或应用内对话框渲染的库，例如 `DialogHost.Avalonia`。
- **线程**：始终在 UI 线程上显示对话框。如果你是从后台线程调用，请使用 `Dispatcher.UIThread.InvokeAsync` 进行调度。

## 第三方 `MessageBox` 实现

| 库 | 类型 |
|---|---|
| [MessageBox.Avalonia](https://github.com/AvaloniaCommunity/MessageBox.Avalonia) | 免费 / 开源 |
| [DialogHost.Avalonia](https://github.com/AvaloniaUtils/DialogHost.Avalonia) | 免费 / 开源 |
| [Ursa.Avalonia](https://github.com/irihitech/Ursa.Avalonia) | 免费 / 开源 |
| [AtomUI.Avalonia](https://github.com/chinware/AtomUI) | 免费 / 开源 |
| [Actipro Avalonia UI Controls](https://www.actiprosoftware.com/products/controls/avalonia) | 商业 |
| [Eremex Avalonia UI Controls](https://eremexcontrols.net/controls/windows-and-dialogs/messagebox/) | 商业 |

## 构建你自己的消息框

如果你不想依赖第三方包，也可以自己创建一个简单的对话框窗口：

1. 新建一个 `Window`，在 AXAML 中布局消息内容和按钮。
2. 使用 `window.ShowDialog(ownerWindow)` 打开它，它会返回一个你可以 `await` 的 `Task`。
3. 在关闭之前，通过为 `Window.Close(result)` 赋值来设置对话框结果。

```xml
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Confirm"
        Width="300" Height="150"
        WindowStartupLocation="CenterOwner">
    <StackPanel Margin="20" Spacing="16">
        <TextBlock Text="Are you sure?" />
        <StackPanel Orientation="Horizontal" HorizontalAlignment="Right" Spacing="8">
            <Button Content="Yes" Click="OnYesClick" />
            <Button Content="No" Click="OnNoClick" />
        </StackPanel>
    </StackPanel>
</Window>
```

```csharp
public partial class ConfirmDialog : Window
{
    public ConfirmDialog()
    {
        InitializeComponent();
    }

    private void OnYesClick(object? sender, RoutedEventArgs e)
    {
        Close(true);
    }

    private void OnNoClick(object? sender, RoutedEventArgs e)
    {
        Close(false);
    }
}
```

要显示对话框并读取结果：

```csharp
var dialog = new ConfirmDialog();
var result = await dialog.ShowDialog<bool>(this);

if (result)
{
    // 用户点击了 Yes
}
```

这种方式仅适用于使用 `ClassicDesktopStyleApplicationLifetime` 的桌面应用。对于单视图应用，你需要改用基于覆盖层的解决方案。

## 另请参阅

- [Window 控件](/controls/primitives/window)
- [如何使用对话框](/docs/how-to/dialogs-how-to)
- [MessageBox 功能请求（GitHub）](https://github.com/AvaloniaUI/Avalonia/issues/670)