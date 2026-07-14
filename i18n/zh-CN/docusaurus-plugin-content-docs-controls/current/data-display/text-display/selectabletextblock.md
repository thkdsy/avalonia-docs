---
id: selectabletextblock
title: SelectableTextBlock
description: 一个只读文本标签，允许用户选择并复制显示的文本。
doc-type: reference
---

`SelectableTextBlock` 是一种只读标签，用于显示可供用户选择和复制的文本。它的行为类似 `TextBlock`，但额外内置了使用鼠标或键盘进行文本选择的支持。它可以显示多行文本，并允许你完全控制所使用的字体。

## 常用属性

| 属性 | 类型 | 说明 |
| -------------------------- | ----------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Text` | `string` | 要显示的文本。 |
| `SelectionStart` | `int` | 当前选择起始位置的字符索引。 |
| `SelectionEnd` | `int` | 当前选择结束位置的字符索引。 |
| `SelectedText` | `string` | 获取当前选中的文本（只读）。 |
| `SelectionBrush` | `IBrush` | 用于高亮选中文本的画刷。 |
| `SelectionForegroundBrush` | `IBrush` | 用于选中文本前景色的画刷。 |
| `FontSize` | `double` | 字体大小。 |
| `FontWeight` | `FontWeight` | 字体粗细。默认是 normal，可选值包括 `Bold`。 |
| `FontStyle` | `FontStyle` | 应用于文本的样式。默认是 normal，可选值包括 `Italic`。 |
| `TextDecorations` | `TextDecorationCollection` | 应用于文本的线条装饰。默认值为 none，可选值包括 `Underline`、`Strikethrough`、`Baseline` 和 `Overline`。若要同时应用多个装饰，请用空格分隔列出。 |
| `TextWrapping` | `TextWrapping` | 控制文本到达控件边缘时是否换行。可选值包括 `NoWrap`、`Wrap` 和 `WrapWithOverflow`。 |
| `xml:space` | XML attribute | 设置 `xml:space="preserve"` 可指示 XML 解析器保留换行和空白字符。若不设置该属性，空白字符默认会被裁剪。 |

## 事件

| 事件 | 说明 |
| -------------------- | ------------------------------------------------------------------ |
| `CopyingToClipboard` | 当选中文本即将复制到剪贴板时触发。可用于修改或取消复制操作。 |

## 基本示例

此示例展示了可选中文本作为标题的用法、一个使用自定义选择画刷的单行文本，以及一个预设了选区范围的多行文本显示。

<XamlPreview>

```xml
<StackPanel xmlns="https://github.com/avaloniaui"
            Width="200"
            Margin="20">
  <SelectableTextBlock Margin="0 5" FontSize="18" FontWeight="Bold">Heading</SelectableTextBlock>
  <SelectableTextBlock Margin="0 5" FontStyle="Italic"
                       xml:space="preserve"
                       SelectionBrush="Red">This is a single line.</SelectableTextBlock>
  <SelectableTextBlock Margin="0 5" xml:space="preserve"
                       SelectionStart="3" SelectionEnd="13">This is a multi-line
  display that has
  returns in it.
  The text block
  respects the
  line breaks
  set out in XAML.</SelectableTextBlock>
</StackPanel>
```

</XamlPreview>

## 以编程方式选择文本

你可以在 code-behind 或视图模型中设置 `SelectionStart` 和 `SelectionEnd` 属性，以控制选中文本的范围。

```xml
<SelectableTextBlock x:Name="MyTextBlock"
                     Text="Select part of this text programmatically." />
<Button Content="Select words 2-4" Click="OnSelectClicked" />
```

```csharp
private void OnSelectClicked(object? sender, RoutedEventArgs e)
{
    MyTextBlock.SelectionStart = 7;
    MyTextBlock.SelectionEnd = 24;
}
```

你也可以通过将 `SelectionStart` 设置为 `0`，并将 `SelectionEnd` 设置为文本长度，来选中全部文本。

```csharp
MyTextBlock.SelectionStart = 0;
MyTextBlock.SelectionEnd = MyTextBlock.Text?.Length ?? 0;
```

## 自定义选中外观

你可以通过设置 `SelectionBrush` 和 `SelectionForegroundBrush` 来自定义选中文本的外观。

```xml
<SelectableTextBlock Text="Custom selection colors"
                     SelectionBrush="#335599FF"
                     SelectionForegroundBrush="White" />
```

## 另请参阅

- [TextBlock](/controls/data-display/text-display/textblock)
- [Label](/controls/data-display/text-display/label)
- [SelectableTextBlock API reference](/api/avalonia/controls/selectabletextblock)
- [`SelectableTextBlock.cs` source code on GitHub](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/SelectableTextBlock.cs)
