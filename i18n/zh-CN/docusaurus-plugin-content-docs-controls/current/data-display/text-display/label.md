---
id: label
title: Label
description: 一个显示文本，并在被点击或通过访问键激活时将焦点转移到目标输入元素的控件。
doc-type: reference
---

[`Label`](/api/avalonia/controls/label) 控件用于显示文本，并将焦点转移到指定的目标控件。当你点击标签，或按下它的访问键并同时按住 Alt 键时，焦点会移动到 `Target` 属性指定的控件。这使得 `Label` 在构建无障碍表单时特别有用，因为每个输入项都可以拥有对应的文本标签。

## 常用属性

| 属性 | 类型 | 说明 |
|---|---|---|
| `Content` | `object` | 要在标签中显示的内容。你可以将它设置为字符串，或绑定到视图模型属性。 |
| `Target` | `IInputElement` | 当标签被点击或其访问键被按下时接收焦点的目标控件。请将其设置为目标控件的 `x:Name`。 |

## 基本用法

若要将 `Label` 与输入控件关联，请将 `Target` 属性设置为该输入控件的名称。当用户点击标签文本时，焦点会移动到目标控件：

```xml
<StackPanel Spacing="4">
    <Label Target="nameBox" Content="Name" />
    <TextBox x:Name="nameBox" />
</StackPanel>
```

## 访问键

你可以通过在 `Content` 字符串中的某个字符前放置下划线（`_`）来定义键盘快捷键（访问键）。支持字母、数字和重音字符。当用户按下 Alt 加该字符时，焦点会移动到目标控件：

```xml
<StackPanel Spacing="4">
    <Label Target="nameBox" Content="_Name" />
    <TextBox x:Name="nameBox" />

    <Label Target="emailBox" Content="_Email" />
    <TextBox x:Name="emailBox" />
</StackPanel>
```

在上面的示例中，按下 Alt+N 会聚焦到 Name `TextBox`，按下 Alt+E 会聚焦到 Email `TextBox`。

:::tip
请在每个视图中选择唯一的访问键字符，以避免冲突。带下划线的字符会在用户按下 Alt 键时显示出来，作为提示。
:::

## 绑定内容

你可以将 `Content` 属性绑定到视图模型属性，这样标签文本就可以动态更新：

```xml
<Label Target="quantityBox" Content="{Binding QuantityLabel}" />
<NumericUpDown x:Name="quantityBox" Value="{Binding Quantity}" />
```

```csharp
public string QuantityLabel => "Quantity";
```

## 构建无障碍表单

`Label` 控件是构建无障碍表单的重要基础组件。通过为每个输入控件配对一个带有 `Target` 的 `Label`，你就为用户提供了两种聚焦字段的方式：点击标签，或按下它的访问键。下面的示例展示了一个完整的表单布局：

```xml
<Grid ColumnDefinitions="Auto,*" RowDefinitions="Auto,Auto,Auto" Margin="8">
    <Label Grid.Row="0" Grid.Column="0" Target="firstNameBox" Content="_First name" Margin="0,0,8,4" />
    <TextBox Grid.Row="0" Grid.Column="1" x:Name="firstNameBox" />

    <Label Grid.Row="1" Grid.Column="0" Target="lastNameBox" Content="_Last name" Margin="0,0,8,4" />
    <TextBox Grid.Row="1" Grid.Column="1" x:Name="lastNameBox" />

    <Label Grid.Row="2" Grid.Column="0" Target="ageBox" Content="_Age" Margin="0,0,8,4" />
    <NumericUpDown Grid.Row="2" Grid.Column="1" x:Name="ageBox" />
</Grid>
```

## Label 与 TextBlock 的区别

| 功能 | `Label` | `TextBlock` |
|---|---|---|
| 焦点转移 | 是（通过 `Target`） | 否 |
| 访问键支持 | 是 | 否 |
| 富文本格式 | 否 | 是（通过 `Inlines`） |
| 典型用途 | 表单字段标签 | 显示文本、段落 |

当你在表单中需要无障碍支持和键盘导航时，请使用 `Label`。而 `TextBlock` 更适合不需要焦点转移的一般文本显示场景。

## 另请参阅

- [TextBlock](/controls/data-display/text-display/textblock)
- [SelectableTextBlock](/controls/data-display/text-display/selectabletextblock)
- [Label API reference](/api/avalonia/controls/label)
- [`Label.cs` source code on GitHub](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/Label.cs)
