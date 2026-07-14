---
title: TextBox
description: 一个支持占位提示、密码掩码、验证和内嵌内容的单行或多行文本输入控件。
doc-type: reference
---

import TextBoxEntryScreenshot from '/img/controls/textbox/textbox-entry.gif';

[`TextBox`](/api/avalonia/controls/textbox) 提供一个用于键盘输入的区域。你可以将它用于用户名这类单行字段，也可以启用多行编辑以输入备注、评论等较长内容。

## 常用属性

你最常使用的通常是这些属性：

| 属性 | 类型 | 说明 |
|---|---|---|
| `Text` | `string` | 输入框中的当前文本。 |
| `PlaceholderText` | `string` | 输入为空时显示的淡化提示文本，也常被称为水印。 |
| `PlaceholderForeground` | `IBrush` | 用于绘制占位提示文本的画刷。 |
| `PasswordChar` | `char` | 隐藏用户输入的字符，并用指定字符替代显示。 |
| `RevealPassword` | `bool` | 当为 `true` 时，显示真实密码文本而不是掩码字符。 |
| `AcceptsReturn` | `bool` | 允许用户输入换行符，从而使输入框支持多行。 |
| `AcceptsTab` | `bool` | 允许用户插入 Tab 字符，而不是移动焦点。 |
| `TextWrapping` | `TextWrapping` | 定义如何处理水平方向上的文本溢出。可选值为：`NoWrap`、`Wrap`、`WrapWithOverflow`。 |
| `MaxLength` | `int` | 限制用户可输入的字符数量。`0` 表示不限制。 |
| `IsReadOnly` | `bool` | 当为 `true` 时，用户可以选中和复制文本，但不能编辑。 |
| `TextAlignment` | `TextAlignment` | 文本的水平对齐方式：`Left`、`Center`、`Right`。 |
| `InnerLeftContent` | `object` | 显示在 `TextBox` 左侧内部的内容（例如图标或标签）。 |
| `InnerRightContent` | `object` | 显示在 `TextBox` 右侧内部的内容（例如按钮或状态指示器）。 |
| `MinLines` | `int` | 最少显示的行数。当 `AcceptsReturn` 为 `true` 时，`TextBox` 会至少显示这么多行。 |
| `CaretBlinkInterval` | `TimeSpan` | 光标闪烁的时间间隔。设置为 `TimeSpan.Zero` 可禁用闪烁。 |

## 示例

此示例包含一个基础单行文本框、一个密码框，以及一个支持自动换行的多行文本框：

```xml
<StackPanel Margin="20">
  <TextBlock Margin="0 5">Name:</TextBlock>
  <TextBox PlaceholderText="Enter your name"/>
  <TextBlock Margin="0 5">Password:</TextBlock>
  <TextBox PasswordChar="*" PlaceholderText="Enter your password"/>
  <TextBlock Margin="0 15 0 5">Notes:</TextBlock>
  <TextBox Height="100" AcceptsReturn="True" TextWrapping="Wrap"/>
</StackPanel>
```

<Image light={TextBoxEntryScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

## 水印（占位提示文本）

设置 `PlaceholderText` 可在 `TextBox` 为空时显示提示文本。你还可以使用 `PlaceholderForeground` 自定义提示文本颜色：

```xml
<TextBox PlaceholderText="e.g. jane@example.com"
         PlaceholderForeground="Gray" />
```

当用户开始输入时，占位提示会自动消失；当文本被清空后，它会重新出现。

## 多行输入

若要接受多行输入，请将 `AcceptsReturn` 设为 `True`。再配合 `TextWrapping="Wrap"` 使用，就能让长行自动换行，而不是水平滚动。使用 `MinLines` 可保证最小可见高度：

```xml
<TextBox AcceptsReturn="True"
         TextWrapping="Wrap"
         MinLines="4"
         PlaceholderText="Enter your comments here..." />
```

如果你同时将 `AcceptsTab` 设为 `True`，按下 Tab 键时将插入制表符，而不是把焦点移动到下一个控件。

## 限制输入长度

使用 `MaxLength` 可限制用户可输入的字符数。这对于邮政编码、用户名等已知最大长度的字段非常有用：

```xml
<TextBox MaxLength="50" PlaceholderText="Username (max 50 characters)" />
```

值为 `0`（默认值）表示不施加限制。

## 验证

你可以通过视图模型上的数据注解特性来验证 `TextBox` 输入。当你实现 `INotifyDataErrorInfo` 时，Avalonia 的绑定系统会自动显示验证错误。例如，使用 CommunityToolkit.Mvvm 的源生成器：

```csharp
using System.ComponentModel.DataAnnotations;
using CommunityToolkit.Mvvm.ComponentModel;

public partial class MyViewModel : ObservableValidator
{
    [ObservableProperty]
    [NotifyDataErrorInfo]
    [Required(ErrorMessage = "Email is required.")]
    [EmailAddress(ErrorMessage = "Enter a valid email address.")]
    private string _email = "";
}
```

```xml
<TextBox Text="{Binding Email}" PlaceholderText="Email address" />
```

验证失败时，`TextBox` 默认会显示错误边框和工具提示。你可以在样式中通过控件的 `:error` 伪类来自定义这种外观。

## 视图模型绑定

使用双向模式绑定 `Text`（这是 `TextBox.Text` 的默认行为）：

```xml
<TextBox Text="{Binding Username}" PlaceholderText="Enter username" />
```

```csharp
[ObservableProperty]
private string _username = "";
```

## 带内嵌内容的输入框

你可以通过 `InnerLeftContent` 和 `InnerRightContent` 在 `TextBox` 内部加入图标或按钮：

```xml
<TextBox PlaceholderText="Search...">
    <TextBox.InnerRightContent>
        <Button Content="✕" Command="{Binding ClearSearchCommand}"
                Background="Transparent" BorderThickness="0" Padding="4" />
    </TextBox.InnerRightContent>
</TextBox>
```

## 另请参阅

- [TextBox API 参考](/api/avalonia/controls/textbox)
- [`TextBox.cs` GitHub 源代码](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/TextBox.cs)
- [MaskedTextBox](/controls/input/text-input/maskedtextbox)
- [AutoCompleteBox](/controls/input/text-input/autocompletebox)
