---
id: checkbox
title: CheckBox
description: 一个通过复选标记让用户切换布尔值的控件，并可选支持表示不确定值的三态模式。
doc-type: reference
---

import CheckBoxTwoStateScreenshot from '/img/reference/controls/checkbox/checkbox-two-state.gif';
import CheckBoxThreeStateScreenshot from '/img/reference/controls/checkbox/checkbox-three-state.gif';

[`CheckBox`](/api/avalonia/controls/checkbox) 控件用于表示一个布尔值：true 用勾选标记表示，false 用空框表示。你也可以启用三态模式，在这种模式下，null 表示“未知”，并显示为一个带阴影的框。

点击控件时，值会按以下顺序切换：选中、未选中、未知（如果启用了三态）。

## 常用属性

你最常使用的通常是这些属性：

| 属性 | 类型 | 说明 |
| -------------- | ------- | --------------------------------------------------------------------------- |
| `IsChecked` | `bool?` | 获取或设置选中状态。`true` 表示选中，`false` 表示未选中，`null` 表示不确定。 |
| `IsThreeState` | `bool` | 为 `true` 时，控件会在选中、未选中和不确定三种状态之间循环切换。 |
| `Content` | `object` | 显示在复选标记旁边的标签内容。 |
| `Command` | `ICommand` | 当用户切换复选框时调用的命令。 |

## 两态示例

在默认的两态模式下，`IsChecked` 会在 `true` 和 `false` 之间切换：

<XamlPreview>

```xml
<UserControl xmlns="https://github.com/avaloniaui">
    <StackPanel Margin="20">
        <CheckBox>Not checked by default</CheckBox>
        <CheckBox IsChecked="True">Checked by default</CheckBox>
    </StackPanel>
</UserControl>
```

</XamlPreview>

<Image light={CheckBoxTwoStateScreenshot} alt="Two-state CheckBox" position="center" maxWidth={400} cornerRadius="true"/>

## 三态示例

当你将 `IsThreeState` 设置为 `true` 时，控件会增加一个不确定状态。你可以将 `IsChecked` 设置为 `{x:Null}`，让它从不确定状态开始：

<XamlPreview>

```xml
<UserControl xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <StackPanel Margin="20">
        <CheckBox IsThreeState="True" IsChecked="False">Not checked by default</CheckBox>
        <CheckBox IsThreeState="True" IsChecked="True">Checked by default</CheckBox>
        <CheckBox IsThreeState="True" IsChecked="{x:Null}">Unknown by default</CheckBox>
    </StackPanel>
</UserControl>
```

</XamlPreview>

<Image light={CheckBoxThreeStateScreenshot} alt="Three-state CheckBox" position="center" maxWidth={400} cornerRadius="true"/>

当将三态 `CheckBox` 绑定到视图模型时，请使用可空的 `bool?` 属性，以便正确往返表示不确定状态。

## 绑定到视图模型

将 `IsChecked` 绑定到视图模型中的 `bool` 属性。下面的示例使用了 MVVM Toolkit 的源生成器：

```csharp
public partial class SettingsViewModel : ObservableObject
{
    [ObservableProperty]
    private bool _autoSave = true;

    [ObservableProperty]
    private bool _showLineNumbers;
}
```

```xml
<StackPanel Spacing="8">
    <CheckBox IsChecked="{Binding AutoSave}" Content="Auto-save on exit" />
    <CheckBox IsChecked="{Binding ShowLineNumbers}" Content="Show line numbers" />
</StackPanel>
```

如果你需要在值变化时作出响应，可以订阅 `PropertyChanged` 事件，或使用诸如 `OnAutoSaveChanged` 这样的分部方法。

## 从集合生成 CheckBox 列表

你可以通过将 `ItemsControl` 与项模板中的 `CheckBox` 结合起来，创建一个可勾选项目列表：

```xml
<ItemsControl ItemsSource="{Binding Features}">
    <ItemsControl.ItemTemplate>
        <DataTemplate>
            <CheckBox IsChecked="{Binding IsEnabled}" Content="{Binding Name}" />
        </DataTemplate>
    </ItemsControl.ItemTemplate>
</ItemsControl>
```

## “全选”模式

三态 `CheckBox` 非常适合作为“全选”控件。当只有部分子项被选中时，将其设置为不确定状态；当用户点击它时，再同步更新所有子项：

```csharp
[ObservableProperty]
private bool? _selectAll = false;

partial void OnSelectAllChanged(bool? value)
{
    if (value.HasValue)
    {
        foreach (var item in Items)
            item.IsSelected = value.Value;
    }
}
```

```xml
<StackPanel Spacing="4">
    <CheckBox IsThreeState="True"
              IsChecked="{Binding SelectAll}"
              Content="Select all" />
    <ItemsControl ItemsSource="{Binding Items}" Margin="24,0,0,0">
        <ItemsControl.ItemTemplate>
            <DataTemplate>
                <CheckBox IsChecked="{Binding IsSelected}" Content="{Binding Name}" />
            </DataTemplate>
        </ItemsControl.ItemTemplate>
    </ItemsControl>
</StackPanel>
```

## 另请参阅

- [ToggleSwitch](/controls/input/selectors/toggleswitch)
- [RadioButton](/controls/input/buttons/radiobutton)
- [CheckBox API reference](/api/avalonia/controls/checkbox)
- [`CheckBox.cs` source code on GitHub](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/CheckBox.cs)
