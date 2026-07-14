---
id: button
title: Button
description: 一个可点击控件，可响应指针输入、触发 Click 事件，并可选地调用 ICommand。
doc-type: reference
---

import ButtonClickScreenshot from '/img/controls/buttons/button/button-click.gif';

[`Button`](/api/avalonia/controls/button) 控件会响应指针操作，并在按下时通过凹陷状态提供视觉反馈。从指针按下到指针释放的过程会被解释为一次点击，你可以通过 [`ClickMode`](/api/avalonia/controls/clickmode) 属性配置这一行为。

你可以在 code-behind 中订阅 `Click` 事件来处理点击，也可以将一个 `ICommand` 实例绑定到 `Command` 属性。有关命令绑定的说明，请参阅 [Adding interactivity](/docs/input-interaction/adding-interactivity)。

## 常用属性

| 属性 | 说明 |
| ------------------ | ------------------------------------------------------------------- |
| `ClickMode` | 描述按钮应如何响应点击。 |
| `Command` | 当按钮被点击时要调用的 `ICommand` 实例。 |
| `CommandParameter` | 调用命令时传递给它的参数。 |
| `Content` | 按钮内部显示的内容，可以是文本或任意控件。 |
| [`Flyout`](/api/avalonia/controls/flyout) | 点击按钮时打开的 `Flyout`。 |
| `IsPressed` | 按钮当前是否处于按下状态（只读）。 |
| `IsDefault` | 为 `true` 时，用户按 Enter 会激活该按钮。 |
| `IsCancel` | 为 `true` 时，用户按 Escape 会激活该按钮。 |

## 示例

此示例展示了一个简单按钮，以及一个 C# code-behind 点击事件处理器。

<XamlPreview>

```xml
<UserControl xmlns="https://github.com/avaloniaui"
             Padding="20">
  <Button Click="OnClick"
          HorizontalAlignment="Center"
          VerticalAlignment="Center">
    Press Me!
  </Button>
</UserControl>
```

```csharp
public partial class MainView : UserControl
{
    private int _clickCount = 0;

    public void OnClick(object sender, RoutedEventArgs args)
    {
        var btn = (Button)sender;
        btn.Content = $"Clicked: {++_clickCount} times";
    }
}
```

</XamlPreview>

## 绑定到命令

在 MVVM 中，更推荐的做法是将 `Command` 属性绑定到视图模型中的 `ICommand`：

```xml
<Button Content="Save" Command="{Binding SaveCommand}" />
```

```csharp
[RelayCommand]
private void Save()
{
    _repository.Save(CurrentItem);
}
```

### 带参数的命令

```xml
<Button Content="Delete"
        Command="{Binding DeleteCommand}"
        CommandParameter="{Binding SelectedItem}" />
```

### 使用 `CanExecute` 禁用按钮

当命令的 `CanExecute` 返回 `false` 时，按钮会自动禁用：

```csharp
[ObservableProperty]
[NotifyCanExecuteChangedFor(nameof(SaveCommand))]
private string _name = "";

[RelayCommand(CanExecute = nameof(CanSave))]
private void Save() { /* ... */ }

private bool CanSave() => !string.IsNullOrWhiteSpace(Name);
```

## 带图标的按钮

```xml
<Button>
    <StackPanel Orientation="Horizontal" Spacing="6">
        <PathIcon Data="{StaticResource save_regular}" Width="16" />
        <TextBlock Text="Save" VerticalAlignment="Center" />
    </StackPanel>
</Button>
```

## `ClickMode`

`ClickMode` 属性控制 `Click` 事件在何时触发：

| 值 | 说明 |
|---|---|
| `Release` | 在指针释放时触发 Click（默认）。 |
| `Press` | 在指针按下时触发 Click。 |
| `Hover` | 当指针进入按钮时触发 Click。 |

## `IsDefault` 和 `IsCancel`

你可以将按钮指定为窗口或对话框的默认操作按钮或取消操作按钮。当你将 `IsDefault` 设置为 `true` 时，用户按下 **Enter** 会激活该按钮；当你将 `IsCancel` 设置为 `true` 时，用户按下 **Escape** 会激活该按钮。

```xml
<StackPanel Orientation="Horizontal" Spacing="8">
  <Button Content="OK" IsDefault="True" Command="{Binding ConfirmCommand}" />
  <Button Content="Cancel" IsCancel="True" Command="{Binding CancelCommand}" />
</StackPanel>
```

## 带浮出层的按钮

你可以给按钮附加一个 `Flyout`，使得点击按钮时打开弹出层：

```xml
<Button Content="Options">
  <Button.Flyout>
    <MenuFlyout>
      <MenuItem Header="Cut" />
      <MenuItem Header="Copy" />
      <MenuItem Header="Paste" />
    </MenuFlyout>
  </Button.Flyout>
</Button>
```

## `Click` 与 `PointerPressed`

判断用户是否按下了按钮时，应始终使用 `Click` 事件，而不是 `PointerPressed`。`Click` 是 `Button` 专属的高级事件，而 `PointerPressed` 是一个低级输入事件，会被 `Button` 在内部处理（并将 `IsHandled` 设为 `true`）。由于该事件已被标记为已处理，因此你的应用无法像处理其他控件那样接收到来自 `Button` 的 `PointerPressed`。

完整的按钮事件列表请参阅 [Button events API reference](/api/avalonia/controls/button)。

## 键盘与无障碍

`Button` 默认可获得焦点，并参与 Tab 导航。当按钮获得键盘焦点后，用户可以通过按 **Space** 或 **Enter** 来激活它。屏幕阅读器会将按钮的 `Content` 作为其无障碍名称来朗读，因此请确保你提供了有意义的文本。如果按钮只包含图标，请设置 `AutomationProperties.Name` 附加属性，以便辅助技术识别它：

```xml
<Button Command="{Binding SaveCommand}"
        AutomationProperties.Name="Save">
  <PathIcon Data="{StaticResource save_regular}" Width="16" />
</Button>
```

## 样式设置

`Button` 提供了多个可在样式中使用的伪类：

| 伪类 | 应用时机 |
|----------------|---------------------------------------|
| `:pointerover` | 指针悬停在按钮上时。 |
| `:pressed` | 按钮被按下时。 |
| `:disabled` | 按钮的 `IsEnabled` 为 `false` 时。 |
| `:focus` | 按钮拥有键盘焦点时。 |

```xml
<Style Selector="Button.accent:pointerover">
  <Setter Property="Background" Value="{DynamicResource SystemAccentColorDark1}" />
</Style>
```

## 另请参阅

- [Button API reference](/api/avalonia/controls/button)
- [`Button.cs` source code on GitHub](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/Button.cs)
- [RepeatButton](/controls/input/buttons/repeatbutton)
- [ToggleButton](/controls/input/buttons/togglebutton)
- [SplitButton](/controls/input/buttons/splitbutton)
- [HyperlinkButton](/controls/input/buttons/hyperlinkbutton)
- [Adding interactivity](/docs/input-interaction/adding-interactivity)
