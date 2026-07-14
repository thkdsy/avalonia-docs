---
id: code-behind
title: Code-behind
description: 使用 code-behind 文件访问控件、设置属性，并处理来自 XAML 的事件。
doc-type: explanation
---

import VsSolutionExplorerScreenshot from '/img/concepts/core-concepts/code-behind/vs-solution-explorer.png';

除了 XAML 文件之外，大多数 Avalonia 控件还会有一个 _code-behind_ 文件，通常使用 C# 编写。按照约定，这个 code-behind 文件的扩展名是 `.axaml.cs`，并且在 IDE 中通常会显示在对应 XAML 文件的下方层级中。

例如，在 Visual Studio 的解决方案资源管理器中，你会看到 `MainWindow.axaml` 文件以及它对应的 code-behind 文件 `MainWindow.axaml.cs`：

<Image light={VsSolutionExplorerScreenshot} alt="Visual Studio solution explorer showing a XAML file with its nested code-behind file" position="center" maxWidth={400} cornerRadius="true"/>

你的 code-behind 文件中会包含一个 `partial` 类，它与 XAML 文件使用相同的名称。`partial` 关键字非常重要，因为它允许 Avalonia 的构建工具生成一个配套文件，用来连接命名控件并调用 XAML 加载器。例如：

```csharp title='MainWindow.axaml.cs'
using Avalonia.Controls;

namespace AvaloniaApplication1.Views
{
    public partial class MainWindow : Window
    {
        public MainWindow()
        {
            InitializeComponent();
        }
    }
}
```

请注意，这里的类名与 XAML 文件名一致，并且同样出现在窗口元素的 `x:Class` 属性中。`x:Class` 中填写的完整限定名必须包含命名空间。

```xml title='MainWindow.axaml'
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        // highlight-next-line
        x:Class="AvaloniaApplication1.Views.MainWindow">
  ...
</Window>
```

:::tip
如果你在代码中修改了类名或它所在的命名空间，请确保 `x:Class` 属性始终保持一致。否则会导致构建错误或运行时错误。
:::

当你首次创建 code-behind 文件时，其中通常只有一个调用 `InitializeComponent()` 方法的构造函数。这个调用是运行时加载对应 XAML 所必需的。如果把它移除，你的 UI 将不会被渲染出来。

## 定位控件

在使用 code-behind 时，你经常需要访问在 XAML 中定义的控件。

要做到这一点，你需要在 XAML 中通过 `Name`（或 `x:Name`）属性为目标控件命名。随后 Avalonia 的构建工具会在你的 partial 类中生成一个强类型字段，这样你就可以直接引用该控件。

下面是一个为 `Button` 命名的 XAML 示例：

```xml title='MainWindow.axaml'
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        x:Class="AvaloniaApplication5.MainWindow">
  // highlight-next-line
  <Button Name="greetingButton">Hello World</Button>
</Window>
```

现在你可以在 code-behind 中通过自动生成的 `greetingButton` 字段访问这个按钮：

```csharp title='MainWindow.axaml.cs'
using Avalonia.Controls;

namespace AvaloniaApplication1.Views
{
    public partial class MainWindow : Window
    {
        public MainWindow()
        {
            InitializeComponent();
            // highlight-next-line
            greetingButton.Content = "Goodbye Cruel World!";
        }
    }
}
```

:::tip
由于这个字段是在构建阶段生成的，所以在项目编译前，你的 IDE 可能会显示警告。完成一次构建后，这个警告通常就会消失。
:::

## 设置属性

一旦你在 code-behind 中拿到了某个控件的引用，就可以读取或设置它的任意属性。例如，你可以修改按钮的 `Background` 属性：

```csharp title='C#'
greetingButton.Background = Brushes.Blue;
```

你也可以读取属性值。当你需要先检查控件当前状态，再决定执行什么操作时，这会非常有用：

```csharp title='C#'
if (greetingButton.IsVisible)
{
    greetingButton.Content = "I'm visible!";
}
```

## 处理事件

大多数交互式应用程序都需要响应用户操作，例如点击、按键或指针移动。在使用 code-behind 模式时，你需要在 code-behind 文件中编写事件处理方法，并通过 XAML 中的事件属性来引用它们。

例如，要处理按钮点击事件，你可以在 XAML 中添加一个 `Click` 属性，让它指向 code-behind 中的方法：

```xml title='MainWindow.axaml'
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        x:Class="AvaloniaApplication4.MainWindow">
  <Button Click="GreetingButtonClickHandler">Hello World</Button>
</Window>
```

```csharp title='MainWindow.axaml.cs'
public partial class MainWindow : Window
{
    public MainWindow()
    {
        InitializeComponent();
    }

    public void GreetingButtonClickHandler(object sender, RoutedEventArgs e)
    {
        // code here.
    }
}
```

其中 `sender` 参数表示触发该事件的控件，`RoutedEventArgs` 参数则携带了事件如何产生以及如何在视觉树中传播的相关信息。

你也可以在代码中而不是在 XAML 中附加事件处理器。当你需要动态添加或移除处理器时，这种方式会更灵活：

```csharp title='C#'
greetingButton.Click += GreetingButtonClickHandler;
```

:::info
有关事件路由的更多信息，请参阅 [Routed events](/docs/input-interaction/routed-events)。
:::

## 何时使用 code-behind，何时使用 MVVM

code-behind 很适合小型应用、原型项目，或者某些特定于视图的逻辑，例如动画和焦点管理。对于更大的应用程序，建议考虑使用 MVVM 模式，它会把 UI 逻辑拆分到更容易测试和维护的视图模型中。你也可以将两种方式结合起来使用：用 MVVM 处理数据和业务逻辑，同时把视图相关代码保留在 code-behind 中。

## 另请参阅

- [Avalonia XAML](/docs/fundamentals/avalonia-xaml)
- [Code-only UI](/docs/fundamentals/coded-ui)
- [The MVVM pattern](/docs/fundamentals/the-mvvm-pattern)
- [UI composition](/docs/fundamentals/ui-composition)
- [Routed events](/docs/input-interaction/routed-events)
