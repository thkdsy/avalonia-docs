---
id: document-viewer
title: 文档查看器
doc-type: guide
tags:
 - avalonia pro
 - avalonia enterprise
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

使用 `FlowDocumentScrollViewer` 显示富文档而不进行编辑。本指南涵盖设置、文档加载、样式、布局和常见的查看器模式。

:::info
此控件作为 [Avalonia Pro](https://avaloniaui.net/pricing) 或更高版本的一部分提供。
:::

## 何时使用 FlowDocumentScrollViewer

两个控件可以托管 `FlowDocument`：

| 控件 | 用途 | 选择/复制 | 光标 | 撤销 | 开销 |
|---|---|---|---|---|---|
| `FlowDocumentScrollViewer` | 只读显示，带文本选择/复制 | 是 | 否 | 否 | 低 |
| `RichTextEditor` | 交互式编辑 | 是 | 是 | 是 | 较高 |

在帮助面板、报告预览、文件浏览器和只读摘要中使用 `FlowDocumentScrollViewer`。它直接支持文本选择和剪贴板复制（由编辑器使用的相同 `TextViewMouse` / `TextViewKeyboard` 组件驱动），但不显示插入光标、编辑操作或撤销管理器。

仅当您需要在只读内容上显示插入光标时（例如，放置光标但不允许编辑），才使用 `IsReadOnly="True"` 的 `RichTextEditor`。这会引入完整的编辑基础设施（光标元素、撤销管理器、编辑组件）。

## 安装

```bash
# 核心包（包括 FlowDocument、FlowDocumentScrollViewer、PlainTextSerializer）
dotnet add package Avalonia.Controls.RichTextEditor

# 添加您需要的格式的序列化器
dotnet add package Avalonia.Controls.Documents.Serialization.Rtf     # RTF
dotnet add package Avalonia.Controls.Documents.Serialization.Docx    # DOCX（Open XML）
dotnet add package Avalonia.Controls.Documents.Serialization.Xaml    # XAML 往返
```

所有文档类型（`FlowDocument`、`Paragraph`、`RichRun` 等）和 `FlowDocumentScrollViewer` 都映射到默认的 Avalonia XML 命名空间（`https://github.com/avaloniaui`）。无需额外的 `xmlns` 声明。

## 最小 XAML 示例

`FlowDocument` 是 `FlowDocumentScrollViewer` 的 `[Content]` 属性，因此可以直接作为子元素编写：

```xml
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="文档查看器" Width="800" Height="600">

    <FlowDocumentScrollViewer Padding="20">
        <FlowDocument FontSize="14">
            <Paragraph FontSize="24" FontWeight="Bold">
                <RichRun Text="欢迎" />
            </Paragraph>
            <Paragraph>
                <RichRun Text="本文档在只读查看器中显示。" />
            </Paragraph>
        </FlowDocument>
    </FlowDocumentScrollViewer>

</Window>
```

查看器在 `ScrollViewer` 内包装了一个虚拟化的 `TextViewBase`。默认启用垂直滚动；水平滚动已禁用。

## 从文件加载文档

### 异步加载（推荐）

使用 `FlowDocument.LoadAsync` 反序列化文件并将结果分配给查看器：

```csharp
await using var stream = File.OpenRead("report.rtf");
var document = await FlowDocument.LoadAsync(stream, new RtfSerializer());
viewer.Document = document;
```

`LoadAsync` 在 UI 线程之外执行反序列化。返回的 `FlowDocument` 立即可用于显示。

### 同步加载

```csharp
using var stream = File.OpenRead("report.rtf");
viewer.Document = FlowDocument.Load(stream, new RtfSerializer());
```

对于大文件，建议使用异步加载以避免阻塞 UI 线程。

### 选择序列化器

根据文件格式选择序列化器：

| 扩展名 | 序列化器 | 包 |
|---|---|---|
| `.rtf` | `RtfSerializer` | `Avalonia.Controls.Documents.Serialization.Rtf` |
| `.docx` | `DocxSerializer` | `Avalonia.Controls.Documents.Serialization.Docx` |
| `.xaml` / `.axaml` | `XamlSerializer` | `Avalonia.Controls.Documents.Serialization.Xaml` |
| `.txt` | `PlainTextSerializer` | 核心（已包含） |

一个将扩展名映射到序列化器的辅助方法：

```csharp
static IDocumentSerializer GetSerializer(string path)
{
    return Path.GetExtension(path).ToLowerInvariant() switch
    {
        ".rtf" => new RtfSerializer(),
        ".docx" => new DocxSerializer(),
        ".xaml" or ".axaml" => new XamlSerializer(),
        _ => new PlainTextSerializer()
    };
}
```

### 从嵌入资源加载

使用 Avalonia 的 `AssetLoader` 从程序集资源打开流。
对 FlowDocument 数据文件使用 `.xml` 扩展名——`.axaml` 和 `.xaml` 扩展名会触发 Avalonia 的 XAML 编译器，它无法编译 `FlowDocument` 根元素。

```csharp
var uri = new Uri("avares://MyApp/Assets/Help.xml");
using var stream = AssetLoader.Open(uri);
viewer.Document = FlowDocument.Load(stream, new XamlSerializer());
```

### 从字节数组加载

```csharp
using var stream = new MemoryStream(rtfBytes);
viewer.Document = await FlowDocument.LoadAsync(stream, new RtfSerializer());
```

## 在代码中构建文档

### 手动构建

```csharp
var document = new FlowDocument();

// 标题
var heading = new Paragraph
{
    FontSize = 24,
    FontWeight = FontWeight.Bold,
    Margin = new Thickness(0, 0, 0, 10)
};
heading.Inlines.Add(new RichRun { Text = "报告标题" });
document.Blocks.Add(heading);

// 带有混合格式的正文段落
var body = new Paragraph();
body.Inlines.Add(new RichRun { Text = "状态：" });
body.Inlines.Add(new RichBold(new RichRun { Text = "完成" }));
body.Inlines.Add(new RichRun { Text = "。请参阅 " });
body.Inlines.Add(new RichHyperlink(new RichRun { Text = "详情" })
{
    NavigateUri = new Uri("https://example.com")
});
body.Inlines.Add(new RichRun { Text = " 以获取更多信息。" });
document.Blocks.Add(body);

viewer.Document = document;
```

### FlowDocumentBuilder（流畅 API）

`FlowDocumentBuilder` 提供了简洁的流畅接口来构建文档：

```csharp
using Avalonia.Controls.Documents.TextModel;

var document = FlowDocumentBuilder.Create()
    .AddParagraph("报告标题")
    .AddParagraph()
    .AddText("正文文本，包含 ")
    .AddBold("粗体")
    .AddText(" 和 ")
    .AddItalic("斜体")
    .AddText(" 格式。")
    .Build();

viewer.Document = document;
```

该构建器也支持列表和表格：

```csharp
var document = FlowDocumentBuilder.Create()
    .AddParagraph("购物清单")
    .StartList(TextMarkerStyle.Disc)
        .AddListItem("苹果")
        .AddListItem("面包")
        .AddListItem("牛奶")
    .EndList()
    .AddParagraph("价格表")
    .StartTable()
        .SetTableColumns(new double[] { 200, 100 })
        .StartTableRow()
            .AddTableCell("项目")
            .AddTableCell("价格")
        .EndTableRow()
        .StartTableRow()
            .AddTableCell("苹果")
            .AddTableCell("¥3.00")
        .EndTableRow()
    .EndTable()
    .Build();
```

`InlineFactory` 类提供了用于组合内联元素的静态工厂方法：

```csharp
var doc = FlowDocumentBuilder.Create()
    .AddParagraph(
        InlineFactory.Text("普通 "),
        InlineFactory.Bold("粗体 "),
        InlineFactory.Italic("斜体"))
    .Build();
```

该构建器最适合线性文档。对于深度嵌套的结构（列表项中的表格、混合内容的节），手动构建可以提供更多控制。

## 文档结构参考

```
FlowDocument
├── Paragraph              包含内联元素的块
│   ├── RichRun            带有统一格式的文本
│   ├── RichBold           粗体包装器（RichSpan 子类）
│   ├── RichItalic         斜体包装器
│   ├── RichUnderline      下划线包装器
│   ├── RichSuperscript    上标定位
│   ├── RichSubscript      下标定位
│   ├── RichSpan           通用内联容器
│   ├── RichHyperlink      可点击链接（NavigateUri）
│   ├── RichLineBreak      显式换行
│   └── RichInlineUIContainer 嵌入控件（内联）
├── Section                将块分组在一起
├── List                   项目符号或编号列表
│   └── ListItem           包含块（Paragraph、嵌套 List 等）
├── Table                  网格布局
│   ├── TableColumn        列宽定义
│   └── TableRowGroup      页眉/正文/页脚分组
│       └── TableRow
│           └── TableCell  包含块
└── BlockUIContainer       嵌入控件（全宽块）
```

### 通用块属性

所有块都继承自 `Block` 并共享这些属性：

| 属性 | 类型 | 描述 |
|---|---|---|
| `Margin` | `Thickness` | 外部间距 |
| `Padding` | `Thickness` | 内部间距 |
| `BorderThickness` | `Thickness` | 边框宽度 |
| `BorderBrush` | `IBrush?` | 边框颜色 |
| `CornerRadius` | `CornerRadius` | 圆角 |
| `TextAlignment` | `TextAlignment` | 左对齐、居中、右对齐、两端对齐（继承） |
| `LineHeight` | `double` | 行间距 |
| `FlowDirection` | `FlowDirection` | 从左到右或从右到左（继承） |

### 通用内联属性

| 属性 | 类型 | 可用对象 |
|---|---|---|
| `Text` | `string` | `RichRun` |
| `FontSize` | `double` | 所有内联元素（继承） |
| `FontWeight` | `FontWeight` | 所有内联元素（继承） |
| `FontStyle` | `FontStyle` | 所有内联元素（继承） |
| `FontFamily` | `FontFamily` | 所有内联元素（继承） |
| `Foreground` | `IBrush?` | 所有内联元素（继承） |
| `TextDecorations` | `TextDecorationCollection?` | 所有内联元素 |
| `BaselineAlignment` | `BaselineAlignment` | 所有内联元素 |

## 样式和主题

### 文档级默认值

`FlowDocument` 属性级联到所有子元素：

```xml
<FlowDocumentScrollViewer>
    <FlowDocument FontFamily="Segoe UI" FontSize="14"
                  Foreground="#333333" TextAlignment="Left">
        <Paragraph>
            <RichRun Text="继承 FlowDocument 的字体和颜色。" />
        </Paragraph>
    </FlowDocument>
</FlowDocumentScrollViewer>
```

单个元素覆盖默认值：

```xml
<Paragraph FontSize="24" FontWeight="Bold" Foreground="DarkBlue">
    <RichRun Text="此标题覆盖文档默认值。" />
</Paragraph>
```

### 使用样式进行主题设置

使用 Avalonia 样式控制查看器的外观：

```xml
<Window.Styles>
    <Style Selector="FlowDocumentScrollViewer">
        <Setter Property="Background" Value="{DynamicResource SystemRegionBrush}" />
        <Setter Property="Padding" Value="24" />
    </Style>
</Window.Styles>
```

### 超链接

`RichHyperlink` 支持 `NavigateUri` 属性并引发 `RequestNavigate` 路由事件：

```xml
<Paragraph>
    <RichRun Text="访问 " />
    <RichHyperlink NavigateUri="https://avaloniaui.net">
        <RichRun Text="Avalonia 网站" />
    </RichHyperlink>
    <RichRun Text=" 以获取更多信息。" />
</Paragraph>
```

在代码后置中处理导航：

```csharp
viewer.AddHandler(RichHyperlink.RequestNavigateEvent, (sender, e) =>
{
    if (e.Uri is { } uri)
    {
        Process.Start(new ProcessStartInfo(uri.AbsoluteUri) { UseShellExecute = true });
        e.Handled = true;
    }
});
```

`RichHyperlink` 提供了 `:pointerover`、`:pressed` 和 `:visited` 伪类用于样式设置。

## 页面布局

### PageWidth 和 PagePadding

默认情况下，内容填充可用宽度（`PageWidth = NaN`）。设置固定的 `PageWidth` 来模拟固定宽度的页面：

```xml
<FlowDocumentScrollViewer>
    <FlowDocument PageWidth="700" PagePadding="40">
        <Paragraph>
            <RichRun Text="此内容被限制在宽 700 DIP、内边距 40 DIP 的页面内。" />
        </Paragraph>
    </FlowDocument>
</FlowDocumentScrollViewer>
```

设置 `PageWidth` 后，页面区域在查看器中居中显示，页面区域外的背景保持可见。

### ShowPageBounds

启用 `ShowPageBounds` 可以在页面边界处渲染视觉指示器。这在打印预览场景中很有用：

```xml
<FlowDocumentScrollViewer ShowPageBounds="True">
    <FlowDocument PageWidth="700" PagePadding="40" PageHeight="900">
        <!-- 内容 -->
    </FlowDocument>
</FlowDocumentScrollViewer>
```

## 嵌入控件

### BlockUIContainer

将任何 Avalonia 控件作为全宽块元素嵌入：

```xml
<FlowDocumentScrollViewer>
    <FlowDocument>
        <Paragraph FontSize="20" FontWeight="Bold">
            <RichRun Text="月度收入" />
        </Paragraph>
        <BlockUIContainer>
            <Image Source="/Assets/revenue-chart.png" MaxHeight="300"
                   HorizontalAlignment="Center" />
        </BlockUIContainer>
        <Paragraph>
            <RichRun Text="图 1：过去 12 个月的收入趋势。" />
        </Paragraph>
    </FlowDocument>
</FlowDocumentScrollViewer>
```

在代码中：

```csharp
var container = new BlockUIContainer(new Image
{
    Source = bitmap,
    MaxHeight = 300
});
document.Blocks.Add(container);
```

### RichInlineUIContainer

在文本中内联嵌入一个小控件：

```xml
<Paragraph>
    <RichRun Text="状态：" />
    <RichInlineUIContainer>
        <Border Background="Green" CornerRadius="4" Padding="4,2">
            <TextBlock Text="活跃" Foreground="White" FontSize="11" />
        </Border>
    </RichInlineUIContainer>
    <RichRun Text=" 自 2026 年 1 月起。" />
</Paragraph>
```

:::note
嵌入的控件是实时的 Avalonia 控件。它们参与布局和渲染，但不会捕获到序列化快照中。
:::

## 后台加载和线程安全

### 安全的异步模式

`FlowDocument.LoadAsync` 在后台线程上反序列化并返回准备好在 UI 线程上分配的文档：

```csharp
async Task LoadDocumentAsync(string path)
{
    IDocumentSerializer serializer = GetSerializer(path);

    await using var stream = File.OpenRead(path);
    viewer.Document = await FlowDocument.LoadAsync(stream, serializer);
}
```

### 基于快照的工作流

对于转换管道（加载、显示、重新导出），只需调用一次 `CreateSnapshot()` 并在操作之间共享结果。快照是不可变的，可以从任何线程安全地使用：

```csharp
// UI 线程：获取快照
var snapshot = viewer.Document.CreateSnapshot();

// 后台线程：从一个快照序列化为多种格式
await Task.Run(async () =>
{
    await using var rtfStream = File.Create("output.rtf");
    await new RtfSerializer().SerializeAsync(snapshot, rtfStream);

    await using var docxStream = File.Create("output.docx");
    await new DocxSerializer().SerializeAsync(snapshot, docxStream);
});
```

`SaveAsync` 是一个便利包装器，它在一次调用中创建快照并序列化。当需要从相同的文档状态序列化为多种格式时，直接使用 `CreateSnapshot()`。

有关线程约束的详细讨论，请参阅[线程安全](/controls/input/text-input/richtexteditor/thread-safety)指南。

## 性能考虑

### 虚拟化

`FlowDocumentScrollViewer` 通过其内部的 `TextViewBase` 虚拟化渲染：

- 只有视口内的块加上缓冲区区域会被实例化和测量。
- 未实例化的块使用估计高度，从 24 DIP 开始，并随着块的测量而动态调整。该估计是所有测量块高度的运行平均值。
- 随着用户滚动，估计值会被实际测量值替换。这可能会导致首次滚动经过未见内容时出现轻微的滚动位置调整。

这意味着拥有数千个块的文档仍然响应迅速——渲染成本与可见内容成正比，而不是文档总大小。

### 大文档

对于包含许多块的文档：

- 使用 `FlowDocument.LoadAsync` 以避免在反序列化期间阻塞 UI 线程。
- 避免使用明显宽于视口的 `PageWidth` 值。更宽的页面会产生更长的文本行，增加换行和渲染工作量。
- 如果加载用户提供的文件，请在打开前验证文件大小。

### 复用快照

当文档用于预览然后导出的管道时，使用 `CreateSnapshot()` 创建一个 `DocumentSnapshot` 并复用它。每次调用都会遍历文档树（结构为 O(n)）。一个快照可以序列化为多种格式，无需冗余的树遍历。

有关更多优化技术，请参阅[性能调优](/controls/input/text-input/richtexteditor/performance-tuning)指南。

## 常见模式

### 文件预览面板

一个文件浏览器，在用户选择文件时预览文档。在选择更改时取消正在进行的加载：

```csharp
public partial class FilePreviewPane : UserControl
{
    private CancellationTokenSource? _loadCts;

    public async Task PreviewFileAsync(string path)
    {
        // 取消任何先前的加载
        _loadCts?.Cancel();
        _loadCts = new CancellationTokenSource();
        var token = _loadCts.Token;

        try
        {
            IDocumentSerializer serializer = GetSerializer(path);
            await using var stream = File.OpenRead(path);
            var document = await FlowDocument.LoadAsync(stream, serializer, token);

            token.ThrowIfCancellationRequested();
            Viewer.Document = document;
        }
        catch (OperationCanceledException)
        {
            // 加载完成前选择已更改——这是预期的
        }
    }
}
```

### 帮助/关于查看器

从嵌入资源加载静态 XAML 文档：

```csharp
public partial class HelpWindow : Window
{
    public HelpWindow()
    {
        InitializeComponent();

        var uri = new Uri("avares://MyApp/Assets/Help.xml");
        using var stream = AssetLoader.Open(uri);
        HelpViewer.Document = FlowDocument.Load(stream, new XamlSerializer());
    }
}
```

```xml
<!-- HelpWindow.axaml -->
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        x:Class="MyApp.HelpWindow"
        Title="帮助" Width="600" Height="500">
    <FlowDocumentScrollViewer x:Name="HelpViewer" Padding="20" />
</Window>
```

### 打印预览

结合 `ShowPageBounds`、固定的 `PageWidth` 和 `PageHeight` 来模拟打印页面：

```xml
<FlowDocumentScrollViewer ShowPageBounds="True"
                          Background="#F0F0F0"
                          Padding="40">
    <FlowDocument PageWidth="612" PageHeight="792" PagePadding="72">
        <!-- US Letter：612 × 792 DIP（96 DPI 下 = 8.5 × 11 英寸） -->
        <!-- 72 DIP 内边距 = 0.75 英寸页边距 -->
        <Paragraph FontSize="20" FontWeight="Bold">
            <RichRun Text="季度报告" />
        </Paragraph>
        <Paragraph>
            <RichRun Text="内容按打印边距布局。" />
        </Paragraph>
    </FlowDocument>
</FlowDocumentScrollViewer>
```

### 动态报告生成

从数据模型生成报告并显示：

```csharp
FlowDocument BuildReport(IReadOnlyList<SalesRecord> records)
{
    var builder = FlowDocumentBuilder.Create()
        .AddParagraph("销售报告");

    builder.StartTable()
        .SetTableColumns(new double[] { 200, 120, 120 })
        .StartTableRow()
            .AddTableCell("产品")
            .AddTableCell("数量")
            .AddTableCell("收入")
        .EndTableRow();

    foreach (var record in records)
    {
        builder.StartTableRow()
            .AddTableCell(record.Product)
            .AddTableCell(record.Quantity.ToString())
            .AddTableCell(record.Revenue.ToString("C"))
        .EndTableRow();
    }

    builder.EndTable();

    builder.AddParagraph()
        .AddText("总收入：")
        .AddBold(records.Sum(r => r.Revenue).ToString("C"));

    return builder.Build();
}

// 使用
viewer.Document = BuildReport(salesData);
```

## 限制

`FlowDocumentScrollViewer` 的当前限制：

| 限制 | 解决方法 |
|---|---|
| 无内置搜索/查找 | 针对文档文本实现搜索并以编程方式滚动 |
| 无分页渲染 | 仅连续滚动；`ShowPageBounds` 以视觉方式显示边界 |
| 嵌入控件不序列化 | `BlockUIContainer` / `RichInlineUIContainer` 子级被排除在快照之外 |
| `ITextView` 不公开暴露 | `FlowDocumentScrollViewer` 上的 `TextView` 属性是内部的 |

## 另请参阅

- [RichTextEditor 控件](/controls/input/text-input/richtexteditor)
- [扩展模式](/controls/input/text-input/richtexteditor/extension-patterns)
- [性能调优](/controls/input/text-input/richtexteditor/performance-tuning)
- [线程安全](/controls/input/text-input/richtexteditor/thread-safety)
