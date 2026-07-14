---
id: index
title: Markdown 控件
description: 使用 Avalonia.Controls.Markdown 控件渲染 Markdown 格式文本，该控件构建在共享的 FlowDocument 模型之上，具有完整的 DocumentNode 样式、选择、流式传输和自定义图像加载功能。
doc-type: reference
tags:
  - avalonia pro
  - avalonia enterprise
---

`Markdown` 控件在 Avalonia 应用程序中渲染 Markdown 格式的文本。它构建在与 [RichTextEditor](/controls/input/text-input/richtexteditor) 相同的 FlowDocument 模型之上，从而提供对基于 DocumentNode 的样式、文本选择和高性能流式更新的全面支持。

:::info
此控件作为 [Avalonia Pro](https://avaloniaui.net/pricing) 或更高版本的一部分提供。
:::

## 入门指南

1. 通过运行 `dotnet add package` 安装 `Avalonia.Controls.Markdown` NuGet 包。

```bash
dotnet add package Avalonia.Controls.Markdown
```

2. 在可执行项目文件（`.csproj`）中包含您的 Avalonia 许可证密钥。您可以从 [Avalonia 门户](https://portal.avaloniaui.net) 获取您的许可证密钥。

```xml
<ItemGroup>
  <AvaloniaUILicenseKey Include="您的许可证密钥" />
</ItemGroup>
```

:::tip
对于多项目解决方案，您可以将许可证密钥存储在[环境变量](https://learn.microsoft.com/en-us/visualstudio/msbuild/how-to-use-environment-variables-in-a-build)或[共享属性文件](https://learn.microsoft.com/en-us/visualstudio/msbuild/customize-by-directory?view=vs-2022#directorybuildprops-example)中，以避免重复。
:::

3. 在您的 `App.axaml` 文件中通过 `StyleInclude` 引用默认主题。这将添加 Markdown 控件所需的资源。

```xml
<Application.Styles>
   <StyleInclude Source="avares://Avalonia.Controls.Markdown/Themes/Default.axaml" />
   <!-- 其他样式 -->
</Application.Styles>
```

有关安装 Avalonia Pro 控件的更多信息，请参阅[安装 Avalonia Pro](/tools/installing-avalonia-pro)。

## 架构

`Markdown` 控件使用以下管道将 Markdown 文本转换为实时的 `FlowDocument`：

1. **Markdown 文本** 被解析为 Markdig AST。
2. AST 被渲染为 `DocumentSnapshot`（一个不可变、线程安全的表示）。
3. 快照被应用到 `FlowDocument`，后者拥有实时的文档树。
4. `EditorTextView`（与 `RichTextEditor` 使用的相同渲染器）渲染文档。

由于该控件与 `RichTextEditor` 共享文档模型，所有文档元素（`Paragraph`、`Section`、`Table`、`RichSpan`、`RichHyperlink` 等）都是完整的 `StyledElement` 实例。这意味着您可以使用 Avalonia 样式选择器和类似 CSS 的类来定位它们，而不仅仅是命名的资源。

## 使用示例

### XAML 用法

```xml
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="using:MarkdownSample"
        mc:Ignorable="d"
        x:Class="MarkdownSample.MainWindow">
 <Markdown xml:space="preserve">
  # Markdown
  ## 标题
  ### 三级标题
  #### 四级标题
  ##### 五级标题
  ###### 六级标题

  **粗体文本**
  *斜体文本*
  ~~删除线~~
  __粗体__ 和 _斜体_

  [链接到 Avalonia](https://avaloniaui.net)

  `行内代码`

  - 无序列表项 1
  - 无序列表项 2
    - 嵌套项 2a
    - 嵌套项 2b
  ---
  1. 有序列表项 1
  2. 有序列表项 2
     1. 嵌套有序项 2a
     2. 嵌套有序项 2b
  ---
  > 块引用示例
  >> 嵌套块引用
  ---
  | 页眉 1 | 页眉 2 |
  |----------|----------|
  | 单元格 1 | 单元格 2 |
  | 单元格 3 | 单元格 4 |

  ![示例图片](https://private-user-images.githubusercontent.com/552074/446176752-21950b56-cd28-4574-9a0a-73bb17b89d31.png)
 </Markdown>
</Window>
```
> **注意：** XAML 示例中的 `xml:space="preserve"` 属性很重要。它确保 Markdown 文本中的空白和换行符被保留，不会被 XAML 编译器规范化。在 XAML 中直接嵌入 Markdown 时，请始终包含此属性。

### XAML 用法（带视图模型绑定）

```xml
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="using:MarkdownSample"
        mc:Ignorable="d"
        x:Class="MarkdownSample.MainWindow">
    <Window.DataContext>
        <local:MarkdownViewModel/>
    </Window.DataContext>
    <Markdown Text="{Binding MarkdownText}" />
</Window>
```

### 视图模型示例（从文件加载）

```csharp
namespace MarkdownSample;

public class MarkdownViewModel
{
    public string MarkdownText { get; }

    public MarkdownViewModel()
    {
        MarkdownText = File.ReadAllText("Example.md");
    }
}
```

### C# 用法

```csharp
var markdown = new Markdown
{
    Text = "# Hello, Markdown!"
};
```

### 流式用法

`Markdown` 控件支持增量追加文本，这对于渲染来自 LLM 或其他流式源的内容非常有用：

```csharp
// 简单的便捷方法，自动管理会话
markdown.AppendText("# 流式传输\n\n内容到达 ");
markdown.AppendText("增量地...");

// 通过流式会话完全控制
using var session = markdown.BeginStreaming();
session.Append("# 响应\n\n");
session.Append("段落文本...");
await session.CompleteAsync();
```

流式引擎在后台线程上运行 Markdown 解析，将 AST 与先前版本进行差异比较，并仅将更改的块应用到实时文档。将 `AutoScrollToEnd="True"` 设置为在流式传输期间将视口固定在底部。

## API 概述

### 属性

| 属性              | 类型                        | 描述                                              |
|--------------------|-----------------------------|----------------------------------------------------|
| Text               | string?                     | 要渲染的 Markdown 文本。                           |
| SelectionBrush     | IBrush?                     | 选择高亮的画笔。                                   |
| CaretBrush         | IBrush?                     | 光标的画笔。                                       |
| SelectedText       | string                      | 只读。当前选择的内容。                             |
| CanCopy            | bool                        | 只读。是否可以执行复制命令。                       |
| AutoScrollToEnd    | bool                        | 在流式传输期间保持视口滚动到底部。                 |

### 方法

| 方法                             | 描述                                                           |
|-----------------------------------|----------------------------------------------------------------|
| Copy()                            | 将当前选择复制到剪贴板。                                       |
| SelectAll()                       | 选择所有内容。                                                 |
| ClearSelection()                  | 清除当前选择。                                                 |
| ScrollToEnd()                     | 将视口滚动到文档末尾。                                         |
| BeginStreaming()                  | 启动一个 `MarkdownStreamingSession` 用于增量追加。             |
| AppendText(string, TimeSpan?)     | 便捷方法，创建或复用流式会话。                                 |

### 事件

| 事件                | 描述                                             |
|---------------------|---------------------------------------------------|
| CopyingToClipboard  | 在选择复制到剪贴板之前触发。可以阻止复制。        |
| SelectionChanged    | 当文本选择更改时触发。                             |

## 自定义 Markdown 管道

`Markdown` 控件使用 [Markdig](https://github.com/xoofx/markdig) 进行解析。默认情况下启用以下扩展：自动链接、警告块、表情符号、脚注、网格表格、管道表格、强调扩展（删除线、上标、下标）、任务列表和一个内置符号扩展。

要添加或移除 Markdig 扩展，请继承 `Markdown` 并重写 `ConfigurePipeline`。构建器已经应用了默认扩展，因此您的重写是累积的：

```csharp
public class CustomMarkdown : Markdown
{
    protected override void ConfigurePipeline(MarkdownPipelineBuilder builder)
    {
        // 添加 Markdig 数学扩展（LaTeX 风格数学块）
        builder.UseMathematics();
    }
}
```

管道构建一次并在控件实例的整个生命周期中缓存。它同时用于静态渲染和流式会话。

### 移除默认扩展

Markdig 没有提供内置的 `Remove` 辅助方法，但您可以在管道构建之前按类型移除扩展：

```csharp
public class NoEmojiMarkdown : Markdown
{
    protected override void ConfigurePipeline(MarkdownPipelineBuilder builder)
    {
        // 移除默认配置添加的表情符号扩展
        var emoji = builder.Extensions.OfType<Markdig.Extensions.Emoji.EmojiExtension>().FirstOrDefault();
        if (emoji != null)
            builder.Extensions.Remove(emoji);
    }
}
```

### 在 XAML 中使用子类

为您的子类注册一个命名空间，并在使用 `Markdown` 的地方替换使用：

```xml
<Window xmlns:local="using:MyApp">
    <local:CustomMarkdown Text="{Binding MarkdownText}" />
</Window>
```

### 向子类应用默认主题

默认主题和样式直接针对基类型，不会自动应用于派生类型。如果您使用子类（如上述示例中的 `CustomMarkdown`），它将呈现为无样式状态。

要将针对 `Markdown` 的相同样式应用到 `CustomMarkdown`，请通过重写 `StyleKeyOverride` 将样式键重定向回基类型。

```csharp
public class CustomMarkdown : Markdown
{
    protected override Type StyleKeyOverride => typeof(Markdown);

    // 您的自定义
}
```

## 自定义图像加载器

图像加载由 `MarkdownImage` 文档元素处理，而不是 `Markdown` 控件本身。您通过针对 `MarkdownImage` 的样式来分配加载器：

```xml
<Style Selector="MarkdownImage">
    <Setter Property="ImageLoader" Value="{StaticResource MyCustomLoader}" />
</Style>
```

请参阅[图像加载器](/controls/data-display/text-display/markdown/imageloader)页面，了解如何实现自定义 `MarkdownImageLoader` 的详细示例。

## 代码高亮器

语法高亮由 `MarkdownCodeBlock` 文档元素处理。您通过样式分配高亮器：

```xml
<Style Selector="MarkdownCodeBlock">
    <Setter Property="Highlighter" Value="{StaticResource TextMateHighlighter}" />
</Style>
```

请参阅[代码高亮器](/controls/data-display/text-display/markdown/codehighlighter)页面了解安装包和使用示例。

## 样式设置

由于 `Markdown` 控件构建在共享文档模型之上，所有渲染的元素（`Paragraph`、`Section`、`Table`、`RichSpan`、`RichHyperlink` 等）都是完整的 Avalonia `StyledElement` 实例，具有类似 CSS 的类。您可以使用标准的 Avalonia 样式选择器来设置它们的样式：

```xml
<!-- 将所有 H1 标题设为红色 -->
<Style Selector="Paragraph.h1">
    <Setter Property="Foreground" Value="Red" />
</Style>

<!-- 自定义引用块背景 -->
<Style Selector="Section.quoteBlock">
    <Setter Property="Background" Value="#f0f0f0" />
</Style>
```

对于字体大小、边距和主题变体颜色等常用值，也提供了命名的资源。

请参阅[Markdown 样式设置](/controls/data-display/text-display/markdown/markdown-styling)页面获取完整的样式选择器和资源列表。

## 安装

请参阅[安装指南](/tools/installing-avalonia-pro)获取有关如何安装 Avalonia Pro 组件的分步说明。

向您的项目添加 Markdown 包：

```bash
dotnet add package Avalonia.Controls.Markdown
```

通过在 `App.axaml` 中使用 `StyleInclude` 引用附带的 `Default.axaml` 主题来添加资源：

```xml
<Application.Styles>
   <StyleInclude Source="avares://Avalonia.Controls.Markdown/Themes/Default.axaml" />
   <!-- 其他样式 -->
</Application.Styles>
```

## 另请参阅

- [Markdown 样式设置](/controls/data-display/text-display/markdown/markdown-styling)
- [图像加载器](/controls/data-display/text-display/markdown/imageloader)
- [代码高亮器](/controls/data-display/text-display/markdown/codehighlighter)
