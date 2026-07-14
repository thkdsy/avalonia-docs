---
id: index
title: RichTextEditor 控件
doc-type: reference
tags:
 - avalonia pro
 - avalonia enterprise
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

`Avalonia.Controls.RichTextEditor` 是 Avalonia 应用程序的富文本编辑解决方案，提供交互式文本编辑、文档架构和文件序列化等功能。

:::info
此控件作为 [Avalonia Pro](https://avaloniaui.net/pricing) 或更高版本的一部分提供。
:::

## 何时使用

使用 `RichTextEditor` 创建用户可以编辑文本内容并执行常见文本操作的区域，如格式化、对齐、高亮或撤销/重做。

## 入门指南

1. 通过运行 `dotnet add package` 安装 `Avalonia.Controls.RichTextEditor` NuGet 包。可选地，为您需要的特定文件格式安装序列化器。

```bash
# 核心编辑器控件和文档模型，包括纯文本序列化
dotnet add package Avalonia.Controls.RichTextEditor

# 序列化器（仅添加您需要的）
dotnet add package Avalonia.Controls.Documents.Serialization.Rtf     # RTF 支持
dotnet add package Avalonia.Controls.Documents.Serialization.Docx    # DOCX（Open XML）支持
dotnet add package Avalonia.Controls.Documents.Serialization.Xaml    # XAML 序列化
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

3. 在您的 `App.axaml` 文件中通过 `StyleInclude` 引用 `RichTextEditor` 的默认主题。这将添加渲染控件所需的资源。

```xml
<Application.Styles>
   <StyleInclude Source="avares://Avalonia.Controls.RichTextEditor/Themes/Default.axaml" />
   <!-- 其他样式 -->
</Application.Styles>
```

有关安装 Avalonia Pro 控件的更多信息，请参阅[安装 Avalonia Pro](/tools/installing-avalonia-pro)。

## 基本用法

使用此设置开始富文本编辑器的基本实现。

<Tabs>
<TabItem value="xaml" label="XAML">

    ```xml
    <Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="我的富文本编辑器"
        Width="800" Height="600">

    <RichTextEditor x:Name="Editor">
        <RichTextEditor.Document>
            <FlowDocument>
                <Paragraph>
                    <RichRun Text="欢迎使用 " />
                    <RichBold>
                        <RichRun Text="RichTextEditor" />
                    </RichBold>
                    <RichRun Text="！" />
                </Paragraph>
            </FlowDocument>
        </RichTextEditor.Document>
    </RichTextEditor>

    </Window>
    ```

</TabItem>

<TabItem value="csharp" label="代码后置">

  ```csharp
  using Avalonia.Controls;
  using Avalonia.Controls.Documents;

  public partial class MainWindow : Window
  {
      public MainWindow()
      {
          InitializeComponent();

          var editor = this.FindControl<RichTextEditor>("Editor");

          // 撤销/重做已准备好使用——编辑器的 UndoManager
          // 在附加 Document 时会自动创建。
          // 要更改限制：
          // editor.UndoLimit = 50;
      }
  }
  ```

</TabItem>
</Tabs>

## 编程方式构建文档

如果愿意，您可以从代码后置而不是 XAML 创建和编辑文档。通过直接调用 `Avalonia.Controls.Documents` 中的相关[组件](#components)、[块元素](#block-elements)或[内联元素](#inline-elements)来实现。

<Tabs>
<TabItem value="create" label="创建文档">

  ```csharp
  var document = new FlowDocument();
  var paragraph = new Paragraph();
  paragraph.Inlines.Add(new RichRun("Hello "));
  paragraph.Inlines.Add(new RichBold(new RichRun("World")));
  paragraph.Inlines.Add(new RichRun("!"));
  document.Blocks.Add(paragraph);

  editor.Document = document;
  ```

</TabItem>

<TabItem value="insert" label="插入文本">

  ```csharp
  var doc = editor.Document?.TextDocument;
  if (doc != null)
  {
      // 在开头插入
      doc.ContentStart.InsertText("页眉：");

      // 在结尾插入
      doc.ContentEnd.InsertText("\n\n页脚");
  }
  ```

</TabItem>

<TabItem value="formatting" label="格式化选中文本">

  ```csharp
  var doc = editor.Document?.TextDocument;
  if (doc != null)
  {
      var range = new TextRange(doc.ContentStart, doc.ContentStart.GetPositionAtOffset(10));

      range.ApplyPropertyValue(RichTextElement.ForegroundProperty, Brushes.Red);
      range.ApplyPropertyValue(RichTextElement.FontSizeProperty, 20.0);
  }
  ```

</TabItem>
</Tabs>

## 加载和保存文件

Load 和 Save 接受一个 `IDocumentSerializer` 实例。每种格式都位于其自己的包中。

```csharp
using Avalonia.Controls.Documents.Serialization.Rtf;

// 加载 RTF（异步，推荐）
await using (var stream = File.OpenRead("document.rtf"))
{
    await editor.LoadAsync(stream, new RtfSerializer());
}

// 保存 RTF（异步，推荐）
await using (var stream = File.Create("output.rtf"))
{
    await editor.SaveAsync(stream, new RtfSerializer());
}
```

同步重载也可用：

```csharp
editor.Load(stream, new RtfSerializer());
editor.Save(stream, new RtfSerializer());
```

可用的序列化器：

| 序列化器 | 包 | 扩展名 |
|---|---|---|
| `RtfSerializer` | `Avalonia.Controls.Documents.Serialization.Rtf` | `.rtf` |
| `DocxSerializer` | `Avalonia.Controls.Documents.Serialization.Docx` | `.docx` |
| `XamlSerializer` | `Avalonia.Controls.Documents.Serialization.Xaml` | `.xaml` |
| `PlainTextSerializer` | `Avalonia.Controls.Documents`（核心） | `.txt` |

### 在没有编辑器的情况下加载文档

`FlowDocument.LoadAsync` 直接从流创建文档，适用于预览或转换场景：

```csharp
await using var stream = File.OpenRead("document.rtf");
var document = await FlowDocument.LoadAsync(stream, new RtfSerializer());
```

## 添加字数统计

您可以创建一个返回字数的回调。在此示例中，我们添加一个连续更新的字数统计，当文本更改时更新。

```csharp
editor.ContentChanged += (sender, args) =>
{
    Console.WriteLine("文档已更改");
    UpdateWordCount();
};

void UpdateWordCount()
{
    string? text = editor.Document?.ContentRange?.Text;
    if (text != null)
    {
        int wordCount = text.Split(new[] { ' ', '\n', '\r' },
                                    StringSplitOptions.RemoveEmptyEntries).Length;
        Console.WriteLine($"字数：{wordCount}");
    }
}
```

## 自定义选择高亮颜色

可以通过为 `SelectionBrush` 指定 ARGB 值来自定义文本选择的高亮颜色。

```xml
<RichTextEditor SelectionBrush="#ffff529e">
```

## 组件

Avalonia 富文本编辑器由三个组件组成：

1. `RichTextEditor`：交互式编辑控件，渲染文档并允许用户打字、选择、格式化、撤销/重做等。
2. `FlowDocumentScrollViewer`：只读查看器，显示文档但不具备编辑功能。
3. `FlowDocument`：文档模型，将富文本内容组织成[块](#block-elements)。

### RichTextEditor 属性

这些属性由 `RichTextEditor` 组件使用。

| 属性 | 类型 | 描述 | 默认值 |
| --- | --- | --- | --- |
| `AcceptsReturn` | `bool` | 确定编辑器是否接受回车键输入。 | `true` |
| `AcceptsTab` | `bool` | 确定编辑器是否接受 Tab 键输入。 | `true` |
| `CaretBrush` | `IBrush` | 光标（文本光标）的颜色。 | 无 |
| `Document` | `FlowDocument` | 选择要显示和编辑的文档。 | 无 |
| `IsReadOnly` | `bool` | 确定编辑器是否为只读。 | `false` |
| `SelectionBrush` | `IBrush` | 文本选择的颜色。 | 无 |
| `SelectionFlyout` | `EditorSelectionFlyout?` | 选择上方显示的迷你工具栏。设置为 `null` 以移除。 | `null` |
| `ShowSelectionFlyout` | `bool` | 显示或隐藏选择弹窗，而无需替换它。 | `true` |
| `Toolbar` | `EditorToolbar` | 自定义工具栏设计和布局。 | `Null` |
| `ShowBlockAdorners` | `bool` | 确定是否显示块装饰器。 | `true` |
| `ShowPageBounds` | `bool` | 确定是否显示页面边界指示器。 | `false` |
| `ShowToolbar` | `bool` | 确定工具栏是否可见。 | `true` |
| `UndoLimit` | `int` | 保留用于撤销操作的最大操作数。 | 100 |

### FlowDocument 属性

这些属性由 `FlowDocument` 组件使用。

| 属性 | 类型 | 描述 | 默认值 |
| --- | --- | --- | --- |
| `Background` | `IBrush` | 文档背景色，以 ARGB 值表示。 | `Null` |
| `FontFamily` | `FontFamily ` | 文档中文本的字体族。 | `Null` |
| `FontSize` | `double` | 文档中文本的字体大小。 | 12 |
| `FontStretch` | `FontStretch` | 文档中文本的字体拉伸，例如 `Normal`、`Condensed`、`Expanded`。 | `Normal` |
| `FontStyle` | `FontStyle` | 文档中文本的字体样式，例如 `Normal`、`Italic`、`Oblique`。 | `Null` |
| `FontWeight` | `FontWeight` | 文档中文本的字体粗细，例如 `Normal`、`Bold`。 | `Normal` |
| `Foreground` | `IBrush` | 文档的前景色，以 ARGB 值表示。 | `Null` |
| `PageHeight` | `double` | 页面高度。 | `double.NaN` |
| `PagePadding` | `Thickness` | 块边框与其内容之间的内部间距。 | `Null` |
| `PageWidth` | `double` | 页面宽度。 | `double.NaN` |
| `TextAlignment` | `TextAlignment` | 文档中文本的对齐方式，即 `Left`、`Center`、`Right`、`Justify`。 | `Null` |

## 块元素

块元素由 `FlowDocument` 使用，用于构建文档模型和组织内容。

| 元素 | 描述 |
| --- | --- |
| `Block` | 块元素的抽象基类。 |
| `BlockUIContainer` | 将 UI 元素作为块嵌入的包装器。 |
| `List` | 显示项目符号或编号列表。 |
| `ListItem` | `List` 中的单个项目。 |
| `Paragraph` | 包含富文本内容的基本块元素。 |
| `Section` | 对其他块元素进行分组的块元素。 |
| `Table` | 显示表格。 |
| `TableCell` | `Table` 中的单个单元格。 |
| `TableColumn` | `Table` 中的一列单元格。 |
| `TableRow` | `Table` 中的一行单元格。 |
| `TableRowGroup` | `Table` 中的一组行。 |

### 属性

| 属性 | 类型 | 描述 | 默认值 |
| --- | --- | --- | --- |
| `Background` | `IBrush` | 块背景色，以 ARGB 值表示。 | `Null` |
| `BorderBrush`| `IBrush` | 块边框颜色，以 ARGB 值表示。 | `Null` |
| `BorderThickness` | `Thickness` | 块边框的厚度。 | `Null` |
| `CellSpacing` | `double` | 由 `Table` 使用。表格单元格之间的间距。 | 0 |
| `Child` | `Control` | 由 `BlockUIContainer` 使用。定义要放入块中的控件。 | `Null` |
| `ColumnSpan` | `int` | 由 `TableCell` 使用。单元格跨越的列数。 | 1 |
| `CornerRadius ` | `CornerRadius` | 应用于块角的半径。 | `Null` |
| `FlowDirection` | `FlowDirection` | 文本流方向，即 `LeftToRight` 或 `RightToLeft`。 | `Null` |
| `FontFamily` | `FontFamily ` | 块中文本的字体族。 | `Null` |
| `FontFeatures` | `FontFeatureCollection` | 应用于块中文本的字体特性集合。 |
| `FontSize` | `double` | 块中文本的字体大小。 | 12 |
| `FontStretch` | `FontStretch` | 块中文本的字体拉伸，例如 `Normal`、`Condensed`、`Expanded`。 | `Normal` |
| `FontStyle` | `FontStyle` | 块中文本的字体样式，例如 `Normal`、`Italic`、`Oblique`。 | `Null` |
| `FontWeight` | `FontWeight` | 块中文本的字体粗细，例如 `Normal`、`Bold`。 | `Normal` |
| `Foreground` | `IBrush` | 块前景色，以 ARGB 值表示。 | `Null` |
| `LetterSpacing` | `double` | 字符之间的额外水平间距。默认值 0 表示正常间距。 | 0 |
| `LineHeight` | `double` | 块中每行文本的高度。 | `double.NaN` |
| `Margin` | `Thickness` | 块元素周围的外部间距。 | `Null` |
| `MarkerOffset` | `double` | 由 `List` 使用。确定列表标记后的间距。 | `double.NaN` |
| `MarkerStyle` | `TextMarkerStyle` | 由 `List` 使用。选择列表标记的样式，例如 `Disc`、`Decimal`、`LowerLatin`。 | `Null` |
| `Padding` | `Thickness` | 块边框与其内容之间的内部间距。 | `Null` |
| `RowSpan` | `int` | 由 `TableCell` 使用。单元格跨越的行数。 | 1 |
| `StartIndex` | `int` | 由 `List` 使用。指定编号列表的起始索引。 | 1 |
| `TabStopPositions` | `double` | 块中文本的制表位位置。 | `double.NaN` |
| `TextAlignment` | `TextAlignment` | 块中文本的对齐方式，即 `Left`、`Center`、`Right`、`Justify`。 | `Null` |
| `TextDecorations` | `TextDecorations` | 应用于块中文本的装饰元素，例如 `Underline`、`Overline`、`Strikethrough`。 |
| `TextIndent` | `double` | 首行文本前的缩进宽度。可以设置为负值以创建悬挂缩进。 | `double.NaN` |

## 内联元素

内联元素用于指定块内的内容样式。

| 元素 | 描述 |
| --- | --- |
| `RichBold` | 指示粗体文本。覆盖全局 `FontWeight` 属性。 |
| `RichHyperlink` | 标记内联超链接。 |
| `RichInline` | 内联元素的抽象基类。 |
| `RichInlineUIContainer` | 在文本流中嵌入 UI 元素的包装器。 |
| `RichItalic` | 指示斜体文本。覆盖全局 `FontStyle` 属性。 |
| `RichLineBreak` | 强制换行。 |
| `RichRun` | 基本文本运行。允许字符级格式化。文本内容由 [`Text` 属性](#properties-1) 定义。 |
| `RichSpan` | 对其他内联元素进行分组的内联元素。 |
| `RichSubscript` | 指示下标文本。将 `BaselineAlignment` 属性设置为 `Subscript`。 |
| `RichSuperscript` | 指示上标文本。将 `BaselineAlignment` 属性设置为 `Superscript`。 |
| `RichUnderline` | 指示下划线文本。覆盖全局 `TextDecorations` 属性。 |

### 属性

| 属性 | 类型 | 使用对象 | 描述 |
| --- | --- | --- | --- |
| `Child` | `Control` | `RichInlineUIContainer` | 定义要放入内联容器中的控件。 |
| `IsVisited` | `bool` | `RichHyperlink` | 超链接是否已被访问。 |
| `NavigateURI` | `Uri` | `RichHyperlink` | 点击超链接时要导航到的 URI。 |
| `Text` | `string` | `RichRun` | 获取或设置文本内容。读写附加的 `TextDocument`。如果未附加，则使用本地存储。 |
| `Tooltip` | `object` | `RichHyperlink` | 与超链接关联的工具提示。 |

### RichHyperlink 伪类

当超链接文本经历状态变化时，`RichHyperlink` 设置以下伪类。

- `:pointerover`：当指针悬停在超链接上时。
- `:pressed`：当超链接被点击时。
- `:visited`：在超链接被点击至少一次后。

## 架构

Avalonia 富文本编辑器将功能分为八层架构。

| 层 | 名称 | 描述 | 关键组件 |
| --- | --- | --- | --- |
| 1 | 文档模型 | 文本上下文和文档层次结构的核心数据存储。使用绳索数据结构进行高效存储和操作。 | `TextDocument`、`TextDocumentNode`、`RopeTextStore`、`RopeSnapshot` |
| 2 | 文本指针 API | 文档中的位置跟踪和导航。 | `TextPointer`、`TextRange`、`LogicalDirection` |
| 3 | 渲染 | 视觉表示、坐标映射、命中测试、行查询。 | `ITextView`、`TextViewBase`、`InteractiveTextView`、`ITextLine`、`DocumentNodes` |
| 4 | 编辑 | 处理来自键盘、鼠标或其他设备的用户输入。 | `TextSelection`、`TextEditorTyping`、`TextEditorKeyboard`、`TextEditorMouse`、`CaretElement` |
| 5 | 高亮 | 用于高亮的视觉效果，用于选择、注释、查找/替换等。 | `IHighlightLayer`、`HighlightLayerBase`、`SelectionHighlightLayer` |
| 6 | 撤销/重做 | 存储操作历史以允许撤销。 | `IUndoManager`、`UndoManager` |
| 7 | 序列化 | 以多种格式导入和导出文档（RTF、DOCX、XAML、纯文本）。 | `IDocumentSerializer`、`DocumentSnapshot`、`FlowDocumentBuilder` |
| 8 | 面向用户的控件 | 将所有层集成到模板化的 Avalonia 控件中。 | `RichTextEditor`、`FlowDocument`、`Block` 元素、`Inline` 元素 |

## 另请参阅

- [文档查看器](/controls/input/text-input/richtexteditor/document-viewer) — 只读 `FlowDocumentScrollViewer` 设置
- [工具栏和选择弹窗](/controls/input/text-input/richtexteditor/toolbar) — 自定义工具栏、迷你栏和上下文菜单
- [扩展模式](/controls/input/text-input/richtexteditor/extension-patterns) — 自定义节点、高亮层、序列化器、组件
- [性能调优](/controls/input/text-input/richtexteditor/performance-tuning)
- [故障排除](/troubleshooting/controls/richtexteditor)
