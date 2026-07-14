---
id: coded-ui
title: 纯代码 UI
description: 不依赖 XAML 文件，完全使用 C# 或 F# 构建 Avalonia 应用程序。
doc-type: explanation
---

Avalonia 并不强制要求使用 XAML。你可以仅使用 C#、F# 或任意 .NET 语言来构建完整的应用程序。凡是能用 XAML 表达的控件、布局、样式、绑定和动画，在代码中通常都有对应的 API。

之所以可行，是因为 Avalonia XAML 最终会被编译为 IL（中间语言），而 C# 和其他 .NET 语言最终也会编译到同一种 IL。XAML 只是描述对象图的一种方式。如果你更愿意直接用 C#、F# 或其他 .NET 语言去构造同样的对象图，这完全没有问题。

## 何时选择纯代码方式

在 XAML 和纯代码之间做选择，往往取决于个人偏好。两者在运行时可以得到相同的结果，而且你也可以在同一个应用程序中自由混用它们。

不过，你仍然需要了解其中的实际权衡。Avalonia 受到 WPF 的启发，而绝大多数资料、教程、社区解答以及开发体验都默认以 XAML 为中心。如果你选择纯代码方式：

- 你在网上找到的大多数示例都会是 XAML 写法，因此你需要自行把这些片段转换成你所使用的语言。
- 不使用 XAML 的开发者群体相对更小，因此寻找与纯代码模式相关的帮助时，可能要花更多精力。
- 像 XAML 预览器和设计时数据这类工具，都是围绕 XAML 工作流构建的。

纯代码开发当然完全可行，但在借助现有生态中的知识和资源时，它并不是阻力最小的那条路。

## 启动一个纯代码应用程序

一个纯代码的 Avalonia 应用程序完全不需要 `.axaml` 文件。最简单的方式是使用 `AppBuilder` 并配合手动启动委托：

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
        app.Styles.Add(new FluentTheme());

        var window = new Window
        {
            Title = "Hello from Code",
            Width = 400,
            Height = 300,
            Content = new TextBlock
            {
                Text = "No XAML here.",
                FontSize = 24,
                HorizontalAlignment = Avalonia.Layout.HorizontalAlignment.Center,
                VerticalAlignment = Avalonia.Layout.VerticalAlignment.Center,
            }
        };

        window.Show();
        app.Run(window);
    }
}
```

项目文件也同样可以非常精简：

```xml title='MyApp.csproj'
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

这就是你所需要的全部内容。不需要 `App.axaml`，不需要 `MainWindow.axaml`，也不需要生成代码。

### 使用应用程序生命周期

对于那些需要更精细窗口管理的应用程序（例如多窗口应用，并且需要在最后一个窗口关闭时退出），你可以使用 `ClassicDesktopStyleApplicationLifetime`，而不是简单的 `Start` 委托：

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
            .AfterSetup(builder => builder.Instance?.Styles.Add(new FluentTheme()))
            .SetupWithLifetime(lifetime);

        lifetime.MainWindow = new MainAppWindow();
        lifetime.Start(args);
    }
}
```

:::info
有关生命周期选项的完整说明，请参阅 [Application lifetimes](/docs/fundamentals/application-lifetimes)。
:::

## 创建控件

每个 Avalonia 控件都可以直接在代码中实例化并配置。C# 的对象初始化器与 XAML 属性的写法天然对应：

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

<Tabs>
<TabItem value="csharp" label="C# Code" default>

```csharp
var button = new Button
{
    Content = "Click Me",
    FontSize = 18,
    HorizontalAlignment = HorizontalAlignment.Center,
    Background = Brushes.SteelBlue,
    Foreground = Brushes.White,
};
```

</TabItem>
<TabItem value="xaml" label="XAML Equivalent">

```xml
<Button Content="Click Me"
        FontSize="18"
        HorizontalAlignment="Center"
        Background="SteelBlue"
        Foreground="White" />
```

</TabItem>
</Tabs>

## 构建布局

代码中的布局与 XAML 一样，遵循相同的父子模型。你先创建一个布局面板，再向其中添加子元素，最后把它赋值给窗口或其他控件的内容属性。

```csharp
var stack = new StackPanel
{
    Spacing = 12,
    Margin = new Thickness(20),
};

stack.Children.Add(new TextBlock { Text = "Name:", FontSize = 16 });
stack.Children.Add(new TextBox { PlaceholderText = "Enter your name" });
stack.Children.Add(new Button { Content = "Submit" });

window.Content = stack;
```

对于网格布局，你需要先定义行和列，再通过附加属性来设置子元素的位置：

```csharp
var grid = new Grid
{
    RowDefinitions = RowDefinitions.Parse("Auto,*,Auto"),
    ColumnDefinitions = ColumnDefinitions.Parse("200,*"),
};

var header = new TextBlock { Text = "Header", FontSize = 24 };
Grid.SetColumnSpan(header, 2);
grid.Children.Add(header);

var sidebar = new ListBox();
Grid.SetRow(sidebar, 1);
grid.Children.Add(sidebar);

var content = new TextBlock { Text = "Main content area" };
Grid.SetRow(content, 1);
Grid.SetColumn(content, 1);
grid.Children.Add(content);
```

## 处理事件

你可以使用标准的 C# 事件处理器或 lambda 表达式来连接事件：

```csharp
var count = 0;
var label = new TextBlock { Text = "Clicks: 0" };

var button = new Button { Content = "Click Me" };
button.Click += (sender, args) =>
{
    count++;
    label.Text = $"Clicks: {count}";
};
```

For routed events with more complex handling:

```csharp
button.AddHandler(Button.ClickEvent, (sender, args) =>
{
    // Handle the event
    args.Handled = true;
}, Avalonia.Interactivity.RoutingStrategies.Bubble);
```

## Applying styles

You can create styles programmatically and add them at any level of the control hierarchy:

```csharp
var style = new Style(x => x.OfType<Button>())
{
    Setters =
    {
        new Setter(Button.FontSizeProperty, 16.0),
        new Setter(Button.PaddingProperty, new Thickness(12, 8)),
        new Setter(Button.BackgroundProperty, Brushes.DarkSlateBlue),
        new Setter(Button.ForegroundProperty, Brushes.White),
    }
};

window.Styles.Add(style);
```

## Data binding from code

Bind control properties to data sources without XAML. The simplest approach uses string-based binding paths:

```csharp
var textBox = new TextBox();
var label = new TextBlock();

// Bind the label text to the textbox text
label.Bind(TextBlock.TextProperty,
    new ReflectionBinding("Text") { Source = textBox });
```

### Compiled bindings

For type-safe bindings with compile-time validation and full IntelliSense support, use `CompiledBinding.Create`. This accepts a LINQ expression instead of a string path, so the compiler catches property name errors before runtime:

```csharp
// Bind to a view model property with type safety
var binding = CompiledBinding.Create<MyViewModel, string>(
    expression: vm => vm.Title
);
textBlock.Bind(TextBlock.TextProperty, binding);

// With an explicit source and two-way mode
var binding = CompiledBinding.Create(
    source: viewModel,
    expression: vm => vm.Title,
    mode: BindingMode.TwoWay
);
textBox.Bind(TextBox.TextProperty, binding);
```

Compiled bindings support property access, nested properties, indexers, type casts, logical negation, and `AvaloniaProperty` access. They also perform better than reflection-based string bindings.

### Reactive patterns

You can also use `GetObservable` and `GetBindingObservable` for reactive patterns:

```csharp
textBox.GetObservable(TextBox.TextProperty).Subscribe(newText =>
{
    // React to text changes
});
```

:::info
For complete coverage of binding from code, see [Binding from code](/docs/data-binding/binding-from-code).
:::

## Custom drawing

For applications that need to render graphics directly (visualizations, games, simulations), use `Canvas` with shape controls or implement custom rendering:

```csharp
var canvas = new Canvas { Background = Brushes.Black };

// Add shapes to the canvas
var circle = new Ellipse
{
    Width = 100,
    Height = 100,
    Fill = Brushes.CornflowerBlue,
};
Canvas.SetLeft(circle, 150);
Canvas.SetTop(circle, 100);
canvas.Children.Add(circle);

// Draw lines
var line = new Line
{
    StartPoint = new Point(0, 0),
    EndPoint = new Point(200, 200),
    Stroke = Brushes.White,
    StrokeThickness = 2,
};
canvas.Children.Add(line);
```

## Threading considerations

When updating the UI from a background thread, use `Dispatcher.UIThread`:

```csharp
var label = new TextBlock { Text = "Waiting..." };

_ = Task.Run(async () =>
{
    // Do work on a background thread
    await Task.Delay(2000);

    // Update the UI on the UI thread
    Dispatcher.UIThread.Post(() =>
    {
        label.Text = "Done!";
    });
});
```

:::info
For complete threading guidance, see [Threading](/docs/app-development/threading).
:::

## F# and Avalonia.FuncUI

F# is particularly well suited to code-only UI development. As an expression-first functional language, F# has strong type inference, lightweight syntax for nested structures, and native support for building domain-specific languages through features like computation expressions, pipelines, and discriminated unions.

Where C# builds UI through object construction and property assignment, F# can express the same trees as pure data composition. The result tends to feel more like describing a UI than assembling one.

[Avalonia.FuncUI](https://github.com/fsprojects/Avalonia.FuncUI) is a community library that brings an Elm-inspired, fully functional architecture to Avalonia for F# developers. It provides:

- A declarative domain-specific language (DSL) for building views as immutable descriptions.
- An Elm/MVU (Model-View-Update) architecture with immutable state and message passing.
- Full access to every Avalonia control through a composable F# API.

```fsharp title='F# with Avalonia.FuncUI'
let view (state: State) (dispatch: Msg -> unit) =
    DockPanel.create [
        DockPanel.children [
            Button.create [
                Button.dock Dock.Bottom
                Button.onClick (fun _ -> dispatch Increment)
                Button.content "Click me"
            ]
            TextBlock.create [
                TextBlock.dock Dock.Top
                TextBlock.fontSize 48.0
                TextBlock.text (string state.Count)
            ]
        ]
    ]
```

### C# compared to F# for coded UI

Both languages are fully capable of building code-only Avalonia applications. The choice between them comes down to ergonomics and preference.

**F# strengths for coded UI:**

- Almost everything is an expression, which makes composing UI trees feel natural and direct.
- Computation expressions and pipelines allow you to build APIs that feel like a mini-language for UI.
- Immutability and algebraic data types pair well with reactive, message-passing architectures.
- Type inference keeps generic-heavy composition code clean and readable.

**C# strengths for coded UI:**

- Larger ecosystem of learning resources, libraries, and community support.
- Object initializer syntax works well for straightforward control configuration.
- More familiar to the majority of .NET developers.
- Full access to all Avalonia APIs without any wrapper layer.

C# is not fundamentally limited for coded UI, but its object-oriented heritage means deeply nested, compositional UI trees can become verbose. F# was designed for exactly that shape of code. If you are open to learning F#, Avalonia.FuncUI offers what is arguably the most ergonomic code-only experience in the .NET ecosystem.

If you prefer C#, a builder-style API with careful design can reduce ceremony significantly. The code-only patterns shown throughout this page work well for most applications.

## See also

- [Avalonia XAML](/docs/fundamentals/avalonia-xaml)
- [Code-behind](/docs/fundamentals/code-behind)
- [Application lifetimes](/docs/fundamentals/application-lifetimes)
- [Binding from code](/docs/data-binding/binding-from-code)
- [Creating data templates in code](/docs/data-templates/creating-data-templates-in-code)
- [How To: Build a Complete App Without XAML](/docs/how-to/coded-ui-how-to)
