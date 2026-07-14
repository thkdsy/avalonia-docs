---
id: adding-interactivity
title: 添加交互性
---

用户界面最基本的职责之一，就是与用户进行交互。在 Avalonia 中，你可以通过事件和命令为应用添加交互能力。本指南将通过简单示例介绍事件和命令。

## 处理事件

Avalonia 中的事件提供了响应用户交互和控件特定操作的方式。你可以按照以下步骤来处理事件：

1. **实现事件处理器**：在 [code-behind](/docs/fundamentals/code-behind) 中编写一个事件处理器，当事件被触发时执行。事件处理器中应包含你希望在事件发生时运行的逻辑。

```csharp title='MainWindow.axaml.cs'
public partial class MainWindow : Window
{
    public MainWindow()
    {
        InitializeComponent();
    }

    // highlight-start
    private void HandleButtonClick(object sender, RoutedEventArgs e)
    {
        // 事件处理逻辑写在这里
    }
    // highlight-end
}
```

2. **订阅事件**：确定你希望在控件上处理的事件。Avalonia 中的大多数控件都会公开各种事件，例如 `Click` 或 `SelectionChanged`。你可以在 XAML 中添加与事件同名的属性，并将其值设为事件处理器方法名，以完成订阅。

```xml title='MainWindow.axaml'
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        x:Class="AvaloniaApplication1.Views.MainWindow">
  // highlight-next-line
  <Button Name="myButton" Content="点我" Click="HandleButtonClick" />
</Window>
```

上面的示例将名为 `HandleButtonClick` 的处理器绑定到了 `Button` 的 `Click` 事件上。

## 使用命令

Avalonia 中的命令提供了一种更高层次的用户交互处理方式，它将用户操作与实现逻辑解耦。事件通常定义在控件的 code-behind 中，而命令则通常绑定到 [数据上下文](/docs/data-binding/data-context) 上的属性或方法。

:::info
凡是提供 `Command` 属性的控件都支持命令。命令通常会在控件的主要交互行为发生时触发，例如按钮点击。
:::

使用命令最简单的方式，是直接绑定到对象数据上下文中的一个方法。

1. **在视图模型中添加方法**：在视图模型里定义一个方法来处理命令。

```csharp title='C#'
    public class MainWindowViewModel
    {
        // highlight-start
        public bool HandleButtonClick()
        {
            // 事件处理逻辑写在这里
        }
        // highlight-end
    }
    ```

2. **绑定该方法**：将该方法与触发它的控件关联起来。

    ```xml title='XAML'
    <Button Content="点我" Command="{Binding HandleButtonClick}" />
    ```

## 配合 CommunityToolkit.Mvvm 使用命令

推荐的命令写法是使用 CommunityToolkit.Mvvm 提供的 `[RelayCommand]` 特性：

```csharp
using CommunityToolkit.Mvvm.ComponentModel;
using CommunityToolkit.Mvvm.Input;

public partial class MainViewModel : ObservableObject
{
    [RelayCommand]
    private void Save()
    {
        // 会生成一个 SaveCommand 属性
    }

    [RelayCommand]
    private async Task LoadAsync()
    {
        // 会生成一个带自动忙碌状态的 LoadCommand
    }
}
```

```xml
<Button Content="Save" Command="{Binding SaveCommand}" />
<Button Content="Load" Command="{Binding LoadCommand}" />
```

## 带参数的命令

使用 `CommandParameter` 可以向命令传递数据：

```xml
<Button Content="Delete"
        Command="{Binding DeleteCommand}"
        CommandParameter="{Binding SelectedItem}" />
```

```csharp
[RelayCommand]
private void Delete(Item item)
{
    Items.Remove(item);
}
```

## 事件与命令的对比

| 特性 | 事件 | 命令 |
|---|---|---|
| 定义位置 | Code-behind | 视图模型 |
| 可测试性 | 较难（需要 UI） | 容易（普通 C# 方法） |
| 最适合 | UI 专属行为（拖拽、调整大小） | 应用逻辑（保存、导航、删除） |
| MVVM 模式 | 不优先推荐 | 推荐 |

对于 UI 专属行为（如动画、视觉反馈），使用事件更合适。对于需要可测试、并与视图解耦的应用逻辑，则应使用命令。

## 另请参阅

- [绑定到命令](/docs/data-binding/binding-to-commands)：完整的命令绑定参考。
- [命令系统](/docs/input-interaction/commanding)：`ICommand` 接口详解。
- [键盘与热键](/docs/input-interaction/keyboard-and-hotkeys)：命令的按键绑定方式。
