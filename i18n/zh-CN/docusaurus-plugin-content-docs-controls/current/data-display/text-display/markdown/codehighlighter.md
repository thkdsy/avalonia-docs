---
id: codehighlighter
title: 代码高亮器
description: 为 Markdown 控件渲染的代码块添加语法高亮功能，提供 ColorCode 和 TextMate 两种实现，作为独立包提供。
doc-type: reference
tags:
  - avalonia pro
  - avalonia enterprise
---

`Markdown` 控件通过 `MarkdownCodeBlock` 上的 `Highlighter` 属性支持围栏代码块的语法高亮。由于 Markdown 控件构建在共享文档模型之上，每个代码块都是一个完整的 `StyledElement`——您可以使用标准的 Avalonia 样式选择器来分配高亮器。两个实现作为独立的 NuGet 包提供：`ColorCodeHighlighter`（轻量级，语言支持有限）和 `TextMateHighlighter`（完整的 TextMate 语法支持，带主题）。

:::info
此控件作为 [Avalonia Pro](https://avaloniaui.net/pricing) 或更高版本的一部分提供。
:::

## 安装

高亮器作为独立的 NuGet 包分发。安装适合您需求的那个：

**ColorCode** 提供一个轻量级的高亮器，覆盖常见语言，如 C#、XML、JSON 和 JavaScript：

```bash
dotnet add package Avalonia.Controls.Markdown.ColorCode
```

**TextMate** 提供完整的 TextMate 语法支持，带有内置主题，覆盖多种语言：

```bash
dotnet add package Avalonia.Controls.Markdown.TextMate
```

## 选择高亮器

| 特性 | `ColorCodeHighlighter` | `TextMateHighlighter` |
|---|---|---|
| 语言覆盖 | 常见语言（C#、XML、JSON、JS 等） | 通过 TextMate 语法提供广泛覆盖 |
| 主题 | 继承您的应用程序主题颜色 | 内置主题，如 `LightPlus`、`DarkPlus` 等 |
| 包大小 | 较小 | 较大（包含语法文件） |
| 设置 | 最少 | 需要设置 `Theme` 属性值 |

如果您只需要高亮少数几种流行语言，并希望保持依赖项小巧，请使用 `ColorCodeHighlighter`。如果您需要广泛的语言支持，或希望独立于应用程序主题控制颜色主题，请使用 `TextMateHighlighter`。

## 在 XAML 中使用 `TextMateHighlighter`

将高亮器定义为资源，并通过样式将其分配给 `MarkdownCodeBlock` 元素：

```xml
<Window xmlns="https://github.com/avaloniaui"
        xmlns:textMate="clr-namespace:Avalonia.Controls;assembly=Avalonia.Controls.Markdown.TextMate">
  <Window.Resources>
    <textMate:TextMateHighlighter x:Key="TextMateHighlighter" Theme="LightPlus"/>
  </Window.Resources>

  <Window.Styles>
    <Style Selector="MarkdownCodeBlock">
      <Setter Property="Highlighter" Value="{StaticResource TextMateHighlighter}" />
    </Style>
  </Window.Styles>

  <Markdown Text="# 示例&#10;&#10;```csharp&#10;var x = 1;&#10;```" />
</Window>
```

您可以通过更改 `TextMateHighlighter` 资源上的 `Theme` 属性在运行时切换主题。当高亮器触发其 `Invalidated` 事件时，代码块会自动重新高亮。

## 在 XAML 中使用 `ColorCodeHighlighter`

```xml
<Window xmlns="https://github.com/avaloniaui"
        xmlns:cc="clr-namespace:Avalonia.Controls;assembly=Avalonia.Controls.Markdown.ColorCode">
  <Window.Resources>
    <cc:ColorCodeHighlighter x:Key="ColorCodeHighlighter" />
  </Window.Resources>

  <Window.Styles>
    <Style Selector="MarkdownCodeBlock">
      <Setter Property="Highlighter" Value="{StaticResource ColorCodeHighlighter}" />
    </Style>
  </Window.Styles>

  <Markdown Text="# 示例&#10;&#10;```csharp&#10;var x = 1;&#10;```" />
</Window>
```

## 在代码后置中设置高亮器

您也可以以编程方式创建样式，或将高亮器分配给特定的 `MarkdownCodeBlock` 实例：

```csharp
// 创建高亮器
var textMateHighlighter = new TextMateHighlighter { Theme = "DarkPlus" };

// 选项 1：通过样式应用（推荐——影响所有代码块）
var style = new Style(x => x.OfType<MarkdownCodeBlock>());
style.Setters.Add(new Setter(MarkdownCodeBlock.HighlighterProperty, textMateHighlighter));
myMarkdownControl.Styles.Add(style);
```

## 在代码块中指定语言

要获得正确的高亮效果，请在 Markdown 源中开始的三连反引号后指定语言标识符。例如：

````markdown
```csharp
Console.WriteLine("Hello, world!");
```
````

如果省略语言标识符，高亮器将把代码块渲染为纯文本而不着色。语言标识符存储在 `MarkdownCodeBlock.LanguageId` 属性上。

## 说明

- `Markdown` 控件会监听高亮器上的属性更改，并在您更新诸如 `Theme` 等属性时自动重新渲染代码块。
- 每个 `Markdown` 控件通过样式接受一个在其所有代码块之间共享的 `CodeHighlighter` 实例。如果您有多个 `Markdown` 控件，可以共享同一个高亮器资源。
- `MarkdownCodeBlock` 扩展了 `Paragraph`，是一个完整的 `StyledElement`，因此您可以在单个选择器中将高亮器样式与其他视觉自定义（背景、内边距、字体系列）结合使用。

## 另请参阅

- [Markdown 控件](/controls/data-display/text-display/markdown)
- [Markdown 样式设置](/controls/data-display/text-display/markdown/markdown-styling)
