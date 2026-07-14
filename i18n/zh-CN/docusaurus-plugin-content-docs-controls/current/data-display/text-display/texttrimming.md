---
id: texttrimming
title: TextTrimming
description: 关于 TextTrimming 属性
---

import CharacterEllipsis from '/img/reference/text/texttrimming/texttrimming-characterellipsis.png';
import LeadingCharacterEllipsis from '/img/reference/text/texttrimming/texttrimming-leadingcharacterellipsis.png';
import NoTrimming from '/img/reference/text/texttrimming/texttrimming-none.png';
import PrefixCharacterEllipsis from '/img/reference/text/texttrimming/texttrimming-prefixcharacterellipsis.png';
import WordEllipsis from '/img/reference/text/texttrimming/texttrimming-wordellipsis.png';
import TextWrappingWithTextTrimming from '/img/reference/text/texttrimming/textwrapping-with-texttrimming.png';

## 概述

[`TextTrimming`](/api/avalonia/media/texttrimming) 属性允许你控制当文本超出控件可用最大空间时的显示方式。此属性可用于显示文本的控件，例如 [`TextBlock`](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/TextBlock.cs)、[`SelectableTextBlock`](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/SelectableTextBlock.cs) 或 [`ContentPresenter`](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/Presenters/ContentPresenter.cs)。

文本裁剪会添加省略号（…）来表示文本已被截断，而不是直接生硬地切掉文本。

:::note
Avalonia 默认使用 Unicode 省略号字符 `U+2026`，而不是三个句点。
:::

## 裁剪模式

Avalonia 提供了六种文本裁剪选项：

1. None
2. CharacterEllipsis
3. WordEllipsis
4. PrefixCharacterEllipsis
5. LeadingCharacterEllipsis
6. PathSegmentEllipsis

### None

不应用裁剪。文本到达控件边界后会被直接截断。

```xml
<TextBlock Text="This is a very long line of text that will get cut off."
           TextTrimming="None"
           Width="200" />
```

<Image light={NoTrimming} alt="A screenshot of an IDE, displaying a long line of text in a box that is abruptly cut off." position="center" maxWidth={400} cornerRadius="true" />

### CharacterEllipsis

在字符边界后截断文本，并在截断处添加省略号。

适用于通用场景下的裁剪，尤其是在界面设计需要精确控制空间使用时。

```xml
<TextBlock Text="This is a very long line of text that will get cut off."
           TextTrimming="CharacterEllipsis"
           Width="200" />
```

<Image light={CharacterEllipsis} alt="A screenshot of an IDE, displaying a long line of text in a box that is cut off after a character, with an ellipsis added." position="center" maxWidth={400} cornerRadius="true" />

### WordEllipsis

在单词结束处截断文本。会尽量保留完整单词，并在无法继续显示时添加省略号。

适用于希望最大化可读性的场景，避免出现被截断到一半的单词。

```xml
<TextBlock Text="This is a very long line of text that will get cut off."
           TextTrimming="WordEllipsis"
           Width="200" />
```

<Image light={WordEllipsis} alt="A screenshot of an IDE, displaying a long line of text in a box that is cut off after a complete word, with an ellipsis added." position="center" maxWidth={400} cornerRadius="true" />

### PrefixCharacterEllipsis

在中间截断文本。会同时显示字符串的开头和结尾，中间用省略号分隔。

默认会先显示前八个字符，然后显示省略号，再显示足以填满可用空间的后续字符。

适用于文件路径、URL 或其他需要同时显示开头与结尾的文本。

```xml
<TextBlock Text="C:\Users\Documents\Projects\MyProject\source.cs"
           TextTrimming="PrefixCharacterEllipsis"
           Width="200" />
```

<Image light={PrefixCharacterEllipsis} alt="A screenshot of an IDE, displaying a long line of text in a box that is cut off in the middle, with an ellipsis placed between the starting and ending characters." position="center" maxWidth={400} cornerRadius="true" />

### LeadingCharacterEllipsis

从开头裁剪文本。显示内容会以省略号开头，后面保留文本结尾部分的字符。

适用于文件路径或其他只关心结尾内容的文本。

```xml
<TextBlock Text="C:\Users\Documents\Projects\MyProject\source.cs"
           TextTrimming="LeadingCharacterEllipsis"
           Width="200" />
```

<Image light={LeadingCharacterEllipsis} alt="A screenshot of an IDE, displaying a long line of text in a box that is cut off at the start, with an ellipsis replacing the starting characters and the ending characters visible." position="center" maxWidth={400} cornerRadius="true" />

### PathSegmentEllipsis

折叠路径中间的片段，同时保留文件路径或 URL 的开头部分（盘符、服务器名）和结尾部分（文件名）。该算法会移除靠近路径中间的片段，并用省略号替代。

例如，当空间有限时，`C:\Users\Alice\Documents\Projects\Avalonia\src\Button.cs` 会变成 `C:\Users\...\Button.cs`。

```xml
<TextBlock Text="C:\Users\Alice\Documents\Projects\Avalonia\src\Controls\Button.cs"
           TextTrimming="PathSegmentEllipsis"
           Width="200" />
```

此模式同时识别正斜杠和反斜杠作为路径分隔符，因此既适用于文件系统路径，也适用于 URL。

## 使用示例

### 与 MaxWidth 结合使用

将 `TextTrimming` 与 `MaxWidth` 结合，可以创建响应式文本显示效果，并在界面中保持稳定的占用区域。

```xml
<TextBlock Text="{Binding UserName}"
           MaxWidth="300"
           TextTrimming="CharacterEllipsis" />
```

### 与 TextWrapping 结合使用

将 `TextTrimming` 与 `TextWrapping` 结合使用后，在启用换行时，裁剪会应用到最后一行可见文本上。

```xml
<TextBlock Text="{Binding Content}"
           Width="300"
           MaxLines="3"
           TextWrapping="Wrap"
           TextTrimming="WordEllipsis" />
```

<Image light={TextWrappingWithTextTrimming} alt="A screenshot of an IDE, displaying a long line of text in a box that wraps within the box for three lines, before being cut off with an ellipsis added." position="center" maxWidth={400} cornerRadius="true" />

## 另请参阅

- [TextBlock 控件](https://docs.avaloniaui.net/docs/reference/controls/textblock)
- [SelectableTextBlock 控件](https://docs.avaloniaui.net/docs/reference/controls/selectable-textblock)
- [TextTrimming API 参考](https://reference.avaloniaui.net/api/Avalonia.Media/TextTrimming/)
