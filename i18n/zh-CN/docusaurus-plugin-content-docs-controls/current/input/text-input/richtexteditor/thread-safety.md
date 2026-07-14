---
id: thread-safety
title: 线程安全
doc-type: guide
tags:
 - avalonia pro
 - avalonia enterprise
---

RichTextEditor 架构使用不可变快照进行后台安全的序列化，同时要求 UI 线程访问实时文档操作。本指南说明线程模型和安全模式。

:::info
此控件作为 [Avalonia Pro](https://avaloniaui.net/pricing) 或更高版本的一部分提供。
:::

## 线程模型

### 需要 UI 线程

以下操作必须在 UI 线程上运行：
- **文档编辑**（`TextDocument`、`TextPointer`、`TextRange`）
- **渲染**（`TextViewBase`、`InteractiveTextView`、`ITextView`）
- **用户交互**（`TextSelection`、组件）
- **撤销/重做操作**
- **元素访问**（`FlowDocument`、`RichTextElement` 及任何子元素——它们是 Avalonia `StyledElement`）

### 后台线程安全

以下操作可以在后台线程上运行：
- **通过 `DocumentSnapshot` 进行序列化**（不可变）
- **`IDocumentSerializer.SerializeAsync` / `DeserializeAsync`**——按接口约定仅为异步
- **RTF 分词**（流式）
- **`DocumentSnapshot` 消费**——`TextDocument.CreateSnapshot()` 必须在 UI 线程上调用，但返回的对象可以从任何线程安全读取

### 不安全的操作

- **实时的 `TextDocument`**——不支持并发修改
- **UI 元素访问**——Avalonia 控件不是线程安全的
- **`TextPointer`/`TextRange`**——绑定到 UI 线程文档
- **`FlowDocumentBuilder`**——仅限 UI 线程（生成的 `FlowDocument` 本身是一个 `StyledElement`）

## 安全模式

### 后台序列化

```csharp
async Task SaveAsync(string path)
{
    // SaveAsync 在内部创建快照并在后台线程上序列化
    await using var stream = File.Create(path);
    await editor.SaveAsync(stream, new RtfSerializer());
}
```

如果需要手动控制快照（例如，用于自定义格式）：

```csharp
async Task SaveManualAsync(string path)
{
    // UI 线程：通过 FlowDocument 快照文档
    var doc = editor.Document;
    if (doc == null) return;

    await using var stream = File.Create(path);
    await doc.SaveAsync(stream, new RtfSerializer());
}
```

### 后台反序列化

```csharp
async Task LoadAsync(string path)
{
    // LoadAsync 反序列化并构建文档
    await using var stream = File.OpenRead(path);
    await editor.LoadAsync(stream, new RtfSerializer());
}
```

或者独立于编辑器加载：

```csharp
async Task LoadStandaloneAsync(string path)
{
    await using var stream = File.OpenRead(path);
    var document = await FlowDocument.LoadAsync(stream, new RtfSerializer());

    // 在 UI 线程上分配
    editor.Document = document;
}
```

### 后台文档处理

```csharp
async Task<string> ExtractPlainTextAsync()
{
    // 通过公共 TextRange API 读取文本（UI 线程）
    string? text = editor.Document?.ContentRange?.Text;
    if (text == null) return string.Empty;

    // 后台线程：处理提取的文本
    return await Task.Run(() =>
    {
        return ProcessText(text);
    });
}
```

## 不安全模式

### 不要从后台线程访问实时文档

```csharp
// 错误——将抛出异常
await Task.Run(() =>
{
    var doc = editor.Document?.TextDocument;
    string? text = editor.Document?.ContentRange?.Text; // 异常
});
```

### 不要从后台线程修改文档

```csharp
// 错误——将抛出异常
await Task.Run(() =>
{
    document.ContentStart.InsertText("Hello"); // 异常
});
```

### 不要从后台线程访问 UI 元素

```csharp
// 错误——将抛出异常
await Task.Run(() =>
{
    // FlowDocument 和 RichTextElement 是 Avalonia StyledElement；
    // 从后台线程访问它们的属性会抛出异常。
    var firstParagraph = flowDocument.Blocks.FirstOrDefault();
    var background = firstParagraph?.Background; // 异常
});
```

## DocumentSnapshot 设计

### 不可变结构

`DocumentSnapshot` 是为线程安全而设计的：
- **不可变**——创建后不能修改
- **无 UI 引用**——纯数据结构
- **共享节点**——与实时文档高效共享内存
- **自包含**——所有数据从实时文档复制

### 快照层次结构

```
DocumentSnapshot（线程安全）
├─ BlockSnapshotNode
│  ├─ InlineSnapshotNode
│  └─ InlineSnapshotNode
└─ BlockSnapshotNode
   └─ InlineSnapshotNode
```

所有节点都是不可变的值类型或只读结构。

### 创建快照

快照创建是序列化器使用的内部机制。后台序列化的公共 API 是通过 `SaveAsync`/`LoadAsync`：

```csharp
// 使用异步 API 保存（内部处理快照）
await using var stream = File.Create("output.rtf");
await editor.SaveAsync(stream, new RtfSerializer());
```

## FlowDocumentBuilder

`FlowDocumentBuilder` 提供了构建文档的流畅 API。它在 UI 线程上运行：

```csharp
var builder = FlowDocumentBuilder.Create();
builder.AddParagraph("第一段");
builder.AddParagraph("第二段");
var document = builder.Build();

editor.Document = document;
```

## 同步策略

### Dispatcher 模式

```csharp
async Task UpdateFromBackgroundAsync()
{
    // 后台工作
    var data = await FetchDataAsync();

    // 切换到 UI 线程
    await Dispatcher.UIThread.InvokeAsync(() =>
    {
        UpdateDocument(data);
    });
}
```

### async/await 模式

```csharp
async Task SaveAndProcessAsync(string path)
{
    // 通过异步序列化器在后台保存
    await using var stream = File.Create(path);
    await editor.SaveAsync(stream, new RtfSerializer());

    // await 后自动回到 UI 线程
    ShowSaveComplete();
}
```

## 常见场景

### 在后台线程进行拼写检查

```csharp
class SpellChecker
{
    public async Task<List<SpellError>> CheckAsync()
    {
        // UI 线程：获取全文
        string? text = editor.Document?.ContentRange?.Text;
        if (string.IsNullOrEmpty(text)) return new List<SpellError>();

        // 后台：检查拼写
        return await Task.Run(() =>
        {
            var errors = new List<SpellError>();
            // 对提取的文本运行拼写检查算法
            return errors;
        });
    }

    public async Task ApplyCorrectionsAsync(
        TextDocument document,
        List<SpellError> errors)
    {
        // UI 线程：应用修正
        await Dispatcher.UIThread.InvokeAsync(() =>
        {
            using (document.BeginChange())
            {
                foreach (var error in errors)
                {
                    var start = document.ContentStart.CreatePointer(error.Offset);
                    var end = document.ContentStart.CreatePointer(error.Offset + error.Length);
                    var range = new TextRange(start, end);
                    range.Text = error.Correction;
                }
            }
        });
    }
}
```

### 在后台进行字数统计

```csharp
async Task<int> CountWordsAsync()
{
    // UI 线程：通过公共 API 获取文本
    string? text = editor.Document?.ContentRange?.Text;
    if (string.IsNullOrEmpty(text)) return 0;

    // 后台：计数
    return await Task.Run(() =>
    {
        return text.Split(new[] { ' ', '\n', '\r', '\t' },
                         StringSplitOptions.RemoveEmptyEntries).Length;
    });
}
```

### 在后台线程导出为 PDF

```csharp
async Task ExportPdfAsync(string path)
{
    // UI 线程：获取完整文档文本
    string? text = editor.Document?.ContentRange?.Text;
    if (text == null) return;

    // 后台：生成 PDF
    await Task.Run(() =>
    {
        var generator = new PdfGenerator();
        generator.GenerateFromText(text, path);
    });
}
```

## 弱引用和线程安全

### 为什么使用弱引用？

`TextDocumentNode` 对 UI 元素使用弱引用：
- 防止内存泄漏
- 允许垃圾回收
- 无强耦合

### 线程安全影响

节点到元素的查找在内部管理。通过公共 API，始终在 UI 线程上访问 `FlowDocument` 元素：

```csharp
// 必须在 UI 线程上
var firstBlock = flowDocument.Blocks.FirstOrDefault();
if (firstBlock != null)
{
    var background = firstBlock.Background;
}
```

## 最佳实践

### 应做事项

1. **在 UI 线程上创建快照**——快速操作
2. **在后台线程上处理快照**——安全且高效
3. **返回 UI 线程进行文档更新**——使用 Dispatcher
4. **在 UI 操作前检查线程**——防御性编程
5. **使用 async/await 编写清晰代码**——自然的线程切换

### 不应做事项

1. **不要从后台线程访问实时文档**
2. **不要从后台线程修改文档**
3. **不要从后台线程访问 UI 元素**
4. **不要假设快照会自动更新**——它们是不可变的
5. **不要持有 UI 元素的长期引用**——使用弱引用

## 性能考虑

### 快照创建成本

- **小文档（<10KB）**：约 1ms
- **大文档（1MB）**：约 10ms
- **影响**：对于后台操作可忽略不计

### 线程切换成本

- **Dispatcher 调用**：约 1-2ms 开销
- **建议**：批量执行 UI 更新，不要每次字符都切换

### 最佳模式

```csharp
// 差：太多次线程切换
await Task.Run(async () =>
{
    for (int i = 0; i < 1000; i++)
    {
        await Dispatcher.UIThread.InvokeAsync(() =>
        {
            UpdateUI(i); // 1000 次调度
        });
    }
});

// 好：一次切换
var results = await Task.Run(() =>
{
    var items = new List<Item>();
    for (int i = 0; i < 1000; i++)
    {
        items.Add(ProcessItem(i));
    }
    return items;
});

await Dispatcher.UIThread.InvokeAsync(() =>
{
    UpdateUIBatch(results); // 1 次调度
});
```

## 另请参阅

- [RichTextEditor 控件](/controls/input/text-input/richtexteditor)
- [性能调优](/controls/input/text-input/richtexteditor/performance-tuning)
- [扩展模式](/controls/input/text-input/richtexteditor/extension-patterns)
