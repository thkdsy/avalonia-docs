---
id: buttonspinner
title: ButtonSpinner
description: 一个带有递增和递减微调按钮的内容控件，用于在值之间循环切换。
doc-type: reference
---

[`ButtonSpinner`](/api/avalonia/controls/buttonspinner) 提供了一个带有向上和向下微调按钮的控件。它的内容非常灵活，但很多行为逻辑需要你自己编写。

## 何时使用 `ButtonSpinner`

当你需要完全控制微调行为时，可以使用 `ButtonSpinner`，例如在一组非数值项中循环切换，或在每次递增、递减时应用自定义逻辑。由于该控件不包含内置值处理机制，因此你需要自行根据 spin 事件更新显示内容。

如果你需要带有内置验证和格式化功能的标准数值输入，请考虑改用 [`NumericUpDown`](/controls/input/selectors/numericupdown)。

## 常用属性

你最常使用的通常是这些属性：

| 属性 | 说明 |
|---|---|
| `ButtonSpinnerLocation` | 微调按钮的位置：`Left` 或 `Right`（默认）。 |
| `ValidSpinDirection` | 限制微调方向：`Increase`、`Decrease` 或 `None`。 |
| `AllowSpin` | 是否启用微调。默认值为 `true`。 |

## 示例

<XamlPreview>

```xml
<UserControl xmlns="https://github.com/avaloniaui"
             Padding="20">
  <ButtonSpinner Spin="OnSpin"
                 HorizontalAlignment="Center"
                 VerticalAlignment="Center">
    Press the spinner
  </ButtonSpinner>
</UserControl>
```

```csharp
public partial class MainView : UserControl
{
    private int _currentValue = 0;

    public void OnSpin(object sender, SpinEventArgs args)
    {
        if (args.Direction == SpinDirection.Increase)
            _currentValue++;
        else
            _currentValue--;

        var btn = (ButtonSpinner)sender;
        btn.Content = $"Value: {_currentValue}";
    }
}
```

</XamlPreview>

## 与 MVVM 一起使用

你可以将 `Spin` 事件绑定到视图模型中的命令。将显示内容放到 `ButtonSpinner` 内部，并把它绑定到视图模型上的属性即可。

```xml
<ButtonSpinner Spin="{Binding SpinCommand}">
    <TextBlock Text="{Binding CurrentValue}" />
</ButtonSpinner>
```

```csharp
public partial class MyViewModel : ObservableObject
{
    [ObservableProperty]
    private string _currentValue = "0";

    [RelayCommand]
    private void Spin(SpinEventArgs args)
    {
        var value = int.Parse(CurrentValue);

        if (args.Direction == SpinDirection.Increase)
            value++;
        else
            value--;

        CurrentValue = value.ToString();
    }
}
```

## 另请参阅

- [ButtonSpinner API reference](/api/avalonia/controls/buttonspinner)
- [`ButtonSpinner.cs` source code on GitHub](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/ButtonSpinner.cs)
- [`NumericUpDown`](/controls/input/selectors/numericupdown)
- [`Button`](/controls/input/buttons/button)
