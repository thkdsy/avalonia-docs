---
id: markdown-styling
title: Markdown 样式设置
tags:
  - avalonia pro
  - avalonia enterprise
---

`Markdown` 控件构建在共享的 FlowDocument 模型之上，其中每个渲染的元素（`Paragraph`、`Section`、`Table`、`RichSpan`、`RichHyperlink` 等）都是一个完整的 Avalonia `StyledElement`。这意味着您可以使用两种互补的方法来设置 Markdown 输出的样式：

1. **DocumentNode 样式选择器**——按类型和类似 CSS 的类（例如，`Paragraph.h1`、`Section.quoteBlock`）定位元素。
2. **命名资源**——覆盖字体大小、边距和画笔颜色等主题值。

:::info
此控件作为 [Avalonia Pro](https://avaloniaui.net/pricing) 或更高版本的一部分提供。
:::

## DocumentNode 样式选择器

由于所有文档元素都是 `StyledElement` 实例，您可以使用标准的 Avalonia 样式选择器来定位它们。Markdown 渲染器为每个元素应用类似的 CSS 类，因此您可以设置特定 Markdown 结构的样式而不影响其他结构。

### 可用的选择器

下表列出了默认主题使用的样式选择器。您可以在应用程序样式中覆盖或扩展其中的任何一个。

#### 块级选择器

| 选择器 | 描述 |
|---|---|
| `:is(Block)` | 所有块级元素（段落、节等）。设置基础边距。 |
| `:is(Block).codeBlock` | 围栏代码块。背景、边框、内边距和等宽字体。 |
| `Paragraph.header` | 所有标题段落（h1–h6）。粗体字重。 |
| `Paragraph.h1` | 一级标题。 |
| `Paragraph.h2` | 二级标题。 |
| `Paragraph.h3` | 三级标题。 |
| `Paragraph.h4` | 四级标题。 |
| `Paragraph.h5` | 五级标题。 |
| `Paragraph.h6` | 六级标题。 |
| `Section.quoteBlock` | 块引用。左边框、内边距、浅色前景色。 |
| `Section.alertBlock` | 所有警告块（注意、提示、重要、警告、小心）。 |
| `Section.alertBlock.note` | 注意警告块边框。 |
| `Section.alertBlock.tip` | 提示警告块边框。 |
| `Section.alertBlock.important` | 重要警告块边框。 |
| `Section.alertBlock.warning` | 警告警告块边框。 |
| `Section.alertBlock.caution` | 小心警告块边框。 |
| `BlockUIContainer.thematicBreak` | 水平线（主题分割）。 |
| `BlockUIContainer.alertBlockHeader` | 警告块标题容器（图标 + 标签）。 |
| `Table` | 表格元素。边框、单元格间距。 |
| `TableCell` | 表格单元格元素。边框。 |
| `TableRow.tableHeader` | 表头行。粗体字重。 |
| `List` | 列表元素。左内边距、行高。 |
| `ListItem` | 列表项元素。行高。 |

#### 内联选择器

| 选择器 | 描述 |
|---|---|
| `RichRun.code` | 行内代码片段。背景和等宽字体。 |
| `RichSpan.header` | 标题内的内联跨度。粗体字重。 |
| `RichSpan.h1` 到 `RichSpan.h6` | 具有标题级字体大小的内联跨度。 |
| `RichSpan.strikeThrough` | 删除线文本装饰。 |
| `RichSpan.inserted` | 下划线文本装饰（插入的文本）。 |
| `RichSpan.marked` | 高亮/标记文本背景。 |
| `RichHyperlink` | 超链接元素。前景色。 |
| `RichHyperlink:pointerover` | 悬停时的超链接。下划线 + 悬停颜色。 |
| `RichHyperlink:visited` | 已访问的超链接颜色。 |

#### 文档元素选择器

| 选择器 | 描述 |
|---|---|
| `MarkdownCodeBlock` | 代码块元素。在此处设置 `Highlighter` 属性。 |
| `MarkdownImage` | 图像内联元素。在此处设置 `ImageLoader` 属性。 |

### 自定义样式示例

覆盖标题颜色：

```xml
<Style Selector="Paragraph.h1">
    <Setter Property="Foreground" Value="#1a73e8" />
</Style>
<Style Selector="Paragraph.h2">
    <Setter Property="Foreground" Value="#188038" />
</Style>
```

自定义引用块外观：

```xml
<Style Selector="Section.quoteBlock">
    <Setter Property="Background" Value="#f8f9fa" />
    <Setter Property="BorderBrush" Value="#6c757d" />
    <Setter Property="BorderThickness" Value="3,0,0,0" />
    <Setter Property="CornerRadius" Value="4" />
</Style>
```

自定义行内代码样式：

```xml
<Style Selector="RichRun.code">
    <Setter Property="Background" Value="#e8f0fe" />
    <Setter Property="FontFamily" Value="Cascadia Code" />
</Style>
```

具有圆角和其他背景的自定义代码块：

```xml
<Style Selector=":is(Block).codeBlock">
    <Setter Property="Background" Value="#1e1e1e" />
    <Setter Property="Foreground" Value="#d4d4d4" />
    <Setter Property="CornerRadius" Value="8" />
    <Setter Property="Padding" Value="20" />
</Style>
```

作用域嵌套选择器也可用。例如，移除表格内的段落边距：

```xml
<Style Selector="Table Paragraph">
    <Setter Property="Margin" Value="0" />
</Style>
```

## 可自定义的资源

除了样式选择器之外，您还可以在主题或资源字典中覆盖命名资源。默认样式会使用这些资源，它们提供了一种无需重写选择器即可调整值的简便方法。

### 块
| 键 | 类型 | 默认值 | 说明 |
|---|---|---|---|
| `MarkdownBlockMargin` | Thickness | `0,8` | 块的外边距 |

### 超链接
| 键 | 类型 | 默认值（亮色） | 默认值（暗色） | 说明 |
|---|---|---|---|---|
| `MarkdownHyperlinkForeground` | Brush | `#0969da` | `#58a6ff` | 链接颜色 |
| `MarkdownHyperlinkForegroundVisited` | Brush | `#551A8B` | `#bc8cff` | 已访问链接颜色 |
| `MarkdownHyperlinkForegroundPointerOver` | Brush | `#0056b3` | `#79c0ff` | 悬停链接颜色 |

### 选择
| 键 | 类型 | 默认值 | 说明 |
|---|---|---|---|
| `MarkdownSelectionBrush` | Brush | `#FF086F9E` | 选择高亮 |

### 代码
| 键 | 类型 | 默认值 | 说明 |
|---|---|---|---|
| `MarkdownCodeFontFamily` | FontFamily | `Courier New` | 行内/代码字体 |

### 代码块
| 键 | 类型 | 默认值（亮色） | 默认值（暗色） | 说明 |
|---|---|---|---|---|
| `MarkdownCodeBlockParagraphPadding` | Thickness | `16` | `16` | 内边距 |
| `MarkdownCodeBlockParagraphBorderThickness` | Thickness | `1` | `1` | 边框厚度 |
| `MarkdownCodeBlockParagraphCornerRadius` | CornerRadius | `6` | `6` | 圆角半径 |
| `MarkdownCodeBlockParagraphBackground` | Brush | `#1f818b98` | `#20484f58` | 背景 |
| `MarkdownCodeBlockParagraphBorderBrush` | Brush | `#e3ebf6` | `#30363d` | 边框画笔 |

### 行内代码
| 键 | 类型 | 默认值（亮色） | 默认值（暗色） | 说明 |
|---|---|---|---|---|
| `MarkdownCodeRunBackground` | Brush | `#1f818b98` | `#20484f58` | 行内代码背景 |

### 主题分割
| 键 | 类型 | 默认值（亮色） | 默认值（暗色） | 说明 |
|---|---|---|---|---|
| `MarkdownThematicBreakRectangleFill` | Brush | `#d1d9e0` | `#30363d` | 主题分割颜色 |

### 标题（按级别）
每个标题都有 `FontSize`、`BorderThickness`、`Padding`、`Margin`。

| 键 | 类型 | 默认值 | 说明 |
|---|---|---|---|
| `MarkdownHeader1ParagraphFontSize` | Double | `31.5` | H1 字号 |
| `MarkdownHeader1ParagraphBorderThickness` | Thickness | `0,0,0,1` | H1 底部边框 |
| `MarkdownHeader1ParagraphPadding` | Thickness | `0,0,0,16` | H1 内边距（底部） |
| `MarkdownHeader1ParagraphMargin` | Thickness | `0,31,0,14` | H1 外边距 |
| `MarkdownHeader2ParagraphFontSize` | Double | `24.5` | H2 字号 |
| `MarkdownHeader2ParagraphBorderThickness` | Thickness | `0,0,0,1` | H2 底部边框 |
| `MarkdownHeader2ParagraphPadding` | Thickness | `0,0,0,12` | H2 内边距（底部） |
| `MarkdownHeader2ParagraphMargin` | Thickness | `0,24.5,0,14` | H2 外边距 |
| `MarkdownHeader3ParagraphFontSize` | Double | `21` | H3 字号 |
| `MarkdownHeader3ParagraphBorderThickness` | Thickness | `0,0,0,1` | H3 底部边框 |
| `MarkdownHeader3ParagraphPadding` | Thickness | `0,0,0,8` | H3 内边距（底部） |
| `MarkdownHeader3ParagraphMargin` | Thickness | `0,21,0,14` | H3 外边距 |
| `MarkdownHeader4ParagraphFontSize` | Double | `16.8` | H4 字号 |
| `MarkdownHeader4ParagraphBorderThickness` | Thickness | `0,0,0,1` | H4 底部边框 |
| `MarkdownHeader4ParagraphPadding` | Thickness | `0,0,0,6` | H4 内边距（底部） |
| `MarkdownHeader4ParagraphMargin` | Thickness | `0,16.8,0,14` | H4 外边距 |
| `MarkdownHeader5ParagraphFontSize` | Double | `14` | H5 字号 |
| `MarkdownHeader5ParagraphBorderThickness` | Thickness | `0,0,0,1` | H5 底部边框 |
| `MarkdownHeader5ParagraphPadding` | Thickness | `0,0,0,4` | H5 内边距（底部） |
| `MarkdownHeader5ParagraphMargin` | Thickness | `0,14,0,14` | H5 外边距 |
| `MarkdownHeader6ParagraphFontSize` | Double | `14` | H6 字号 |
| `MarkdownHeader6ParagraphBorderThickness` | Thickness | `0,0,0,1` | H6 底部边框 |
| `MarkdownHeader6ParagraphPadding` | Thickness | `0,0,0,2` | H6 内边距（底部） |
| `MarkdownHeader6ParagraphMargin` | Thickness | `0,14,0,14` | H6 外边距 |

### 引用块
| 键 | 类型 | 默认值（亮色） | 默认值（暗色） | 说明 |
|---|---|---|---|---|
| `MarkdownQuoteBlockSectionBorderThickness` | Thickness | `4,0,0,0` | `4,0,0,0` | 引用块左边框厚度 |
| `MarkdownQuoteBlockSectionBorderBrush` | Brush | `#DDDDDD` | `#3b434b` | 引用块边框画笔 |
| `MarkdownQuoteBlockSectionForeground` | Brush | `#777777` | `#8b949e` | 引用块前景色 |
| `MarkdownQuoteBlockSectionPadding` | Thickness | `15,0` | `15,0` | 引用块内边距 |
| `MarkdownQuoteBlockFirstChildSectionMargin` | Thickness | `0,0,0,14` | `0,0,0,14` | 引用中第一个子元素的边距 |
| `MarkdownQuoteBlockLastChildSectionMargin` | Thickness | `0` | `0` | 引用中最后一个子元素的边距 |

### 表格
| 键 | 类型 | 默认值（亮色） | 默认值（暗色） | 说明 |
|---|---|---|---|---|
| `MarkdownTableCellBorderBrush` | Brush | `Black` | `#30363d` | 表格单元格边框画笔 |
| `MarkdownTableBorderBrush` | Brush | `Black` | `#30363d` | 表格边框画笔 |
| `MarkdownTableBorderThickness` | Thickness | `0,0,1,1` | `0,0,1,1` | 表格外边框厚度 |
| `MarkdownTableCellSpacing` | Double | `0` | — | 单元格间距 |
| `MarkdownTableCellBorderThickness` | Thickness | `1,1,0,0` | `1,1,0,0` | 单元格边框厚度 |
| `MarkdownTableCellParagraphPadding` | Thickness | `12,5` | `12,5` | 单元格段落内边距 |

### 内联样式
| 键 | 类型 | 默认值（亮色） | 默认值（暗色） | 说明 |
|---|---|---|---|---|
| `MarkdownMarkedSpanBackground` | Brush | `Yellow` | `#bb8009` | 标记跨度的高亮背景 |

### 警告块（按类型）
| 键 | 类型 | 默认值（亮色） | 默认值（暗色） | 说明 |
|---|---|---|---|---|
| `MarkdownAlertBlockNoteBorderBrush` | Brush | `#0969da` | `#58a6ff` | 注意警告边框画笔 |
| `MarkdownAlertBlockNoteParagraphForeground` | Brush | `#0969da` | `#58a6ff` | 注意警告段落前景色 |
| `MarkdownAlertBlockTipBorderBrush` | Brush | `#1a7f37` | `#3fb950` | 提示警告边框画笔 |
| `MarkdownAlertBlockTipParagraphForeground` | Brush | `#1a7f37` | `#3fb950` | 提示警告段落前景色 |
| `MarkdownAlertBlockImportantBorderBrush` | Brush | `#8250df` | `#bc8cff` | 重要警告边框画笔 |
| `MarkdownAlertBlockImportantParagraphForeground` | Brush | `#8250df` | `#bc8cff` | 重要警告段落前景色 |
| `MarkdownAlertBlockWarningBorderBrush` | Brush | `#9a6700` | `#d29922` | 警告警告边框画笔 |
| `MarkdownAlertBlockWarningParagraphForeground` | Brush | `#9a6700` | `#d29922` | 警告警告段落前景色 |
| `MarkdownAlertBlockCautionBorderBrush` | Brush | `#d1242f` | `#f85149` | 小心警告边框画笔 |
| `MarkdownAlertBlockCautionParagraphForeground` | Brush | `#d1242f` | `#f85149` | 小心警告段落前景色 |
| `MarkdownAlertBlockHeaderMargin` | Thickness | `0 0 0 8` | — | 警告标题边距 |

### 警告标题内容模板
- `MarkdownAlertBlockHeaderNoteContentTemplate` — 注意警告标题的内容模板
- `MarkdownAlertBlockHeaderTipContentTemplate` — 提示警告标题的内容模板
- `MarkdownAlertBlockHeaderImportantContentTemplate` — 重要警告标题的内容模板
- `MarkdownAlertBlockHeaderWarningContentTemplate` — 警告警告标题的内容模板
- `MarkdownAlertBlockHeaderCautionContentTemplate` — 小心警告标题的内容模板

### 复制按钮
| 键 | 类型 | 默认值（亮色） | 默认值（暗色） | 说明 |
|---|---|---|---|---|
| `MarkdownCopyButtonFill` | Brush | `#59636e` | `#8b949e` | 复制按钮填充色 |
| `MarkdownCopyButtonContentTemplate` | DataTemplate | — | — | 复制按钮的内容模板 |

## 如何覆盖

### 样式选择器（推荐）

将您的样式放在 `App.axaml` 中 `Default.axaml` 的 `StyleInclude` 之后，或放在合并的资源字典中。后面的样式具有更高优先级：

```xml
<Application.Styles>
    <StyleInclude Source="avares://Avalonia.Controls.Markdown/Themes/Default.axaml" />

    <!-- 您的覆盖 -->
    <Style Selector="Paragraph.h1">
        <Setter Property="Foreground" Value="Navy" />
    </Style>
    <Style Selector="Section.quoteBlock">
        <Setter Property="Background" Value="#f0f4f8" />
    </Style>
</Application.Styles>
```

### 命名资源

在您的应用程序主题或资源字典中定义资源：

```xml
<SolidColorBrush x:Key="MarkdownHyperlinkForeground" Color="#FF0000" />
```

## 示例：自定义主题

```xml
<ResourceDictionary>
    <SolidColorBrush x:Key="MarkdownSelectionBrush" Color="#FFD700" />
    <FontFamily x:Key="MarkdownCodeFontFamily">Cascadia Code</FontFamily>
</ResourceDictionary>
```

结合样式选择器实现暗色代码主题：

```xml
<Style Selector=":is(Block).codeBlock">
    <Setter Property="Background" Value="#2d2d2d" />
    <Setter Property="Foreground" Value="#cccccc" />
    <Setter Property="CornerRadius" Value="8" />
</Style>
```

## 另请参阅

- [Markdown 控件](/controls/data-display/text-display/markdown)
