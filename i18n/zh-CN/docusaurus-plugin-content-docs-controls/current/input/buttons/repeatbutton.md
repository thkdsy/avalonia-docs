---
id: repeatbutton
title: RepeatButton
description: 当用户持续按住按钮时，会重复触发点击事件的按钮。
doc-type: reference
---

`RepeatButton` 是一种在按钮被持续按下时，会定期生成点击事件的控件。

## 常用属性

你最常使用的通常是这些属性：

| 属性 | 说明 |
| -------- | ---------------------------------------------------------------------------------------- |
| `Delay`    | 开始生成重复点击前要等待的时间（毫秒）。默认值为 300。 |
| `Interval` | 两次点击生成之间的时间间隔（毫秒）。默认值为 100。 |

## 示例

此示例展示了一个使用默认延迟和间隔生成点击事件的重复按钮。

<XamlPreview>

```xml
<UserControl xmlns="https://github.com/avaloniaui"
             Padding="20">
  <RepeatButton Click="OnClick"
                HorizontalAlignment="Center"
                VerticalAlignment="Center">
    Press and hold down
  </RepeatButton>
</UserControl>
```

```csharp
public partial class MainView : UserControl
{
    private int _clickCount = 0;

    public void OnClick(object sender, RoutedEventArgs args)
    {
        var btn = (RepeatButton)sender;
        btn.Content = $"Clicked: {++_clickCount} times";
    }
}
```

</XamlPreview>

## 自定义延迟和间隔

你可以直接在 XAML 中配置 `Delay` 和 `Interval` 属性，以控制重复点击开始的速度以及触发频率。下面的示例会在第一次重复前等待 500 毫秒，之后每 50 毫秒触发一次：

```xml
<RepeatButton Delay="500" Interval="50" Click="OnClick">
    Fast repeat after half-second delay
</RepeatButton>
```

## 常见使用场景

在任何需要用户按住按钮时持续执行某个操作的场景中，`RepeatButton` 都非常有用。常见示例包括音量控制、滚动按钮、数字步进器和缩放控件。在这些场景下，重复行为使用户无需反复点击即可进行增量调整。

## 另请参阅

- [Button](/controls/input/buttons/button)
- [ButtonSpinner](/controls/input/buttons/buttonspinner)
- [RepeatButton API reference](/api/avalonia/controls/repeatbutton)
- [`RepeatButton.cs` source code on GitHub](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/RepeatButton.cs)
