---
id: numericupdown
title: NumericUpDown
description: 一个允许用户通过微调按钮、键盘方向键或鼠标滚轮输入并调整数值的控件。
doc-type: reference
---

`NumericUpDown` 是一个可编辑的数值输入控件，带有向上和向下的微调按钮。输入中的非数字字符会被忽略。你可以通过点击微调按钮、按下键盘方向键或滚动鼠标滚轮来更改数值。

## 常用属性

你最常使用的通常是这些属性：

| 属性 | 类型 | 说明 |
|---|---|---|
| `Value` | `decimal?` | 获取或设置当前数值。 |
| `Increment` | `decimal` | 微调按钮、键盘方向键和鼠标滚轮使用的步进值。默认值为 `1`。 |
| `Minimum` | `decimal?` | 允许的最小值。 |
| `Maximum` | `decimal?` | 允许的最大值。 |
| `FormatString` | `string` | 应用于显示值的格式字符串。使用自定义步进值时尤其重要。 |
| `ButtonSpinnerLocation` | `Location` | 微调按钮的位置：`Left` 或 `Right`（默认）。 |
| `AllowSpin` | `bool` | 是否允许通过微调按钮、键盘和鼠标滚轮进行增减。默认值为 `true`。 |
| `ShowButtonSpinner` | `bool` | 微调按钮是否可见。默认值为 `true`。 |
| `InnerLeftContent` | `object` | 显示在输入区域左侧的内容（例如货币符号）。 |
| `InnerRightContent` | `object` | 显示在输入区域右侧的内容（例如单位标签）。 |

## 示例

这是一个不限制数值范围的基础示例：

<XamlPreview>

```xml
<StackPanel xmlns="https://github.com/avaloniaui"
            Margin="20">
  <TextBlock Margin="0 5">Number of items:</TextBlock>
  <NumericUpDown Value="10" />
</StackPanel>
```

</XamlPreview>

### 自定义步进和范围

`Value`、`Minimum`、`Maximum` 和 `Increment` 属性都使用可为空的十进制数，因此在需要时你可以定义自定义的小数范围和步进值。

:::info
当你使用自定义小数步进值和范围时，请记得设置 `FormatString` 属性。否则，显示出来的值可能不会保留你期望的精度。
:::

<XamlPreview>

```xml
<StackPanel xmlns="https://github.com/avaloniaui"
            Margin="20">
  <TextBlock Margin="0 5">Opacity:</TextBlock>
  <NumericUpDown Value="0.5" Increment="0.05"
      FormatString="0.00"
      Minimum="0" Maximum="1"/>
</StackPanel>
```

</XamlPreview>

### 隐藏微调按钮

如果你想要一个不带微调按钮的纯数字文本框，请将 `ShowButtonSpinner` 设置为 `False`。你还可以结合 `AllowSpin="False"` 一起使用，以同时禁用键盘和鼠标滚轮修改。

```xml
<NumericUpDown Value="42"
               ShowButtonSpinner="False"
               AllowSpin="False" />
```

### 添加前缀或后缀

使用 `InnerLeftContent` 和 `InnerRightContent` 可以在输入区域内部显示诸如货币符号或单位等标签。

```xml
<NumericUpDown Value="9.99" Increment="0.01" FormatString="0.00">
  <NumericUpDown.InnerLeftContent>
    <TextBlock Text="$" VerticalAlignment="Center" />
  </NumericUpDown.InnerLeftContent>
</NumericUpDown>
```

### 绑定到视图模型

你可以将 `Value`、`Minimum` 和 `Maximum` 绑定到视图模型上的属性。由于 `Value` 是可空的 `decimal`，因此你的视图模型属性类型也应与之一致。

```xml
<NumericUpDown Value="{Binding Quantity}"
               Minimum="1" Maximum="{Binding MaxQuantity}" />
```

```csharp
[ObservableProperty]
private decimal? _quantity = 1;

[ObservableProperty]
private decimal? _maxQuantity = 100;
```

## 实用说明

- 如果用户输入的值超出了 `Minimum`/`Maximum` 范围，控件会在失去焦点时将该值限制到最近的边界。
- 将 `Value` 设置为 `null` 会清空输入内容。当你希望表示“未设置”状态时，这会很有用。
- `FormatString` 属性接受标准的 .NET 数字格式字符串。例如，`"C2"` 会将值显示为带两位小数的货币格式，而 `"P0"` 会将其显示为不带小数的百分比格式。

## 另请参阅

- [Slider](/controls/input/selectors/slider)
- [TextBox](/controls/input/text-input/textbox)
- [Binding to Controls](/docs/data-binding/binding-to-controls)
- [NumericUpDown API Reference](/api/avalonia/controls/numericupdown)
- [`NumericUpDown.cs` Source on GitHub](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/NumericUpDown/NumericUpDown.cs)
