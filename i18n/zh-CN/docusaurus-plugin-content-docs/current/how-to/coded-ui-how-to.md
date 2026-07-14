---
id: coded-ui-how-to
title: "如何：不使用 XAML 构建完整应用"
description: 完全使用 C#、不依赖任何 XAML 文件来构建一个功能完整的 Avalonia 应用程序。
doc-type: how-to
---

本指南将带你一步步使用纯 C# 构建一个完整可运行的 Avalonia 应用程序，全程不使用任何 XAML 文件。你将创建一个简单的计数器应用，其中包含带样式的控件、布局、事件处理以及数据绑定，全部通过代码完成。

## 前置条件

- .NET 10 SDK 或更高版本
- 一个文本编辑器或 IDE（Visual Studio、Rider 或 VS Code）

## 步骤 1：创建项目

创建一个新的控制台应用程序，并添加 Avalonia 相关包：

```bash
dotnet new console -n CodedUIApp
cd CodedUIApp
dotnet add package Avalonia --version 12.0.0
dotnet add package Avalonia.Desktop --version 12.0.0
dotnet add package Avalonia.Themes.Fluent --version 12.0.0
```

你的 `.csproj` 应类似如下：

```xml title='CodedUIApp.csproj'
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net10.0</TargetFramework>
    <OutputType>Exe</OutputType>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Avalonia" Version="12.0.0" />
    <PackageReference Include="Avalonia.Desktop" Version="12.0.0" />
    <PackageReference Include="Avalonia.Themes.Fluent" Version="12.0.0" />
  </ItemGroup>
</Project>
```

请注意，这里没有 `Avalonia.Markup.Xaml` 包，因为在这个方案里你并不需要它。

## 步骤 2：启动应用程序

将 `Program.cs` 的内容替换为：

```csharp title='Program.cs'
using Avalonia;
using Avalonia.Controls;
using Avalonia.Themes.Fluent;

class Program
{
    public static void Main(string[] args)
    {
        AppBuilder.Configure<Application>()
                  .UsePlatformDetect()
                  .Start(AppMain, args);
    }

    static void AppMain(Application app, string[] args)
    {
        // 应用 Fluent 主题，让控件拥有更完整的默认外观
        app.Styles.Add(new FluentTheme());

        var window = new CounterWindow();
        window.Show();
        app.Run(window);
    }
}
```

`Start` 方法接收一个委托，该委托会在 Avalonia 完成初始化后执行。在这个委托内部，你可以访问 `Application` 实例，并继续添加主题、创建窗口以及启动事件循环。

## 步骤 3：构建窗口

创建一个名为 `CounterWindow.cs` 的新文件：

```csharp title='CounterWindow.cs'
using Avalonia;
using Avalonia.Controls;
using Avalonia.Layout;
using Avalonia.Media;

class CounterWindow : Window
{
    private readonly TextBlock _countLabel;
    private int _count;

    public CounterWindow()
    {
        Title = "Counter App (No XAML)";
        Width = 400;
        Height = 300;
        WindowStartupLocation = WindowStartupLocation.CenterScreen;

        _countLabel = new TextBlock
        {
            Text = "0",
            FontSize = 48,
            HorizontalAlignment = HorizontalAlignment.Center,
        };

        var incrementButton = new Button
        {
            Content = "Increment",
            FontSize = 18,
            HorizontalAlignment = HorizontalAlignment.Center,
            HorizontalContentAlignment = HorizontalAlignment.Center,
            Width = 160,
        };
        incrementButton.Click += OnIncrementClick;

        var decrementButton = new Button
        {
            Content = "Decrement",
            FontSize = 18,
            HorizontalAlignment = HorizontalAlignment.Center,
            HorizontalContentAlignment = HorizontalAlignment.Center,
            Width = 160,
        };
        decrementButton.Click += OnDecrementClick;

        var resetButton = new Button
        {
            Content = "Reset",
            FontSize = 14,
            HorizontalAlignment = HorizontalAlignment.Center,
        };
        resetButton.Click += (_, _) =>
        {
            _count = 0;
            _countLabel.Text = "0";
        };

        var buttonRow = new StackPanel
        {
            Orientation = Orientation.Horizontal,
            HorizontalAlignment = HorizontalAlignment.Center,
            Spacing = 12,
            Children = { incrementButton, decrementButton },
        };

        Content = new StackPanel
        {
            VerticalAlignment = VerticalAlignment.Center,
            Spacing = 20,
            Children =
            {
                _countLabel,
                buttonRow,
                resetButton,
            },
        };
    }

    private void OnIncrementClick(object? sender, Avalonia.Interactivity.RoutedEventArgs e)
    {
        _count++;
        _countLabel.Text = _count.ToString();
    }

    private void OnDecrementClick(object? sender, Avalonia.Interactivity.RoutedEventArgs e)
    {
        _count--;
        _countLabel.Text = _count.ToString();
    }
}
```

运行应用程序：

```bash
dotnet run
```

你应该会看到一个窗口，其中包含一个较大的计数值显示区，以及三个分别用于递增、递减和重置计数的按钮。

## 步骤 4：添加自定义样式

你可以通过添加编程式样式进一步改善界面外观。请在设置内容之前，于构造函数中加入如下样式代码：

```csharp title='CounterWindow.cs (add to constructor, before Content assignment)'
// 为此窗口中的所有按钮添加统一样式
Styles.Add(new Avalonia.Styling.Style(x => x.OfType<Button>())
{
    Setters =
    {
        new Avalonia.Styling.Setter(Button.PaddingProperty, new Thickness(16, 8)),
        new Avalonia.Styling.Setter(Button.CornerRadiusProperty, new CornerRadius(8)),
    }
});
```

添加到窗口 `Styles` 集合中的样式，会作用于该窗口内所有匹配的控件，这与在 XAML 的 `<Window.Styles>` 中声明样式的行为是一致的。

## 步骤 5：添加数据绑定

在更复杂的场景中，你可以使用代码形式的数据绑定，而不是直接手动更新控件属性。下面展示如何把控件绑定到一个视图模型：

```csharp title='CounterViewModel.cs'
using System.ComponentModel;
using System.Runtime.CompilerServices;

class CounterViewModel : INotifyPropertyChanged
{
    private int _count;

    public int Count
    {
        get => _count;
        set
        {
            if (_count != value)
            {
                _count = value;
                OnPropertyChanged();
                OnPropertyChanged(nameof(CountText));
            }
        }
    }

    public string CountText => Count.ToString();

    public void Increment() => Count++;
    public void Decrement() => Count--;
    public void Reset() => Count = 0;

    public event PropertyChangedEventHandler? PropertyChanged;

    protected void OnPropertyChanged([CallerMemberName] string? name = null)
        => PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(name));
}
```

接下来把标签绑定到视图模型。你可以使用基于字符串的绑定，也可以使用编译绑定。编译绑定具备类型安全、编译期校验以及完整的 IntelliSense 支持：

```csharp title='CounterWindow.cs (updated constructor, string-based)'
var viewModel = new CounterViewModel();
DataContext = viewModel;

_countLabel.Bind(TextBlock.TextProperty, new Avalonia.Data.Binding("CountText"));

incrementButton.Click += (_, _) => viewModel.Increment();
decrementButton.Click += (_, _) => viewModel.Decrement();
resetButton.Click += (_, _) => viewModel.Reset();
```

```csharp title='CounterWindow.cs (updated constructor, compiled binding)'
var viewModel = new CounterViewModel();
DataContext = viewModel;

_countLabel.Bind(TextBlock.TextProperty,
    CompiledBinding.Create<CounterViewModel, string>(
        expression: vm => vm.CountText));

incrementButton.Click += (_, _) => viewModel.Increment();
decrementButton.Click += (_, _) => viewModel.Decrement();
resetButton.Click += (_, _) => viewModel.Reset();
```

这样可以把 UI 逻辑与界面呈现分离开来，让你在纯代码场景下也能获得与 XAML 中相同的 MVVM 优势。编译绑定版本还能在构建阶段捕获属性名错误，而不是等到运行时才静默失败。

## 步骤 6：添加 Grid 布局

随着 UI 逐渐复杂，你可能会需要更精细的布局控制。下面这个示例展示了如何用 `Grid` 替换简单的 `StackPanel`：

```csharp
var grid = new Grid
{
    RowDefinitions = RowDefinitions.Parse("*,Auto,Auto"),
    ColumnDefinitions = ColumnDefinitions.Parse("*,*"),
    Margin = new Thickness(20),
};

// 计数显示区域跨越两列
Grid.SetColumnSpan(_countLabel, 2);
grid.Children.Add(_countLabel);

// 按钮放在第二行
Grid.SetRow(incrementButton, 1);
Grid.SetColumn(incrementButton, 0);
grid.Children.Add(incrementButton);

Grid.SetRow(decrementButton, 1);
Grid.SetColumn(decrementButton, 1);
grid.Children.Add(decrementButton);

// 重置按钮位于第三行，并跨越两列
Grid.SetRow(resetButton, 2);
Grid.SetColumnSpan(resetButton, 2);
grid.Children.Add(resetButton);

Content = grid;
```

## 步骤 7：添加自定义绘制（可选）

对于需要直接绘制内容的应用程序，你可以结合 `Canvas` 和形状控件来实现：

```csharp title='DrawingWindow.cs'
using System;
using Avalonia;
using Avalonia.Controls;
using Avalonia.Controls.Shapes;
using Avalonia.Media;

class DrawingWindow : Window
{
    private readonly Canvas _canvas;

    public DrawingWindow()
    {
        Title = "Code-Only Drawing";
        Width = 640;
        Height = 480;
        _canvas = new Canvas { Background = Brushes.Black };
        Content = _canvas;
        Resized += OnResized;
    }

    private void OnResized(object? sender, WindowResizedEventArgs e)
    {
        _canvas.Children.Clear();

        double cx = Width / 2;
        double cy = Height / 2;
        double radius = Math.Min(Width, Height) * 0.35;
        int segments = 80;

        for (int i = 0; i < segments; i++)
        {
            double angle1 = 2 * Math.PI * i / segments;
            double angle2 = 2 * Math.PI * (i + 1) / segments;

            _canvas.Children.Add(new Line
            {
                StartPoint = new Point(cx + radius * Math.Cos(angle1),
                                       cy + radius * Math.Sin(angle1)),
                EndPoint = new Point(cx + radius * Math.Cos(angle2),
                                     cy + radius * Math.Sin(angle2)),
                Stroke = Brushes.CornflowerBlue,
                StrokeThickness = 2,
            });
        }
    }
}
```

## 多窗口应用程序

对于包含多个窗口的应用程序，可以使用 `ClassicDesktopStyleApplicationLifetime` 来管理应用的退出行为：

```csharp title='Program.cs'
using Avalonia;
using Avalonia.Controls;
using Avalonia.Controls.ApplicationLifetimes;
using Avalonia.Themes.Fluent;

class Program
{
    public static void Main(string[] args)
    {
        var lifetime = new ClassicDesktopStyleApplicationLifetime
        {
            Args = args,
            ShutdownMode = ShutdownMode.OnLastWindowClose,
        };

        AppBuilder.Configure<Application>()
            .UsePlatformDetect()
            .AfterSetup(b => b.Instance?.Styles.Add(new FluentTheme()))
            .SetupWithLifetime(lifetime);

        lifetime.MainWindow = new CounterWindow();
        lifetime.Start(args);
    }
}
```

你可以在代码中的任意位置打开额外窗口：

```csharp
var secondWindow = new DrawingWindow();
secondWindow.Show();
```

在 `ShutdownMode.OnLastWindowClose` 模式下，应用会等到所有已打开窗口都关闭后才退出。

## 总结

本指南演示了：即使一行 XAML 都不写，你也依然可以构建出一个结构清晰、功能完整的 Avalonia 应用程序。核心模式包括：

| 关注点 | 纯代码方案 |
|---|---|
| 启动 | `AppBuilder.Configure<Application>().UsePlatformDetect().Start(delegate)` |
| 主题 | `app.Styles.Add(new FluentTheme())` |
| 控件 | 使用对象初始化器直接实例化 |
| 布局 | 向面板中添加子元素（`StackPanel`、`Grid`、`DockPanel`） |
| 事件 | 通过 `+=` 或 lambda 连接处理器 |
| 样式 | 创建 `Style` 对象并加入 `Styles` 集合 |
| 绑定 | `control.Bind(property, new ReflectionBinding(...))` 或 `CompiledBinding.Create(expression)` |
| 绘制 | 使用 `Canvas` 以及 `Line`、`Ellipse`、`Rectangle` 等图形 |
| 多窗口 | 使用 `ClassicDesktopStyleApplicationLifetime` 搭配 `ShutdownMode` |

:::tip
如果你想进一步了解这些模式背后的概念，请参阅 [Code-Only UI](/docs/fundamentals/coded-ui)。
:::

## 另请参阅

- [Code-Only UI](/docs/fundamentals/coded-ui)
- [Application lifetimes](/docs/fundamentals/application-lifetimes)
- [Binding from code](/docs/data-binding/binding-from-code)
- [Creating data templates in code](/docs/data-templates/creating-data-templates-in-code)
- [Threading](/docs/app-development/threading)
