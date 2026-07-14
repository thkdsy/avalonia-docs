---
id: binding-to-commands
title: 绑定到命令
description: 在 MVVM 模式中，将 UI 控件绑定到 ICommand 实现，以处理用户操作。
doc-type: explanation
---

命令提供了一种清晰的方式，用来把用户交互（例如按钮点击、菜单选择、键盘快捷键）连接到视图模型中的逻辑。本页介绍如何使用 `ICommand` 将控件绑定到命令。

## 基础命令绑定

任何实现了 `ICommandSource` 的控件（例如 `Button`、`MenuItem` 和 `ToggleButton`）都具有 `Command` 属性：

```xml
<Button Content="Save" Command="{Binding SaveCommand}" />
```

视图模型会公开一个命令属性：

```csharp
public partial class MainViewModel : ObservableObject
{
    [RelayCommand]
    private void Save()
    {
        // Save logic
    }
}
```

CommunityToolkit.Mvvm 提供的 `[RelayCommand]` 特性会生成一个 `SaveCommand` 属性，其类型为 `IRelayCommand`。其命名规则是在方法名后追加 `Command`。

## CommandParameter

你可以使用 `CommandParameter` 把数据从 UI 传递给命令：

```xml
<ListBox ItemsSource="{Binding Items}">
    <ListBox.ItemTemplate>
        <DataTemplate>
            <StackPanel Orientation="Horizontal" Spacing="8">
                <TextBlock Text="{Binding Name}" />
                <Button Content="Delete"
                        Command="{Binding $parent[ListBox].((vm:MainViewModel)DataContext).DeleteCommand}"
                        CommandParameter="{Binding}" />
            </StackPanel>
        </DataTemplate>
    </ListBox.ItemTemplate>
</ListBox>
```

视图模型会接收到这个参数：

```csharp
[RelayCommand]
private void Delete(Item item)
{
    Items.Remove(item);
}
```

## CanExecute

命令可以根据条件启用或禁用绑定的控件。当 `CanExecute` 返回 `false` 时，控件会自动变为禁用状态。

### 使用 CommunityToolkit.Mvvm

可以通过 `[RelayCommand(CanExecute = ...)]` 特性来指定一个 `CanExecute` 方法：

```csharp
[ObservableProperty]
[NotifyCanExecuteChangedFor(nameof(SaveCommand))]
private string _name = "";

[RelayCommand(CanExecute = nameof(CanSave))]
private void Save()
{
    // Save logic
}

private bool CanSave() => !string.IsNullOrWhiteSpace(Name);
```

`[NotifyCanExecuteChangedFor]` 特性可以确保每当 `Name` 变化时，`SaveCommand` 都会重新评估它的 `CanExecute`。

### 手动实现 ICommand

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

    public event EventHandler? CanExecuteChanged;

    public bool CanExecute(object? parameter) => _canExecute?.Invoke() ?? true;
    public void Execute(object? parameter) => _execute();

    public void RaiseCanExecuteChanged()
        => CanExecuteChanged?.Invoke(this, EventArgs.Empty);
}
```

## 异步命令

耗时操作应使用异步命令，以避免阻塞 UI 线程：

```csharp
[RelayCommand]
private async Task SaveAsync(CancellationToken token)
{
    await _repository.SaveAsync(token);
}
```

CommunityToolkit.Mvvm 会生成一个类型为 `IAsyncRelayCommand` 的 `SaveCommand`，它具备以下特点：
- 异步执行方法
- 在命令执行期间禁用控件（防止重复点击）
- 提供 `CancellationToken` 参数以支持取消操作
- 暴露 `IsRunning` 属性，便于显示进度指示器

```xml
<Button Content="Save" Command="{Binding SaveCommand}" />
<ProgressBar IsVisible="{Binding SaveCommand.IsRunning}" IsIndeterminate="True" />
```

## 键盘快捷键

你可以使用 `HotKey` 或 `KeyBinding` 将命令绑定到键盘快捷键：

```xml
<!-- HotKey on a Button -->
<Button Content="_Save"
        Command="{Binding SaveCommand}"
        HotKey="Ctrl+S" />

<!-- KeyBinding on a window or control -->
<Window.KeyBindings>
    <KeyBinding Gesture="Ctrl+Z" Command="{Binding UndoCommand}" />
    <KeyBinding Gesture="Ctrl+Shift+Z" Command="{Binding RedoCommand}" />
</Window.KeyBindings>
```

`Content="_Save"` 中的下划线会创建一个访问键（在 Windows/Linux 上通常是 Alt+S）。

## 支持命令的控件

| 控件 | 命令属性 | 触发时机 |
|---|---|---|
| `Button` | `Command` | 点击 |
| `MenuItem` | `Command` | 点击 |
| `ToggleButton` | `Command` | 切换 |
| `RadioButton` | `Command` | 选择变化 |
| `HyperlinkButton` | `Command` | 点击 |
| `SplitButton` | `Command` | 主按钮点击 |

## 从不同的 DataContext 绑定命令

当命令位于父级视图模型上，而绑定发生在模板内部时：

```xml
<!-- 使用 $parent 访问祖先元素的 DataContext -->
<Button Command="{Binding $parent[Window].DataContext.DeleteCommand}"
        CommandParameter="{Binding}" />

<!-- 使用已命名的祖先元素 -->
<Button Command="{Binding #Root.((vm:MainViewModel)DataContext).DeleteCommand}"
        CommandParameter="{Binding}" />
```

在编译绑定中，需要使用 `((Type)expression)` 语法显式地对 DataContext 进行类型转换。

## 另请参阅

- [Commanding](/docs/input-interaction/commanding): Full commanding reference including manual ICommand patterns.
- [Keyboard and Hotkeys](/docs/input-interaction/keyboard-and-hotkeys): Hotkey and keybinding setup.
- [Data Binding Syntax](/docs/data-binding/data-binding-syntax): Binding paths, modes, and converters.
