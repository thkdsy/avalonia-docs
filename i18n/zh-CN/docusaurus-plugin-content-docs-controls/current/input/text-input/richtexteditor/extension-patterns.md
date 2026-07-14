---
id: extension-patterns
title: 扩展模式
doc-type: guide
tags:
 - avalonia pro
 - avalonia enterprise
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

RichTextEditor 被设计为在多个层次上支持扩展。本指南涵盖无需修改核心代码即可扩展功能的模式。

:::info
此控件作为 [Avalonia Pro](https://avaloniaui.net/pricing) 或更高版本的一部分提供。
:::

## 扩展点

1. **自定义文档元素**——新的块/内联类型
2. **自定义高亮层**——查找、拼写检查、注释
3. **自定义序列化格式**——HTML、Markdown 等
4. **自定义编辑器组件**——新的输入处理器
5. **分组撤销操作**——通过 `IUndoManager.BeginUndoUnit`（自定义 `IUndoUnit` 子类不是公共扩展点）

## 自定义文档元素

自定义文档元素需要三个部分：

1. **元素类**——模型类型（扩展 `RichSpan`、`RichHyperlink`、`Section` 等）
2. **快照节点**——通过快照/撤销往返保留自定义数据
3. **处理器**——创建元素、捕获快照和恢复格式

在启动时通过 `TextDocumentNodeKind.Register` 注册每个元素：

```csharp
using Avalonia.Controls.Documents.TextModel;

public static class CustomNodeRegistration
{
    public static TextDocumentNodeKind CalloutBlockKind { get; private set; }
    public static TextDocumentNodeKind MentionInlineKind { get; private set; }

    public static void Register()
    {
        CalloutBlockKind = TextDocumentNodeKind.Register(
            "CalloutBlock",
            NodeKindFlags.Block | NodeKindFlags.BlockContainer,
            typeof(CalloutBlock),
            new CalloutBlockHandler());

        MentionInlineKind = TextDocumentNodeKind.Register(
            "MentionInline",
            NodeKindFlags.Inline,
            typeof(MentionInline),
            new MentionHandler());
    }
}
```

### 创建自定义内联元素

此示例创建一个 `MentionInline`，它扩展了 `RichHyperlink`，因此提及可以免费获得指针悬停效果、点击处理和工具提示。处理器将 `NavigateUri` 设置为 `mention:{userId}` URI——处理编辑器上的 `RequestNavigate` 以拦截点击。

<Tabs>
<TabItem value="element" label="元素">

```csharp
using Avalonia.Controls.Documents;

public class MentionInline : RichHyperlink
{
    public string? UserId { get; set; }
    public string? DisplayName { get; set; }
}
```

</TabItem>
<TabItem value="snapshot" label="快照节点">

通过撤销和序列化保留 `UserId` / `DisplayName`：

```csharp
using Avalonia.Controls.Documents.TextModel;
using Avalonia.Controls.Documents.Serialization.Snapshot;

public class MentionSnapshotNode : InlineSnapshotNode
{
    public string? UserId { get; }
    public string? DisplayName { get; }

    public MentionSnapshotNode(
        TextDocumentNodeKind kind,
        int startOffset,
        int length,
        InlineFormatting inlineFormatting,
        SnapshotNodeChildren children,
        TextElementFormatting textElementFormatting,
        string? userId,
        string? displayName)
        : base(kind, startOffset, length, inlineFormatting, children, textElementFormatting)
    {
        UserId = userId;
        DisplayName = displayName;
    }
}
```

</TabItem>
<TabItem value="handler" label="处理器">

```csharp
using Avalonia.Controls.Documents;
using Avalonia.Controls.Documents.TextModel;
using Avalonia.Controls.Documents.TextModel.Handlers;
using Avalonia.Controls.Documents.Serialization.Snapshot;
using Avalonia.Media;

public class MentionHandler : InlineNodeKindHandler
{
    private static readonly ISolidColorBrush MentionBackground =
        new SolidColorBrush(Color.Parse("#E3F2FD"));
    private static readonly ISolidColorBrush MentionForeground =
        new SolidColorBrush(Color.Parse("#1565C0"));

    public override RichTextElement? CreateElement(TextDocumentNodeKind kind)
    {
        var mention = new MentionInline();
        ApplyDefaultStyle(mention);
        return mention;
    }

    protected override SnapshotNode CreateInlineSnapshot(
        TextDocumentNodeKind kind, int startOffset, int length,
        InlineFormatting inlineFormatting, SnapshotNodeChildren children,
        TextElementFormatting textElementFormatting,
        RichTextElement? element, SnapshotNode? deferredSnapshot)
    {
        string? userId = null;
        string? displayName = null;

        if (deferredSnapshot is MentionSnapshotNode ms)
        {
            userId = ms.UserId;
            displayName = ms.DisplayName;
        }
        if (element is MentionInline mention)
        {
            userId = mention.UserId;
            displayName = mention.DisplayName;
        }

        return new MentionSnapshotNode(
            kind, startOffset, length,
            inlineFormatting, children, textElementFormatting,
            userId, displayName);
    }

    public override void ApplyFormatting(RichTextElement element, SnapshotNode snapshotNode)
    {
        base.ApplyFormatting(element, snapshotNode);

        if (element is MentionInline mention && snapshotNode is MentionSnapshotNode ms)
        {
            mention.UserId = ms.UserId;
            mention.DisplayName = ms.DisplayName;
            ApplyDefaultStyle(mention);
        }
    }

    public static void ApplyDefaultStyle(MentionInline mention)
    {
        mention.Background = MentionBackground;
        mention.Foreground = MentionForeground;
        mention.FontWeight = FontWeight.SemiBold;

        if (mention.UserId is { } userId)
        {
            mention.NavigateUri = new Uri($"mention:{userId}");
            mention.ToolTip = mention.DisplayName is { } name
                ? $"@{name} ({userId})"
                : $"@{userId}";
        }
    }
}
```

</TabItem>
</Tabs>

**用法：**

```csharp
var mention = new MentionInline { UserId = "alice", DisplayName = "Alice" };
mention.Inlines.Add(new RichRun { Text = "@Alice" });
MentionHandler.ApplyDefaultStyle(mention);
paragraph.Inlines.Add(mention);
```

### 创建自定义块元素

此示例创建一个 `CalloutBlock`，它扩展了 `Section`（一个块容器），并提供了一个自定义的 `StackLayoutNode` 子类，用于绘制彩色强调条和着色背景。`CalloutType` 枚举控制配色方案。

<Tabs>
<TabItem value="element" label="元素">

```csharp
using Avalonia.Controls.Documents;

public enum CalloutType { Note, Warning, Tip, Important }

public class CalloutBlock : Section
{
    public CalloutType Type { get; set; }
}
```

</TabItem>
<TabItem value="snapshot" label="快照节点">

```csharp
using Avalonia.Controls.Documents.TextModel;
using Avalonia.Controls.Documents.Serialization.Snapshot;

public class CalloutSnapshotNode : BlockSnapshotNode
{
    public CalloutType CalloutType { get; }

    public CalloutSnapshotNode(
        TextDocumentNodeKind kind,
        int startOffset,
        int length,
        BlockFormatting blockFormatting,
        SnapshotNodeChildren children,
        TextElementFormatting textElementFormatting,
        CalloutType calloutType)
        : base(kind, startOffset, length, blockFormatting, children, textElementFormatting)
    {
        CalloutType = calloutType;
    }
}
```

</TabItem>
<TabItem value="docnode" label="DocumentNode">

带强调条和着色背景的自定义渲染：

```csharp
using Avalonia;
using Avalonia.Controls.Documents;
using Avalonia.Controls.Documents.Primitives.DocumentNodes;
using Avalonia.Media;

public class CalloutDocumentNode : StackLayoutNode
{
    private const double AccentBarWidth = 4;
    private readonly CalloutBlock _callout;

    public CalloutDocumentNode(CalloutBlock callout) : base(callout)
    {
        _callout = callout;
    }

    protected override IEnumerable<RichTextElement> GetEnumerable() => _callout.Blocks;

    public override void Render(DrawingContext context)
    {
        var bounds = new Rect(Bounds.Size);
        context.FillRectangle(GetBackgroundBrush(_callout.Type), bounds);
        context.FillRectangle(GetAccentBrush(_callout.Type),
            new Rect(0, 0, AccentBarWidth, bounds.Height));
    }

    private static ISolidColorBrush GetAccentBrush(CalloutType type) => type switch
    {
        CalloutType.Note     => new SolidColorBrush(Color.Parse("#1976D2")),
        CalloutType.Warning  => new SolidColorBrush(Color.Parse("#F57C00")),
        CalloutType.Tip      => new SolidColorBrush(Color.Parse("#388E3C")),
        CalloutType.Important => new SolidColorBrush(Color.Parse("#D32F2F")),
        _ => new SolidColorBrush(Color.Parse("#757575"))
    };

    private static ISolidColorBrush GetBackgroundBrush(CalloutType type) => type switch
    {
        CalloutType.Note     => new SolidColorBrush(Color.Parse("#E3F2FD")),
        CalloutType.Warning  => new SolidColorBrush(Color.Parse("#FFF3E0")),
        CalloutType.Tip      => new SolidColorBrush(Color.Parse("#E8F5E9")),
        CalloutType.Important => new SolidColorBrush(Color.Parse("#FFEBEE")),
        _ => new SolidColorBrush(Color.Parse("#F5F5F5"))
    };
}
```

</TabItem>
<TabItem value="handler" label="处理器">

```csharp
using Avalonia;
using Avalonia.Controls.Documents;
using Avalonia.Controls.Documents.Primitives.DocumentNodes;
using Avalonia.Controls.Documents.TextModel;
using Avalonia.Controls.Documents.TextModel.Handlers;
using Avalonia.Controls.Documents.Serialization.Snapshot;

public class CalloutBlockHandler : BlockNodeKindHandler
{
    public override RichTextElement? CreateElement(TextDocumentNodeKind kind)
    {
        return new CalloutBlock { Padding = new Thickness(12, 6, 6, 6) };
    }

    protected override SnapshotNode CreateBlockSnapshot(
        TextDocumentNodeKind kind, int startOffset, int length,
        BlockFormatting blockFormatting, SnapshotNodeChildren children,
        TextElementFormatting textElementFormatting,
        RichTextElement? element, SnapshotNode? deferredSnapshot)
    {
        var calloutType = CalloutType.Note;

        if (deferredSnapshot is CalloutSnapshotNode cs)
            calloutType = cs.CalloutType;
        if (element is CalloutBlock callout)
            calloutType = callout.Type;

        return new CalloutSnapshotNode(
            kind, startOffset, length,
            blockFormatting, children, textElementFormatting,
            calloutType);
    }

    public override void ApplyFormatting(RichTextElement element, SnapshotNode snapshotNode)
    {
        base.ApplyFormatting(element, snapshotNode);

        if (element is CalloutBlock callout && snapshotNode is CalloutSnapshotNode cs)
        {
            callout.Type = cs.CalloutType;
            callout.Padding = new Thickness(12, 6, 6, 6);
        }
    }

    public override DocumentNode? CreateDocumentNode(RichTextElement element)
        => element is CalloutBlock callout ? new CalloutDocumentNode(callout) : null;
}
```

</TabItem>
</Tabs>

**用法：**

```csharp
var callout = new CalloutBlock
{
    Type = CalloutType.Warning,
    Padding = new Thickness(12, 6, 6, 6),
    Margin = new Thickness(0, 5, 0, 5)
};
var body = new Paragraph();
body.Inlines.Add(new RichRun { Text = "前方有破坏性变更。" });
callout.Blocks.Add(body);
doc.Blocks.Add(callout);
```

## 自定义高亮层

### 查找/替换高亮层

```csharp
using Avalonia.Controls.Documents.Primitives.Highlighting; // HighlightLayerBase, HighlightRegion, HighlightStyle

public class FindHighlightLayer : HighlightLayerBase
{
    public FindHighlightLayer()
        : base(name: "查找", zIndex: 50)
    {
    }

    public void HighlightMatches(IEnumerable<TextRange> matches)
    {
        ClearRegions();

        foreach (var match in matches)
        {
            var region = new HighlightRegion(
                match.Start,
                match.End,
                brush: Brushes.Yellow,
                opacity: 0.4);
            AddRegion(region);
        }

        OnRegionsChanged();
    }

    public void HighlightCurrent(TextRange current)
    {
        var region = new HighlightRegion(
            current.Start,
            current.End,
            brush: Brushes.Orange,
            opacity: 0.5);
        AddRegion(region);
        OnRegionsChanged();
    }
}
```

**集成：**

```csharp
var findLayer = new FindHighlightLayer();
editor.HighlightLayers.Add(findLayer);

// 查找匹配项
var matches = FindInDocument(searchText);
findLayer.HighlightMatches(matches);
```

### 拼写检查层

```csharp
public class SpellCheckHighlightLayer : HighlightLayerBase
{
    public SpellCheckHighlightLayer()
        : base(name: "拼写检查", zIndex: 10)
    {
    }

    public async Task CheckSpellingAsync(TextDocument document)
    {
        var errors = await RunSpellCheckAsync(document);

        // 在 UI 线程上更新高亮
        await Dispatcher.UIThread.InvokeAsync(() =>
        {
            ClearRegions();

            foreach (var error in errors)
            {
                var region = new HighlightRegion(
                    document.CreatePointer(error.Offset),
                    document.CreatePointer(error.Offset + error.Length),
                    brush: Brushes.Red,
                    style: HighlightStyle.WavyUnderline);
                AddRegion(region);
            }

            OnRegionsChanged();
        });
    }

    private Task<List<SpellError>> RunSpellCheckAsync(TextDocument doc)
    {
        return Task.Run(() =>
        {
            // 此处为拼写检查逻辑
            return new List<SpellError>();
        });
    }
}
```

## 自定义序列化格式

### HTML 序列化器

```csharp
using Avalonia.Controls.Documents;
using Avalonia.Controls.Documents.TextModel;
using Avalonia.Controls.Documents.TextModel.Snapshot;

public class HtmlSerializer : IDocumentSerializer
{
    public string FormatName => "Html";
    public string FileExtension => ".html";
    public string MimeType => "text/html";

    public bool CanDeserialize(Stream stream) => true;

    public async Task<DocumentSnapshot> DeserializeAsync(
        Stream stream, CancellationToken cancellationToken = default)
    {
        using var reader = new StreamReader(stream);
        string html = await reader.ReadToEndAsync(cancellationToken);

        var builder = FlowDocumentBuilder.Create();
        ParseHtml(html, builder);
        var doc = builder.Build();
        var textDoc = doc.EnsureTextDocument();
        return textDoc.CreateSnapshot();
    }

    public Task SerializeAsync(
        DocumentSnapshot snapshot, Stream stream,
        CancellationToken cancellationToken = default)
    {
        using var writer = new StreamWriter(stream);
        writer.WriteLine("<html><body>");

        foreach (var child in snapshot.Root.Children)
        {
            if (child is BlockSnapshotNode block)
                WriteBlock(block, snapshot, writer);
        }

        writer.WriteLine("</body></html>");
        return Task.CompletedTask;
    }

    private void WriteBlock(BlockSnapshotNode block,
                            DocumentSnapshot snapshot, StreamWriter writer)
    {
        writer.Write("<p>");

        foreach (var child in block.Children)
        {
            if (child is InlineSnapshotNode inline)
                WriteInline(inline, snapshot, writer);
        }

        writer.WriteLine("</p>");
    }

    private void WriteInline(InlineSnapshotNode inline,
                             DocumentSnapshot snapshot, StreamWriter writer)
    {
        var text = snapshot.GetText(inline.StartOffset, inline.Length);

        bool isBold = inline.Kind == TextDocumentNodeKind.Bold;
        bool isItalic = inline.Kind == TextDocumentNodeKind.Italic;

        if (isBold) writer.Write("<strong>");
        if (isItalic) writer.Write("<em>");

        // 编码文本以防止 XSS
        writer.Write(System.Net.WebUtility.HtmlEncode(text));

        if (isItalic) writer.Write("</em>");
        if (isBold) writer.Write("</strong>");
    }

    private void ParseHtml(string html, FlowDocumentBuilder builder)
    {
        // 解析 HTML 并填充构建器
    }
}
```

**用法：**

```csharp
var serializer = new HtmlSerializer();
await using var stream = File.Create("output.html");
await editor.SaveAsync(stream, serializer);
```

## 自定义编辑器组件

### 自动完成组件

`ITextViewComponent` 接口允许创建输入处理器组件，该组件与编辑器的主机基础设施集成。

```csharp
using Avalonia.Controls.Documents.Primitives.Components; // ITextViewComponent, TextViewComponentBase
using Avalonia.Controls.Documents.Primitives; // ITextEditorHost
using Avalonia.Controls.Documents.TextModel;

public class AutoCompleteComponent : ITextViewComponent
{
    private ITextEditorHost? _host;
    private Popup? _completionPopup;
    private ListBox? _completionList;

    public bool IsAttached => _host is not null;

    public void OnAttach(ITextEditorHost host)
    {
        if (_host is not null)
            OnDetach();

        _host = host;
        _host.ContentChanged += OnContentChanged;
        InitializePopup();
    }

    public void OnDetach()
    {
        if (_host is null)
            return;

        _host.ContentChanged -= OnContentChanged;
        _host = null;
        _completionPopup = null;
    }

    private void OnContentChanged(object? sender, EventArgs e)
    {
        var selection = _host?.Selection;
        if (selection is not { IsEmpty: true }) return;

        var caretPos = selection.Start;
        string wordBeforeCaret = GetWordBeforeCaret(caretPos);

        if (wordBeforeCaret.Length >= 3)
            ShowCompletions(wordBeforeCaret);
        else
            HideCompletions();
    }

    private string GetWordBeforeCaret(TextPointer caret)
    {
        var doc = caret.TextDocument;
        if (doc == null) return string.Empty;

        int offset = caret.Offset;
        int readStart = Math.Max(0, offset - 64);
        if (readStart >= offset) return string.Empty;

        var start = doc.CreatePointer(readStart);
        var range = new TextRange(start, caret);
        string text = range.Text;

        int i = text.Length - 1;
        while (i >= 0 && char.IsLetterOrDigit(text[i]))
            i--;

        return text[(i + 1)..];
    }

    private void InsertCompletion()
    {
        if (_completionList?.SelectedItem is string completion)
        {
            var editor = _host as RichTextEditor;
            var caret = editor?.Selection?.CaretPosition;
            if (caret == null) return;

            string prefix = GetWordBeforeCaret(caret);
            var start = caret.CreatePointer(-prefix.Length);
            if (start != null)
            {
                var range = new TextRange(start, caret);
                range.Text = completion;
            }

            HideCompletions();
        }
    }

    private void ShowCompletions(string prefix) { /* ... */ }
    private void HideCompletions() { /* ... */ }
    private void InitializePopup() { /* ... */ }
}
```

**注册：**

```csharp
editor.RegisterComponent(new AutoCompleteComponent());
```

## 自定义撤销单元

### 将操作分组为单个撤销步骤

使用 `UndoManager.BeginUndoUnit` 将作用域内的所有内容记录为一个可撤销的操作：

```csharp
using Avalonia.Controls.Documents.Undo;

var undoManager = editor.UndoManager;
if (undoManager != null)
{
    using (undoManager.BeginUndoUnit("查找并全部替换"))
    {
        // 此作用域内的所有编辑操作是一个撤销步骤
        foreach (var match in matches)
        {
            match.Text = replacement;
        }
    }
}
```

`IUndoUnit` 是用于检查记录的撤销条目（只读的 `Description` 属性）的公共接口。内部的撤销/重做机制由框架处理。请使用 `BeginUndoUnit`，而不是创建自定义的撤销单元类型。

## 最佳实践

### 应做事项

1. **实现 `ITextViewComponent`**——使用附加/分离生命周期以进行正确的清理和初始扫描支持
2. **订阅 `host.UIScope` 上的输入事件**——主机本身不接收输入事件；只有 UIScope 接收
3. **对指针拦截使用 `RoutingStrategies.Tunnel`**——内置组件（如 `TextEditorMouse`）在 Bubble 阶段将事件标记为已处理；使用 Tunnel 先检查事件
4. **使用 `ITextView.GetTextPositionFromPoint` 进行命中测试**——选择状态可能已过时（尤其是在 Tunnel 期间）；直接命中测试点击点
5. **从基类继承**——使用 `HighlightLayerBase`，而不是原始的 `IHighlightLayer`
6. **优雅地处理 null**——主机、UIScope 和 TextView 在转换期间可能为 null
7. **编写单元测试**——全面测试扩展
8. **对长时间运行的操作使用异步**——不要阻塞 UI 线程

### 不应做事项

1. **不要在主机/编辑器上直接订阅事件**——使用 `host.UIScope` 通过 `AddHandler`/`RemoveHandler`
2. **不要依赖指针处理器中的选择状态**——而是对点击点进行命中测试；在 Tunnel 阶段选择尚未更新
3. **不要访问内部成员**——仅使用公共 API
4. **不要持有强文档引用**——会导致内存泄漏
5. **不要阻塞 UI 线程**——对 CPU/IO 操作使用异步
6. **不要假设文档结构**——访问前进行验证
7. **不要绕过撤销系统**——始终记录可撤销的操作
8. **不要忘记分离**——清理事件处理器

## 完整示例：智能链接检测

此组件检测文档中的 URL，用蓝色下划线高亮它们，并支持 Ctrl+Click 打开链接。演示的关键模式：

- **`ITextViewComponent` 生命周期**——在附加时（现有内容）和每次后续文本或文档更改时进行扫描
- **`host.UIScope`**——在 UIScope 上订阅指针事件，而不是主机本身，因为只有 UIScope 接收输入事件
- **`RoutingStrategies.Tunnel`**——在 Tunnel 阶段订阅，以便处理器在 `TextEditorMouse` 在 Bubble 阶段将事件标记为已处理之前触发
- **`ITextView` 命中测试**——使用 `GetTextPositionFromPoint` 将点击位置解析为 `TextPointer`；在 Tunnel 阶段选择状态已过时

```csharp
// 通过 editor.RegisterComponent(new SmartLinkExtension()) 注册。
// 通过 editor.UnregisterComponent(component) 注销。

public class SmartLinkExtension : ITextViewComponent
{
    private ITextEditorHost? _host;
    private readonly LinkHighlightLayer _linkLayer = new();
    private readonly List<DetectedLink> _links = new();

    public bool IsAttached => _host is not null;

    public event Action<Uri>? LinkActivated;

    public void OnAttach(ITextEditorHost host)
    {
        if (_host is not null)
            OnDetach();

        _host = host;

        if (_host is RichTextEditor editor)
            editor.HighlightLayers.Add(_linkLayer);

        _host.ContentChanged += OnTextChanged;
        _host.DocumentChanged += OnDocumentChanged;

        // 在 UIScope（接收输入事件的元素）上订阅
        // 使用 Tunnel，以便我们在 TextEditorMouse 的 Bubble 处理器之前触发。
        _host.UIScope?.AddHandler(
            InputElement.PointerPressedEvent,
            OnPointerPressed,
            RoutingStrategies.Tunnel);

        // 立即扫描现有内容
        _ = DetectLinksAsync();
    }

    public void OnDetach()
    {
        if (_host is null)
            return;

        _host.ContentChanged -= OnTextChanged;
        _host.DocumentChanged -= OnDocumentChanged;

        _host.UIScope?.RemoveHandler(
            InputElement.PointerPressedEvent,
            OnPointerPressed);

        if (_host is RichTextEditor editor)
            editor.HighlightLayers.Remove(_linkLayer);

        _links.Clear();
        _linkLayer.ClearHighlights();
        _host = null;
    }

    private async void OnTextChanged(object? sender, EventArgs e)
        => await DetectLinksAsync();

    private async void OnDocumentChanged(object? sender, EventArgs e)
        => await DetectLinksAsync();

    private async Task DetectLinksAsync()
    {
        var doc = _host?.TextDocument;
        if (doc is null) return;

        var start = doc.CreatePointer(0);
        var end = doc.CreatePointer(doc.Length);
        string text = new TextRange(start, end).Text;

        var found = await Task.Run(() => FindUrls(text));

        await Dispatcher.UIThread.InvokeAsync(() =>
        {
            // 在等待期间防范分离或文档交换
            if (_host is null) return;
            var currentDoc = _host.TextDocument;
            if (currentDoc != doc) return;

            _links.Clear();
            _linkLayer.ClearHighlights();

            foreach (var (offset, length, uri) in found)
            {
                _links.Add(new DetectedLink(
                    offset, offset + length, uri));
                _linkLayer.AddLink(
                    currentDoc.CreatePointer(offset),
                    currentDoc.CreatePointer(offset + length));
            }

            _linkLayer.RaiseChanged();
        });
    }

    private void OnPointerPressed(object? sender, PointerPressedEventArgs e)
    {
        var host = _host;
        if (host is null) return;

        var uiScope = host.UIScope;
        var textView = host.TextView;
        if (uiScope is null || textView is null) return;

        if (!e.KeyModifiers.HasFlag(KeyModifiers.Control)) return;
        if (!e.GetCurrentPoint(uiScope).Properties.IsLeftButtonPressed) return;

        // 命中测试点击点——不要依赖选择状态，
        // 在 Tunnel 阶段选择状态尚未更新。
        var clickPoint = e.GetPosition((Visual)textView);
        var pointer = textView.GetTextPositionFromPoint(
            clickPoint, snapToText: false);
        if (pointer is null) return;

        int offset = pointer.Offset;
        var link = _links.Find(l => offset >= l.Start && offset <= l.End);

        if (link is not null)
        {
            LinkActivated?.Invoke(link.Uri);
            e.Handled = true;
        }
    }

    private record DetectedLink(int Start, int End, Uri Uri);
}
```

## 测试扩展

```csharp
public class MentionInlineTests
{
    [Fact]
    public void MentionInline_PreservesUserIdThroughSnapshot()
    {
        // 安排——注册自定义类型
        var kind = TextDocumentNodeKind.Register(
            "TestMention", NodeKindFlags.Inline,
            typeof(MentionInline), new MentionHandler());

        var doc = new FlowDocument();
        var para = new Paragraph();
        var mention = new MentionInline { UserId = "alice", DisplayName = "Alice" };
        mention.Inlines.Add(new RichRun { Text = "@Alice" });
        MentionHandler.ApplyDefaultStyle(mention);
        para.Inlines.Add(mention);
        doc.Blocks.Add(para);

        var textDoc = doc.EnsureTextDocument();

        // 操作——快照往返
        var snapshot = textDoc.CreateSnapshot();
        var restored = snapshot.ToFlowDocument();
        var restoredDoc = restored.EnsureTextDocument();

        // 断言
        var restoredMention = restoredDoc.Root
            .DescendantsOfKind(kind)
            .First().Element as MentionInline;

        Assert.NotNull(restoredMention);
        Assert.Equal("alice", restoredMention.UserId);
        Assert.Equal("Alice", restoredMention.DisplayName);
    }
}
```

## 另请参阅

- [RichTextEditor 控件](/controls/input/text-input/richtexteditor)
- [性能调优](/controls/input/text-input/richtexteditor/performance-tuning)
- [线程安全](/controls/input/text-input/richtexteditor/thread-safety)
