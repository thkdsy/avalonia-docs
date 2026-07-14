---
id: setting-up-the-headless-platform
title: 无头测试平台
---

无头平台可以在没有可见窗口的情况下运行 Avalonia，因此非常适合用于 CI/CD 环境中的自动化测试，以及没有显示设备的机器。它提供完整的 Avalonia 控件树、布局、样式和数据绑定能力，但会把真实的窗口系统与渲染后端替换为内存中的实现。

:::tip[不仅适用于测试]
无头平台在测试之外也很有用。如果你需要在没有可见窗口的情况下渲染控件（例如服务端生成图像、导出 PDF 或批处理），可以像 [视觉回归测试](#visual-regression-testing) 中展示的那样，将 `UseHeadlessDrawing = false` 以启用 Skia 渲染器。这样你就能在内存中获得完整的渲染流水线。另请参阅 [在 Docker 中以无头方式运行](/docs/deployment/docker#using-the-headless-platform-instead)。
:::

## 模拟用户输入

无头平台没有真实的输入设备，因此你需要通过 `Window` 上的扩展方法来模拟输入。这些方法触发的事件与真实输入产生的事件是相同的。

### 键盘输入

| 方法 | 说明 |
|---|---|
| `Window.KeyPress(Key, RawInputModifiers, PhysicalKey, string?)` | 模拟按键按下。 |
| `Window.KeyRelease(Key, RawInputModifiers, PhysicalKey, string?)` | 模拟按键释放。 |
| `Window.KeyPressQwerty(PhysicalKey, RawInputModifiers)` | 使用 QWERTY 布局映射模拟按键按下。 |
| `Window.KeyReleaseQwerty(PhysicalKey, RawInputModifiers)` | 使用 QWERTY 布局映射模拟按键释放。 |
| `Window.KeyTextInput(string)` | 模拟文本输入（独立于按下/释放事件）。适用于向 `TextBox` 及类似控件输入文本。 |

### 鼠标输入

| 方法 | 说明 |
|---|---|
| `Window.MouseDown(Point, MouseButton, RawInputModifiers)` | 在指定位置模拟鼠标按钮按下。 |
| `Window.MouseUp(Point, MouseButton, RawInputModifiers)` | 模拟鼠标按钮释放。 |
| `Window.MouseMove(Point, MouseButton, RawInputModifiers)` | 模拟鼠标移动。 |
| `Window.MouseWheel(Point, Vector, RawInputModifiers)` | 模拟鼠标滚轮滚动。 |

### 拖放

| 方法 | 说明 |
|---|---|
| `Window.DragDrop(Point, RawDragEventType, DataObject, DragDropEffects, RawInputModifiers)` | 模拟一次外部拖放操作（例如用户把文件从操作系统拖入你的应用）。 |

## 常见测试模式

### 测试按钮点击

```csharp
[AvaloniaTest]
public void Button_Click_Updates_ViewModel()
{
    var vm = new MyViewModel();
    var button = new Button
    {
        Command = vm.IncrementCommand,
        HorizontalAlignment = HorizontalAlignment.Stretch,
        VerticalAlignment = VerticalAlignment.Stretch
    };
    var window = new Window { Width = 100, Height = 100, Content = button };
    window.Show();

    window.MouseDown(new Point(50, 50), MouseButton.Left);
    window.MouseUp(new Point(50, 50), MouseButton.Left);

    Assert.Equal(1, vm.Count);
}
```

:::tip
你也可以直接通过 `button.RaiseEvent(new RoutedEventArgs(Button.ClickEvent))` 触发事件。这样做很方便，但不会执行已绑定的命令。若要通过键盘测试命令，请先调用 `button.Focus()`，再执行 `window.KeyReleaseQwerty(PhysicalKey.Space, RawInputModifiers.None)`。
:::

### 测试文本输入

```csharp
[AvaloniaTest]
public void TextBox_Accepts_Text_Input()
{
    var textBox = new TextBox();
    var window = new Window { Content = textBox };
    window.Show();

    textBox.Focus();
    window.KeyTextInput("Hello World");

    Assert.Equal("Hello World", textBox.Text);
}
```

### 测试数据绑定

```csharp
[AvaloniaTest]
public void TextBox_Binds_To_ViewModel()
{
    var vm = new MyViewModel { Name = "Alice" };
    var textBox = new TextBox
    {
        [!TextBox.TextProperty] = new Binding("Name")
    };
    var window = new Window
    {
        DataContext = vm,
        Content = textBox
    };
    window.Show();

    Assert.Equal("Alice", textBox.Text);

    // 模拟用户编辑文本
    textBox.Focus();
    textBox.Text = "Bob";

    Assert.Equal("Bob", vm.Name);
}
```

### 测试键盘快捷键

```csharp
[AvaloniaTest]
public void Ctrl_S_Triggers_Save()
{
    var saved = false;
    var window = new Window();
    window.KeyBindings.Add(new KeyBinding
    {
        Gesture = new KeyGesture(Key.S, KeyModifiers.Control),
        Command = ReactiveCommand.Create(() => saved = true)
    });
    window.Show();

    window.KeyPress(Key.S, RawInputModifiers.Control, PhysicalKey.S, "s");
    window.KeyRelease(Key.S, RawInputModifiers.Control, PhysicalKey.S, "s");

    Assert.True(saved);
}
```

### 测试已加载 XAML 的视图

你可以在无头测试中直接实例化实际视图：

```csharp
[AvaloniaTest]
public void MainView_Shows_Welcome_Message()
{
    var vm = new MainViewModel();
    var view = new MainView { DataContext = vm };
    var window = new Window { Content = view };
    window.Show();

    var textBlock = window.FindControl<TextBlock>("WelcomeText");
    Assert.Equal("Welcome to Avalonia!", textBlock?.Text);
}
```

## 刷新异步操作

Avalonia 中有些操作是异步的（例如窗口大小变化、布局过程、延迟调度的 dispatcher 任务）。如果你设置完属性后立刻做断言，变化可能还没有真正生效。

可以使用 `Dispatcher.UIThread.RunJobs()` 来刷新 dispatcher 队列：

```csharp
var window = new Window();
window.Show();

window.Width = 100;
window.Height = 100;

Dispatcher.UIThread.RunJobs();

Assert.Equal(new Size(100, 100), window.ClientSize);
```

你也可以强制渲染计时器执行一次 tick，这在测试动画或依赖渲染的行为时很有帮助：

```csharp
AvaloniaHeadlessPlatform.ForceRenderTimerTick();
```

:::tip
输入辅助方法和 `CaptureRenderedFrame` 内部已经调用了这些逻辑，因此在使用它们时通常不需要手动刷新。
:::

## 视觉回归测试

默认情况下，无头平台使用的是一个不会产生像素输出的伪绘图后端。你可以启用 Skia 渲染器来捕获渲染帧，并将它们与基线图像进行比较。

### 启用 Skia 渲染器

```csharp title="App.axaml.cs"
public static AppBuilder BuildAvaloniaApp() => AppBuilder.Configure<TestApplication>()
    .UseSkia()
    .UseHeadless(new AvaloniaHeadlessPlatformOptions
    {
        UseHeadlessDrawing = false
    });
```

### 捕获一帧

```csharp
var window = new Window
{
    Content = new TextBlock { Text = "Hello World" }
};
window.Show();

var frame = window.CaptureRenderedFrame();
frame.Save("output.png");
```

`CaptureRenderedFrame` 会返回一个 `WriteableBitmap`。你可以锁定它并读取像素数据，以便在内存中进行比较。

### 与基线图像比较

视觉回归测试的一种常见模式是：渲染某个控件，保存输出结果，然后逐像素与已知正确的参考图像进行比较：

```csharp
[AvaloniaTest]
public void Border_Renders_Correctly()
{
    var control = new Border
    {
        Width = 100,
        Height = 100,
        Background = Brushes.Blue,
        BorderBrush = Brushes.Black,
        BorderThickness = new Thickness(2),
        CornerRadius = new CornerRadius(8)
    };
    var window = new Window { Content = control };
    window.Show();

    var actual = window.CaptureRenderedFrame();

    // 与测试项目中保存的基线图像比较
    var expected = new Bitmap("expected/Border_Renders_Correctly.png");
    AssertImagesMatch(expected, actual, tolerance: 0.02);
}

private static void AssertImagesMatch(Bitmap expected, WriteableBitmap actual,
    double tolerance)
{
    // 实现像素比较逻辑，或使用图像比较库
}
```

:::tip
Avalonia 自身在内部的 [渲染测试套件](https://github.com/AvaloniaUI/Avalonia/tree/master/tests/Avalonia.RenderTests) 中也使用了这种方法。每个测试都会渲染一个控件，把输出保存为 PNG，并以可配置的误差容忍度与基线图像比较。
:::

## Testing view models without UI

View models that implement `INotifyPropertyChanged` or use `ReactiveUI` can be tested with plain unit tests without the headless platform. You only need the headless platform when your test involves Avalonia controls, layout, or input.

```csharp
// No [AvaloniaTest] needed, just a regular [Fact]
[Fact]
public void ViewModel_Increments_Count()
{
    var vm = new MainViewModel();

    vm.IncrementCommand.Execute(null);

    Assert.Equal(1, vm.Count);
}
```

## Manual setup

:::caution
This is an advanced usage scenario. For most cases, use the [XUnit](/docs/testing/headless-xunit) or [NUnit](/docs/testing/headless-nunit) integration, which handles setup automatically.
:::

### Install packages

You need two packages:
- [Avalonia.Headless](https://www.nuget.org/packages/Avalonia.Headless) (includes Avalonia)
- [Avalonia.Themes.Fluent](https://www.nuget.org/packages/Avalonia.Themes.Fluent) (headless controls need a theme)

:::tip
The headless platform does not require a specific theme. You can swap `FluentTheme` for any other theme.
:::

### Setup application

```xml title="App.axaml"
<Application xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             x:Class="Tests.App">
  <Application.Styles>
    <FluentTheme />
  </Application.Styles>
</Application>
```

```csharp title="App.axaml.cs"
using Avalonia;
using Avalonia.Headless;

public class App : Application
{
    public override void Initialize()
    {
        AvaloniaXamlLoader.Load(this);
    }
}
```

### Run a headless session

```csharp title="Program.cs"
using Avalonia.Controls;
using Avalonia.Headless;

using var session = HeadlessUnitTestSession.StartNew(typeof(App));

await session.Dispatch(() =>
{
    var textBox = new TextBox();
    var window = new Window { Content = textBox };
    window.Show();

    textBox.Focus();
    window.KeyTextInput("Hello World");

    if (textBox.Text != "Hello World")
        throw new Exception("Text input failed");
}, CancellationToken.None);
```

## See also

- [Headless Testing with XUnit](/docs/testing/headless-xunit): XUnit integration with `[AvaloniaFact]`.
- [Headless Testing with NUnit](/docs/testing/headless-nunit): NUnit integration with `[AvaloniaTest]`.
- [UI Testing with Appium](/docs/testing/ui-testing-with-appium): End-to-end testing with a real application window.
- [Avalonia's test suite](https://github.com/AvaloniaUI/Avalonia/tree/master/tests): How Avalonia tests itself.
