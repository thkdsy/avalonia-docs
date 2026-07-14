---
id: embedding-avalonia-in-xpf
title: 在 XPF 中嵌入 Avalonia
---

## 嵌入 Avalonia 控件

### 步骤 1：添加一个 Avalonia `UserControl`

向你的应用程序中添加一个 Avalonia `UserControl`，其中包含你希望承载的 Avalonia 内容。例如：

```xml
<UserControl xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
             xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
             mc:Ignorable="d" d:DesignWidth="800" d:DesignHeight="450"
             x:Class="MyXpfApplication.MyAvaloniaView">
  <Button>Hello Avalonia!</Button>
</UserControl>
```

```csharp
using Avalonia.Controls;

namespace MyXpfApplication;

public partial class MyAvaloniaView : UserControl
{
    public MyAvaloniaView()
    {
        InitializeComponent();
    }
}
```

### 步骤 2：宿主化 Avalonia `UserControl`

实例化一个 `AvaloniaHost`，以便在 XPF 控件中承载 Avalonia 内容：

```xml
<Window x:Class="MyXpfApplication.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:MyXpfApplication"
        // highlight-next-line
        xmlns:xpf="clr-namespace:Atlantis;assembly=PresentationCore"
        mc:Ignorable="d"
        Title="MainWindow">
  // highlight-start
  <xpf:AvaloniaHost>
    <local:MyAvaloniaView/>
  </xpf:AvaloniaHost>
  // highlight-end
</Window>
```

## 为 Avalonia 控件设置样式

在 XPF 中，你只能通过 code-behind 为 Avalonia 控件添加 Style。请参考下面的示例。

### XAML 代码

下面是一个 XAML 代码片段示例，演示如何将一个 Avalonia 控件，具体来说是一个 `Button`，嵌入到 XPF `Window` 中：

```xml
<Window
    x:Class="YourNamespace.MainWindow"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:atlantis="clr-namespace:Atlantis;assembly=PresentationCore"
    xmlns:avalonia="clr-namespace:Avalonia.Controls;assembly=Avalonia.Controls"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    Title="Avalonia Embedded in XPF"
    Width="800"
    Height="450"
    mc:Ignorable="d">
    <atlantis:AvaloniaHost>
        <!-- Avalonia control within AvaloniaHost -->
        <avalonia:Button Content="Click me" x:Name="myButton" />
    </atlantis:AvaloniaHost>
</Window>
```

### Code-behind C# 代码

在 code-behind 文件（`MainWindow.xaml.cs`）中，你可以使用 `Styles` 属性为 Avalonia 控件应用样式。下面的 C# 代码演示了如何为 Avalonia `Button` 创建一个样式：

```csharp
using Avalonia.Styling;
using System.Linq;
using System.Windows;
using Setter = Avalonia.Styling.Setter;
using Style = Avalonia.Styling.Style;
using Button = Avalonia.Controls.Button;
using SolidColorBrush = Avalonia.Media.SolidColorBrush;

namespace YourNamespace
{
    /// <summary>
    /// MainWindow.xaml 的交互逻辑
    /// </summary>
    public partial class MainWindow : Window
    {

        public MainWindow()
        {
            InitializeComponent();

            // 为 Button 创建一个样式
            var myButtonStyle = new Style(x => x.OfType<Button>())
            {
                Setters = 
                {
                    new Setter(Button.BackgroundProperty, new SolidColorBrush(Avalonia.Media.Colors.Green)),
                    new Setter(Button.ForegroundProperty, new SolidColorBrush(Avalonia.Media.Colors.Red))
                    // 根据需要添加更多 setter
                }
            };

            // 将样式应用到 Button
            myButton.Styles.Add(myButtonStyle);
        }

    }
}
```
请确保将 "YourNamespace" 替换为你项目的实际命名空间。此示例将 Avalonia `Button` 的背景色设置为绿色，前景色设置为红色，该按钮被嵌入在 XPF 中。请根据你的样式需求调整这些 setter 和其他属性。如果你正确执行所有步骤，你的 Avalonia 控件将会根据样式发生变化。

## 动态添加全局样式

在 code-behind 中动态为 Avalonia 控件添加全局样式可以提供灵活性，并允许你在运行时应用样式。
使用以下 C# 代码来实现这一点：
```csharp
 // Retrieve the current Avalonia application instance
var avaloniaApp = Avalonia.Controls.Application.Current;

// Dynamically add a global style for Button controls
avaloniaApp.Styles.Add(new StyleInclude()
{
    Source = new Uri("avares://YourNamespace/Styles/CustomStyles.xaml") // Adjust the URI accordingly
});
```
这里的 "CustomStyles.xaml" 是包含你希望全局应用的 Avalonia 样式的 XAML 文件。

### 使用自定义 Avalonia 应用程序
在更高级的场景中，你可能需要用自定义样式完全替换默认应用的样式，
为此你需要重新定义 Avalonia Application。第一步是禁用自动 XPF 初始化。
```xml
  <PropertyGroup>
    <DisableAutomaticXpfInit>true</DisableAutomaticXpfInit>
  </PropertyGroup>
```
然后，你需要创建一个新的 Avalonia Application，并配套 XAML 和 code-behind。在 XAML 中放置你想要全局定义的样式。

:::note
`DataGrid` 主题是 DevTools 正常工作的必需条件。
:::

```xml
 <StyleInclude Source="avares://Avalonia.Controls.DataGrid/Themes/Simple.xaml"/>
```

之后，你需要创建一个单独的类，用于为你的 XPF 项目初始化 Avalonia。 

```csharp
public class MyXpfAvaloniaInitializer
{
    [ModuleInitializer]
    public static void Init()
    {
        if (Avalonia.Application.Current == null)
        {
            AppBuilder.Configure<MyApp>()
                .UsePlatformDetect()
                .With(new Win32PlatformOptions()
                {
                    // 默认使用系统 DPI 感知。如果进程在清单中设置了不同的感知级别，则该值将由操作系统优先采用
                    DpiAwareness = Win32DpiAwareness.SystemDpiAware
                })
                .WithAvaloniaXpf()
                .SetupWithLifetime(new ClassicDesktopStyleApplicationLifetime() { ShutdownMode = Avalonia.Controls.ShutdownMode.OnExplicitShutdown });
        }
    }
}
```

:::note
这里 `ModuleInitializer` 特性不是必需的。你可以在任何地方自行初始化 Avalonia，但你应该记住，Avalonia 的初始化应在 WPF 初始化之前完成。对于使用 XPF 和 F# 的人来说，这一点可能会很有用。
:::

这样之后，你的样式就会应用到整个应用程序。

## 访问 Avalonia 功能

有时 WPF API 可能无法提供你所需的特定功能。在这些情况下，通常可以使用 Avalonia API 来弥补这一缺口。

## 获取 Avalonia 窗口

许多 Avalonia 功能都是通过顶层 `Window` 类暴露的。由于 XPF 的 `Window` 也是一个 Avalonia `Window`，你可以使用以下模式来获取底层 Avalonia `Window` 的引用：

```csharp
if (XpfWpfAbstraction.GetAvaloniaWindowForWindow(xpfWindow) is { } avaloniaWindow)
{
    // 你现在拥有了一个 Avalonia Window。
}
```