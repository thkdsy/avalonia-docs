---
id: removing-the-titlebar
title: 移除标题栏
---

默认情况下，XPF 应用程序包含一个用于窗口管理的标题栏。不过，在某些情况下，你可能希望移除标题栏。本指南将带你了解移除标题栏的过程。
本文介绍了两种自定义窗口外观的方法：使用 WPF API，以及使用 Avalonia API。
你可以使用 WPF API 来移除标题栏，但 Avalonia API 能够对标题栏设置提供更高程度的灵活性和控制。推荐使用 Avalonia 方案。

## 使用 WPF API
在 WPF 中，你可以通过将 `WindowStyle` 属性设置为 `None` 来移除窗口的标题栏。此外，你可能还希望将 `AllowsTransparency` 属性设置为 `True`，以移除窗口的可调整大小边框。
```xml
<Window x:Class="YourNamespace.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Your Window Title" Height="350" Width="525"
        WindowStyle="None" AllowsTransparency="True">
    <!-- 你的内容放在这里 -->
</Window>
```

:::note
请记住，当你移除标题栏时，可能需要实现用于窗口拖动、调整大小和关闭的自定义控件，因为默认的标题栏功能将不再可用。
:::

```csharp
using System.Windows;

namespace YourNamespace
{
    public partial class MainWindow : Window
    {
        public MainWindow()
        {
            InitializeComponent();
        }

        private void Window_MouseLeftButtonDown(object sender, MouseButtonEventArgs e)
        {
            DragMove();
        }

        private void CloseButton_Click(object sender, RoutedEventArgs e)
        {
            Close();
        }
    }
}
```

然后在 XAML 中，你需要附加事件处理程序：

```xml
<Window x:Class="YourNamespace.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Your Window Title" Height="350" Width="525"
        WindowStyle="None" AllowsTransparency="True"
        MouseLeftButtonDown="Window_MouseLeftButtonDown">
    <!-- 你的内容放在这里 -->

    <!-- 关闭按钮示例 -->
    <Button Content="X" HorizontalAlignment="Right" VerticalAlignment="Top" Margin="0,5,5,0" Click="CloseButton_Click"/>
</Window>
```

这个示例展示了一个基础设置，你可能需要根据你的具体需求以及你希望提供的功能进行调整。

## 使用 Avalonia API

首先，你需要找到 `MainWindow.xaml.cs` 文件。然后，你只需要粘贴下面的代码。

```csharp
protected override void OnSourceInitialized(EventArgs e)
{
    base.OnSourceInitialized(e);

    if (XpfWpfAbstraction.IsRunningOnXpf)
    {
        if (XpfWpfAbstraction.GetAvaloniaWindowForWindow(this) is { } window)
        {
            window.ExtendClientAreaToDecorationsHint = true;
            window.ExtendClientAreaChromeHints = Avalonia.Platform.ExtendClientAreaChromeHints.NoChrome;
        }

    }
}
```
`ExtendClientAreaToDecorationsHint` 负责移除标题栏，但你仍然会有关闭、最小化和全屏按钮。 
如果你不需要它们，你需要将 `ExtendClientAreaChromeHints` 设置为 `NoChrome`。

:::note
请记住，当你指定这些设置时，窗口将无法拖动，因为 `XpfHost` 控件会吞掉点击事件，因为它的 `IsHitTestVisible` 被设置为 `True`。要解决这个问题，你需要例如在它顶部设置一个 `margin`，为标题栏预留该区域。
:::