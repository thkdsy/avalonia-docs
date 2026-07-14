---
id: textblock
title: TextBlock
description: 一个用于显示格式化文本的只读控件，支持多行、内联格式设置以及完整的字体控制。
doc-type: reference
---

import TextBlockRunScreenshot from '/img/controls/textblock/textblock-run.png';
import TextBlockUIContainerScreenshot from '/img/controls/textblock/textblock-uicontainer.png';

[`TextBlock`](/api/avalonia/controls/textblock) 是一个用于显示文本的只读标签。它可以显示多行内容，并允许你完全控制所使用的字体。对于需要让用户选择并复制的文本，请改用 `SelectableTextBlock`。

## 常用属性

| 属性 | 类型 | 说明 |
| ----------------- | -------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Text` | `string` | 要显示的文本。 |
| `FontSize` | `double` | 字体大小，单位为设备无关像素。 |
| `FontWeight` | `FontWeight` | 字体粗细。默认值为 `Normal`，可选值包括 `Bold`。 |
| `FontStyle` | `FontStyle` | 应用于文本的样式。默认值为 `Normal`，可选值包括 `Italic`。 |
| `FontFamily` | `FontFamily` | 用于渲染文本的字体族。你可以使用逗号分隔的列表来指定后备字体。 |
| `Foreground` | `IBrush` | 用于绘制文本的画刷。 |
| `Background` | `IBrush` | 用于绘制文本后方区域的画刷。 |
| [`TextAlignment`](/api/avalonia/media/textalignment) | `TextAlignment` | 控制文本在控件内部的水平对齐方式。可选值包括 `Left`、`Center`、`Right`、`Justify` 和 `DetectFromContent`。 |
| [`TextWrapping`](/api/avalonia/media/textwrapping) | `TextWrapping` | 控制文本到达控件边缘时是否换行。可选值包括 `NoWrap`（默认）、`Wrap` 和 `WrapWithOverflow`。 |
| [`TextTrimming`](/api/avalonia/media/texttrimming) | `TextTrimming` | 控制文本溢出时的截断方式。可选值包括 `None`（默认）、`CharacterEllipsis`、`WordEllipsis` 等。完整说明请参阅 [TextTrimming](/controls/data-display/text-display/texttrimming)。 |
| `MaxLines` | `int` | 限制可见行数。与 `TextWrapping` 和 `TextTrimming` 结合使用时，超出部分会在达到此行数后被截断。 |
| `LineHeight` | `double` | 每一行文本的高度。设置为 `NaN`（默认）时，将由字体度量决定行高。 |
| `TextDecorations` | `TextDecorationCollection` | 应用于文本的线条装饰。默认值为 none，可选值包括 `Underline`、`Strikethrough`、`Baseline` 和 `Overline`。若要同时应用多个装饰，请用空格分隔列出。 |
| `LetterSpacing` | `double` | 字符之间的额外间距，单位为设备无关像素。默认值为 `0`。这是从 `TextElement` 继承而来的附加属性，因此你也可以在父控件上设置它。 |
| `Padding` | `Thickness` | 控件边界与文本内容之间的间距。 |
| `xml:space` | XML attribute | 设置 `xml:space="preserve"` 可指示 XML 解析器保留换行和空白字符。若不设置该属性，空白字符默认会被裁剪。 |

## 基本示例

此示例展示了如何使用多个 `TextBlock` 来显示标题、带有额外空格的单行文本，以及多行文本内容。

<XamlPreview>

```xml
<StackPanel xmlns="https://github.com/avaloniaui" Margin="20">
  <TextBlock Margin="0 5" FontSize="18" FontWeight="Bold">Heading</TextBlock>
  <TextBlock Margin="0 5" FontStyle="Italic" xml:space="preserve">This is  a single line.</TextBlock>
  <TextBlock Margin="0 5" xml:space="preserve">This is a multi-line
  display that has
  returns in it.
  The text block
  respects the line
  breaks set out in XAML.</TextBlock>
</StackPanel>
```

</XamlPreview>

## 文本换行

默认情况下，`TextBlock` 不会自动换行。当文本宽度超过可用空间时，它会被裁剪。你可以通过设置 `TextWrapping` 来控制这一行为：

| 值 | 行为 |
| ----------------- | --------------------------------------------------------------------------------------------- |
| `NoWrap` | 不换行，文本可能会被裁剪（默认）。 |
| `Wrap` | 在可用宽度内容纳得下的最近字符处换行。 |
| `WrapWithOverflow` | 尽可能换行，但如果单个单词宽于控件本身，则允许该单词溢出显示。 |

```xml
<TextBlock Width="200"
           TextWrapping="Wrap"
           Text="This is a long sentence that will wrap when it reaches the edge of the control." />
```

## 文本裁剪

当文本超出可用空间时，你可以显示省略号，而不是直接生硬裁剪。通过设置 `TextTrimming` 属性，你可以控制省略号出现的位置。常见选项包括 `CharacterEllipsis` 和 `WordEllipsis`。

```xml
<TextBlock Width="150"
           TextTrimming="CharacterEllipsis"
           Text="This text will be trimmed with an ellipsis." />
```

你可以将 `TextWrapping`、`TextTrimming` 和 `MaxLines` 结合使用，使文本在固定行数内换行，并在最后一行裁剪超出内容：

```xml
<TextBlock Width="200"
           MaxLines="3"
           TextWrapping="Wrap"
           TextTrimming="WordEllipsis"
           Text="This is a long paragraph that wraps for up to three lines, then trims any remaining overflow with an ellipsis." />
```

如需查看完整的裁剪模式列表和可视化示例，请参阅 [TextTrimming](/controls/data-display/text-display/texttrimming)。

## 文本对齐

使用 `TextAlignment` 属性可以控制文本在控件中的水平位置：

```xml
<StackPanel Width="300" Spacing="8">
  <TextBlock TextAlignment="Left" Text="Left-aligned text" />
  <TextBlock TextAlignment="Center" Text="Center-aligned text" />
  <TextBlock TextAlignment="Right" Text="Right-aligned text" />
  <TextBlock TextAlignment="Justify" TextWrapping="Wrap"
             Text="Justified text spreads words evenly across the full width of the control when wrapping is enabled." />
</StackPanel>
```

## 内联元素

文本内联元素允许你在单个 `TextBlock` 中对文本和控件进行多样化格式设置。通常 `TextBlock.Text` 用于显示一段统一格式的文本，而子内容则允许你使用一组内联元素。

### Run

`Run` 内联元素表示一段格式统一且连续的文本。你可以将 `Run.Text` 绑定到视图模型属性，并为每个 Run 单独设置样式。

```xml
<TextBlock xmlns="https://github.com/avaloniaui">
  <TextBlock.Styles>
    <Style Selector="Run.activity">
      <Setter Property="Foreground" Value="#C469EE" />
      <Setter Property="FontStyle" Value="Italic" />
      <Setter Property="TextDecorations" Value="Underline" />
    </Style>
  </TextBlock.Styles>

  <Run Text="Your name is" />
  <Run FontSize="24" FontWeight="Bold" Foreground="Orange" Text="{Binding Name}" />
  <Run Text="and your favorite activity is" />
  <Run Classes="activity" Text="{Binding Activity}" />
</TextBlock>
```

<Image light={TextBlockRunScreenshot} alt="A TextBlock using Run inlines with mixed formatting and data binding" position="center" maxWidth={400} cornerRadius="true"/>

### LineBreak

`LineBreak` 内联元素会在文本流中强制换行。

<XamlPreview>

```xml
<TextBlock xmlns="https://github.com/avaloniaui">
    This is the first line and<LineBreak />here comes the second
</TextBlock>
```

</XamlPreview>

### Span

[`Span`](/api/avalonia/controls/documents/span) 内联元素可将其他内联元素分组，并应用自己的格式。Avalonia 还提供了几个从 `Span` 派生出的预定义格式内联元素：`Bold`、`Italic` 和 `Underline`。你也可以继承 `Span` 来创建自己的格式，而不一定非要使用样式。

<XamlPreview>

```xml
<TextBlock xmlns="https://github.com/avaloniaui"
           TextWrapping="Wrap">
  This text is <Span Foreground="Green"> green with <Bold>bold sections,</Bold>
  <Italic>italic <Span Foreground="Red">red</Span> sections,</Italic>
  some
  <Run FontSize="24"> enlarged font runs,</Run>
  and</Span>
  back to the original formatting
</TextBlock>
```

</XamlPreview>

### InlineUIContainer

`InlineUIContainer` 允许你将任意 `Control` 作为一个内联元素嵌入到文本流中。

```xml
<TextBlock xmlns="https://github.com/avaloniaui"
           ClipToBounds="False"
           FontSize="32"
           TextWrapping="Wrap">
    This <Span BaselineAlignment="TextTop">example</Span> shows the <Bold>power</Bold> of
    <InlineUIContainer BaselineAlignment="Baseline">
        <Image Width="32" Height="32" VerticalAlignment="Top" Source="/Assets/avalonia-logo.ico" />
    </InlineUIContainer>
    in creating rich text displays with
    <InlineUIContainer>
        <Button Padding="0,8,0,0">
            <TextBlock ClipToBounds="False" FontSize="24" Text="inline button" />
        </Button>
    </InlineUIContainer>
    inline controls
</TextBlock>
```

<Image light={TextBlockUIContainerScreenshot} alt="A TextBlock with inline UI containers including an image and a button" position="center" maxWidth={400} cornerRadius="true"/>

## 另请参阅

- [SelectableTextBlock](/controls/data-display/text-display/selectabletextblock)
- [Label](/controls/data-display/text-display/label)
- [TextTrimming](/controls/data-display/text-display/texttrimming)
- [TextBlock API reference](/api/avalonia/controls/textblock)
- [`TextBlock.cs` source code on GitHub](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/TextBlock.cs)
