---
id: commanding
title: 命令系统
---

命令系统用于将用户操作（按钮点击、菜单选择、键盘快捷键）连接到视图模型中的逻辑。Avalonia 使用标准的 .NET `ICommand` 接口，从而实现 UI 与业务逻辑之间的清晰分离。

## 命令系统如何工作

支持命令的控件（例如 `Button` 和 `MenuItem`）都具有 `Command` 属性。当用户激活控件时，它会调用 `ICommand.Execute`。控件还会监听 `ICommand.CanExecute`，并在命令不可执行时自动将自己禁用。

```xml
<Button Content="Save" Command="{Binding SaveCommand}" />
```

当 `SaveCommand.CanExecute()` 返回 `false` 时，按钮会显示为禁用状态，且无法点击。

## ICommand interface

`System.Windows.Input.ICommand` 接口定义了以下成员：

```csharp
public interface ICommand
{
    bool CanExecute(object? parameter);
    void Execute(object? parameter);
    event EventHandler? CanExecuteChanged;
}
```

| 成员 | 用途 |
|---|---|
| `CanExecute` | 返回命令当前是否可以执行。控件会调用它来决定自身是否启用。 |
| `Execute` | 执行命令动作。控件在用户激活时调用它。 |
| `CanExecuteChanged` | 当 `CanExecute` 的返回值可能发生变化时触发。控件会监听此事件并重新查询 `CanExecute`。 |

## 使用 RelayCommand（CommunityToolkit.Mvvm）

创建命令最常见的方式，是使用 CommunityToolkit.Mvvm 包中的 `[RelayCommand]` 特性：

```csharp
using CommunityToolkit.Mvvm.ComponentModel;
using CommunityToolkit.Mvvm.Input;

public partial class MainViewModel : ObservableObject
{
    [ObservableProperty]
    private string _name = "";

    [RelayCommand]
    private void Save()
    {
        // 保存逻辑写在这里
    }

    [RelayCommand(CanExecute = nameof(CanDelete))]
    private void Delete()
    {
        // 删除逻辑写在这里
    }

    private bool CanDelete() => !string.IsNullOrEmpty(Name);
}
```

源生成器会自动创建 `SaveCommand` 和 `DeleteCommand` 属性。每当触发 `CanExecuteChanged` 时，`DeleteCommand` 都会重新评估 `CanDelete()`。

```xml
<StackPanel Spacing="8">
    <TextBox Text="{Binding Name}" />
    <Button Content="Save" Command="{Binding SaveCommand}" />
    <Button Content="Delete" Command="{Binding DeleteCommand}" />
</StackPanel>
```

### 异步命令

`[RelayCommand]` 也支持异步方法。生成出的命令能够处理 `Task` 返回类型，并提供自动的忙碌状态跟踪：

```csharp
[RelayCommand]
private async Task LoadDataAsync()
{
    IsLoading = true;
    try
    {
        var data = await _dataService.GetDataAsync();
        Items = new ObservableCollection<Item>(data);
    }
    finally
    {
        IsLoading = false;
    }
}
```

当 `LoadDataAsync` 正在运行时，`LoadDataCommand.IsRunning` 会为 `true`。你可以将其绑定到进度指示器上。

### 通知 CanExecute 变化

当某些属性变化会影响 `CanExecute` 时，可以使用 `[NotifyCanExecuteChangedFor]`：

```csharp
[ObservableProperty]
[NotifyCanExecuteChangedFor(nameof(DeleteCommand))]
private string _name = "";
```

这会告诉源生成器：每当 `Name` 发生变化时，都调用 `DeleteCommand.NotifyCanExecuteChanged()`，从而让绑定到该命令的控件重新判断命令是否可执行。

## CommandParameter

`CommandParameter` 属性会将数据传递给命令的 `Execute` 和 `CanExecute` 方法：

```xml
<Button Content="Open"
        Command="{Binding OpenCommand}"
        CommandParameter="{Binding SelectedItem}" />
```

```csharp
[RelayCommand]
private void Open(object? parameter)
{
    if (parameter is Item item)
    {
        // 打开该项目
    }
}
```

在 CommunityToolkit.Mvvm 中，也可以使用强类型参数：

```csharp
[RelayCommand]
private void Open(Item item)
{
    // 源生成器会将 OpenCommand 生成为 RelayCommand<Item>
}
```

```xml
<Button Content="Open"
        Command="{Binding OpenCommand}"
        CommandParameter="{Binding SelectedItem}" />
```

## 手动实现 ICommand

在不使用源生成器的场景下，可以手动创建命令：

```csharp
public class RelayCommand : ICommand
{
    private readonly Action _execute;
    private readonly Func<bool>? _canExecute;

    public RelayCommand(Action execute, Func<bool>? canExecute = null)
    {
        _execute = execute;
        _canExecute = canExecute;
    }

    public bool CanExecute(object? parameter) => _canExecute?.Invoke() ?? true;

    public void Execute(object? parameter) => _execute();

    public event EventHandler? CanExecuteChanged;

    public void RaiseCanExecuteChanged()
        => CanExecuteChanged?.Invoke(this, EventArgs.Empty);
}
```

```csharp
public class MainViewModel
{
    public ICommand SaveCommand { get; }

    public MainViewModel()
    {
        SaveCommand = new RelayCommand(
            execute: () => { /* save logic */ },
            canExecute: () => IsModified);
    }
}
```

## 支持命令的控件

| 控件 | 命令属性 | 触发时机 |
|---|---|---|
| `Button` | `Command` | 点击时 |
| `MenuItem` | `Command` | 点击时 |
| [`KeyBinding`](/api/avalonia/input/keybinding) | `Command` | 按下按键手势时 |
| `ToggleButton` | `Command` | 切换状态时 |
| `SplitButton` | `Command` | 主按钮点击时 |

## 键盘快捷键与命令

可以使用 `KeyBinding` 将命令绑定到键盘快捷键：

```xml
<Window.KeyBindings>
    <KeyBinding Gesture="Ctrl+S" Command="{Binding SaveCommand}" />
    <KeyBinding Gesture="Ctrl+Z" Command="{Binding UndoCommand}" />
    <KeyBinding Gesture="Delete" Command="{Binding DeleteCommand}" />
</Window.KeyBindings>
```

`KeyBinding` 会评估 `CanExecute`，只有在按下对应手势且命令可执行时才会触发命令。

## HotKey 附加属性

对于控件来说，`HotKey` 附加属性提供了更简洁的语法：

```xml
<Button Content="_Save"
        Command="{Binding SaveCommand}"
        HotKey="Ctrl+S" />
```

即使按钮本身没有焦点，`HotKey` 也能触发该按钮的命令。

## 另请参阅

- [如何绑定 CanExecute](/docs/data-binding/how-to-bind-can-execute)：CanExecute 的绑定模式。
- [键盘与热键](/docs/input-interaction/keyboard-and-hotkeys)：按键绑定与键盘输入。
- [MVVM 模式](/docs/fundamentals/the-mvvm-pattern)：分离 UI 与逻辑的架构模式。
