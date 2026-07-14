---
id: how-to-bind-can-execute
title: 如何绑定 CanExecute
description: 通过绑定命令的 CanExecute 方法，自动启用或禁用按钮。
doc-type: how-to
---

import BindCanExecuteScreenshot from '/img/guides/data/bind-canexecute.gif';

## 概览

一个可触发操作的控件是否处于启用状态，是用户体验设计中“功能可感知性”的关键部分。把当前无法执行的命令禁用掉，能够增强用户信心。例如，如果某个按钮或菜单项由于应用程序状态不正确而无法执行，你更应该把它显示为不可用，而不是等用户点击后再弹出错误。

本指南将介绍如何把 [`Button`](/api/avalonia/controls/button) 绑定到一个带有 `CanExecute` 逻辑的命令上，让控件自动启用或禁用。这里采用 MVVM 模式，以保证视图与视图模型之间保持清晰分离。

## 前置条件

- 一个具备 MVVM 结构的基础 Avalonia 应用程序（包含视图和对应的视图模型）。
- 了解 [data binding](/docs/data-binding/introduction-to-data-binding) 和 `ICommand` 的基本用法。

## 示例

在这个示例中，按钮只有在消息不为空时才能点击。一旦动作执行完成，消息会被重置为空字符串，因此按钮又会重新变为禁用状态。

### 定义视图

`TextBox` 绑定到 `Message` 属性，`Button` 的 `Command` 绑定到 `ExampleCommand`。Avalonia 会根据命令 `CanExecute` 方法返回的值，自动设置按钮的 `IsEnabled` 状态。

```xml title='MainWindow.axaml'
<StackPanel Margin="20">
  <TextBox Margin="0 5" Text="{Binding Message}"
           PlaceholderText="Add a message to enable the button"/>
  <Button Command="{Binding ExampleCommand}">
    Run the example
  </Button>
  <TextBlock Margin="0 5" Text="{Binding Output}" />
</StackPanel>
```

### 创建一个简单的 `RelayCommand`

如果你没有使用 CommunityToolkit.Mvvm 或 ReactiveUI 之类的框架，也可以自己实现一个轻量级的 `RelayCommand`。下面这个类封装了一个执行用的 `Action`，以及一个可选的 `Func<bool>` 作为是否可执行检查。

```csharp title='RelayCommand.cs'
using System;
using System.Windows.Input;

namespace AvaloniaGuides.ViewModels
{
    public class RelayCommand : ICommand
    {
        private readonly Action _execute;
        private readonly Func<bool>? _canExecute;

        public RelayCommand(Action execute, Func<bool>? canExecute = null)
        {
            _execute = execute ?? throw new ArgumentNullException(nameof(execute));
            _canExecute = canExecute;
        }

        public event EventHandler? CanExecuteChanged;

        public bool CanExecute(object? parameter) => _canExecute?.Invoke() ?? true;

        public void Execute(object? parameter) => _execute();

        public void RaiseCanExecuteChanged() =>
            CanExecuteChanged?.Invoke(this, EventArgs.Empty);
    }
}
```

### 实现视图模型

在构造函数中，命令通过两个参数创建：一个是真正要执行的动作，另一个是决定命令是否可以执行的函数。每当 `Message` 变化时，属性 setter 都会调用 `RaiseCanExecuteChanged`，从而让绑定系统重新评估按钮的启用状态。

```csharp title='MainWindowViewModel.cs'
using System.ComponentModel;
using System.Runtime.CompilerServices;

namespace AvaloniaGuides.ViewModels
{
    public class MainWindowViewModel : INotifyPropertyChanged
    {
        private string _message = string.Empty;
        private string _output = "Waiting...";

        public event PropertyChangedEventHandler? PropertyChanged;

        public string Message
        {
            get => _message;
            set
            {
                if (_message != value)
                {
                    _message = value;
                    OnPropertyChanged();
                    ExampleCommand.RaiseCanExecuteChanged();
                }
            }
        }

        public string Output
        {
            get => _output;
            set
            {
                if (_output != value)
                {
                    _output = value;
                    OnPropertyChanged();
                }
            }
        }

        public RelayCommand ExampleCommand { get; }

        public MainWindowViewModel()
        {
            ExampleCommand = new RelayCommand(
                PerformAction,
                () => !string.IsNullOrWhiteSpace(Message));
        }

        private void PerformAction()
        {
            Output = $"The action was called. {Message}";
            Message = string.Empty;
        }

        protected void OnPropertyChanged([CallerMemberName] string? name = null)
        {
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(name));
        }
    }
}
```

<Image light={BindCanExecuteScreenshot} alt="App showing a button enabled and disabled based on CanExecute binding" position="center" maxWidth={400} cornerRadius="true"/>

## 工作原理

1. 当用户在 `TextBox` 中输入内容时，`Message` 属性的 setter 会被触发。
2. setter 调用 `ExampleCommand.RaiseCanExecuteChanged()`，从而触发 `CanExecuteChanged` 事件。
3. Avalonia 会响应该事件，并调用命令的 `CanExecute`。如果返回 `false`，绑定的 `Button` 就会自动禁用。
4. 当用户清空文本（或者执行动作后把 `Message` 重置为空字符串）时，`CanExecute` 会返回 `false`，按钮便再次禁用。

## 提示与边界情况

- **始终调用 `RaiseCanExecuteChanged`**（或等价通知），并且要在所有 `CanExecute` 所依赖属性的 setter 中调用它。如果你忘了，按钮状态就可能过期，直到其他事件触发重新计算。
- **多个依赖项。** 如果 `CanExecute` 会检查多个属性，那么要在这些属性各自的 setter 中都调用 `RaiseCanExecuteChanged`。
- **线程安全。** `CanExecuteChanged` 应当在 UI 线程上触发。如果你是在后台线程更新属性，请先使用 `Dispatcher.UIThread.Post` 把变更切回 UI 线程。
- **使用 CommunityToolkit.Mvvm。** `[RelayCommand(CanExecute = nameof(CanRun))]` 源生成器可以去掉上面展示的大量样板代码。生成出来的命令在你调用 `NotifyCanExecuteChanged()` 时，会自动触发 `CanExecuteChanged`。
- **使用 ReactiveUI。** `ReactiveCommand.Create` 接收一个 `canExecute` observable。只要该 observable 发出新值，命令就会自动重新评估，因此你无需手动触发事件。
- **`CommandParameter` 绑定。** 当你通过绑定传入 `CommandParameter` 时，该参数值会被传递到 `CanExecute(object? parameter)`。请确保你的实现能处理初始化布局阶段的 `null` 参数，因为此时绑定系统可能尚未解析出最终参数值。
- **菜单项。** 同样的模式也适用于 `MenuItem`。只要把 `MenuItem.Command` 绑定到你的命令上，当 `CanExecute` 返回 `false` 时，菜单项也会自动变灰。

## 另请参阅

- [Binding to commands](/docs/data-binding/binding-to-commands)
- [Commanding](/docs/input-interaction/commanding)
- [Data binding overview](/docs/data-binding/introduction-to-data-binding)
