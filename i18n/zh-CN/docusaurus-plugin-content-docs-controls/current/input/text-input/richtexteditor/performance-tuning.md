---
id: performance-tuning
title: 性能调优
doc-type: guide
tags:
 - avalonia pro
 - avalonia enterprise
---

`RichTextEditor` 的性能调优指南。涵盖批量编辑、事件优化、内存管理、序列化和性能分析策略。

:::info
此控件作为 [Avalonia Pro](https://avaloniaui.net/pricing) 或更高版本的一部分提供。
:::

## 核心性能特性

### 时间复杂度

| 操作 | 复杂度 | 说明 |
|-----------|-----------|-------|
| 插入文本 | O(log n) | 绳索数据结构 |
| 删除文本 | O(log n) | 平衡树更新 |
| 查找位置 | O(log n) | 树遍历 |
| 撤销/重做 | O(1) - O(log n) | 结构撤销 |
| 序列化 | O(n) | 流式分词器 |
| 渲染 | O(visible nodes) | 视口剔除 |

### 内存使用

- **文档**：O(n) 文本 + O(m) 节点
- **绳索开销**：约 2 倍基础文本
- **撤销栈**：结构撤销约 10%（对比传统方式 100 倍）
- **快照**：共享结构，最小开销

## 性能检查清单

- 批量处理所有多编辑操作
- 对耗时操作使用 `UpdateFinished` 而不是 `Changed`
- 对用户触发的更新进行防抖
- 在编辑器上设置适当的 `UndoLimit`
- 在批量加载期间禁用撤销
- 在后台线程上执行序列化
- 最小化指针分配
- 优化前先进行分析

## 批量编辑优化

### 始终批量处理多个操作

单个更改通知代替每次编辑都通知：

```csharp
// 差——100 次 Changed 事件，100 次布局传递
for (int i = 0; i < 100; i++)
{
    pointer.InsertText("第 " + i + " 行\n");
}

// 好——1 次 UpdateFinished 事件，1 次布局传递
using (document.BeginChange())
{
    for (int i = 0; i < 100; i++)
    {
        pointer.InsertText("第 " + i + " 行\n");
    }
}
```

**影响**：批量操作加速 10-100 倍。

## 事件处理器优化

### 延迟耗时操作

使用 `UpdateFinished` 代替响应每次编辑：

```csharp
// 差——每次按键都会调用
editor.ContentChanged += (s, e) =>
{
    RebuildUI();
};

// 好——每批调用一次
var textDoc = editor.Document?.TextDocument;
textDoc.UpdateFinished += (s, e) =>
{
    RebuildUI();
};
```

### 防抖用户触发的更新

```csharp
private DispatcherTimer _updateTimer;

void Setup()
{
    _updateTimer = new DispatcherTimer
    {
        Interval = TimeSpan.FromMilliseconds(300)
    };
    _updateTimer.Tick += OnDelayedUpdate;

    editor.ContentChanged += (s, e) =>
    {
        _updateTimer.Stop();
        _updateTimer.Start(); // 重启计时器
    };
}

void OnDelayedUpdate(object? sender, EventArgs e)
{
    _updateTimer.Stop();
    // 耗时操作（字数统计、拼写检查等）
    UpdateStatistics();
}
```

**影响**：在连续输入期间减少 CPU 使用率。

## 指针和范围优化

### 最小化指针分配

```csharp
// 差——为每个字符索引创建指针
for (int i = 0; i < 1000; i++)
{
    var p = document.ContentStart.GetPositionAtOffset(i); // 每次调用 O(log n)
}

// 好——快照一次并批量读取文本
var snapshot = document.CreateSnapshot();
string slice = snapshot.GetText(offset: 0, length: 1000);
for (int i = 0; i < slice.Length; i++)
{
    char c = slice[i]; // 直接数组访问
}
```

### 尽可能复用指针

```csharp
var pointer = document.ContentStart;
for (int i = 0; i < 100; i++)
{
    pointer.InsertText("行\n");
    // pointer 自动更新到插入位置之后
}
```

## 内存管理

### 撤销栈限制

```csharp
// 默认：100 次操作（通过 RichTextEditor.UndoLimit 设置）
editor.UndoLimit = 50;  // 为内存受限环境减少限制
editor.UndoLimit = 200; // 为高级用户增加限制
```

**权衡**：内存与撤销历史深度。

### 为批量加载禁用撤销

```csharp
void LoadLargeDocument(string rtfPath)
{
    var undoManager = editor.UndoManager;
    if (undoManager != null)
        undoManager.IsEnabled = false;

    try
    {
        using var stream = File.OpenRead(rtfPath);
        editor.Load(stream, new RtfSerializer());
    }
    finally
    {
        if (undoManager != null)
            undoManager.IsEnabled = true;
    }
}
```

**影响**：加载速度提升 50%，无撤销内存开销。

### 需要时清除撤销历史

```csharp
// 保存文档后
editor.ClearUndoHistory();
```

## 序列化性能

### 使用后台线程

```csharp
async Task SaveDocumentAsync(string path)
{
    // SaveAsync 内部处理快照创建
    await using var stream = File.Create(path);
    await editor.SaveAsync(stream, new RtfSerializer());
}
```

**影响**：保存期间不阻塞 UI。

### 流式处理大文件

```csharp
// 流式分词器高效处理大文件
await using var stream = File.OpenRead("large.rtf");
await editor.LoadAsync(stream, new RtfSerializer());
// 内存使用：O(输出大小)，而非 O(文件大小)
```

## 渲染性能

### 视口剔除

内置功能：仅渲染可见元素。无需操作。

### 减少布局传递

```csharp
// 批量格式化更改
using (document.BeginChange())
{
    range1.ApplyPropertyValue(prop1, value1);
    range2.ApplyPropertyValue(prop2, value2);
    range3.ApplyPropertyValue(prop3, value3);
}
// 单次布局传递
```

### 简化复杂文档

- 限制嵌套深度（< 10 层）
- 合并具有相同格式的相邻运行
- 使用元数据规范化

## 大型文档策略

### 测试限制

- 10,000+ 段落
- 1MB+ RTF 文件
- 100+ 撤销操作

### 对于非常大的文档（100MB+）

考虑：
1. **分页**——按需加载章节
2. **虚拟滚动**——仅渲染可见页面
3. **只读模式**——通过 `FlowDocumentScrollViewer` 禁用撤销以节省内存
4. **流式处理**——分块处理

## 基准测试

### 内置基准测试

```bash
cd benchmarks/Avalonia.Controls.Documents.Benchmarks
dotnet run -c Release
```

基准测试涵盖：
- 文本插入/删除
- 批量编辑
- 序列化
- 元数据规范化

### 自定义基准测试

```csharp
using BenchmarkDotNet.Attributes;
using BenchmarkDotNet.Running;

[MemoryDiagnoser]
public class CustomBenchmark
{
    private TextDocument _document;

    [GlobalSetup]
    public void Setup()
    {
        _document = new TextDocument("初始文本");
    }

    [Benchmark]
    public void BulkInsert()
    {
        var pointer = _document.ContentStart;
        _document.BeginChange();
        for (int i = 0; i < 1000; i++)
        {
            pointer.InsertText("X");
        }
        _document.EndChange();
    }
}

BenchmarkRunner.Run<CustomBenchmark>();
```

## 反模式

### 不要轮询文档状态

```csharp
// 差：轮询
var timer = new DispatcherTimer { Interval = TimeSpan.FromMilliseconds(100) };
timer.Tick += (s, e) => CheckDocumentState();

// 好：事件驱动
var textDoc = editor.Document?.TextDocument;
if (textDoc != null)
    textDoc.UpdateFinished += (s, e) => UpdateState();
```

### 不要在每次按键时重建 UI

```csharp
// 差
editor.ContentChanged += (s, e) => RebuildEntireUI();

// 好
var textDoc = editor.Document?.TextDocument;
if (textDoc != null)
{
    textDoc.UpdateFinished += (s, e) =>
    {
        if (e.HasChanges)
            RefreshAffectedRegions();
    };
}
```

### 不要为撤销存储完整的文本副本

```csharp
// 差：通过完整文本撤销
undoStack.Push(editor.Document?.ContentRange?.Text ?? "");

// 好：内置 UndoManager（编辑器自动创建）
editor.UndoLimit = 100;
```

## 性能分析提示

### 使用诊断工具

**Windows**：Visual Studio 性能分析器
**macOS/Linux**：dotnet-trace、PerfView

### 需要监控的热点路径

1. `TextDocument.InsertText/DeleteText`
2. `RopeTextStore` 操作
3. `FlowDocumentView` 中的布局
4. 事件处理器（`Changed`、`UpdateFinished`）

### 危险信号

- 自定义事件处理器中的 O(n^2) 算法
- 过度分配（简单编辑超过 1MB）
- 布局抖动（每次编辑多次布局传递）
- 无限制的撤销增长

## 另请参阅

- [RichTextEditor 控件](/controls/input/text-input/richtexteditor)
- [线程安全](/controls/input/text-input/richtexteditor/thread-safety)
- [扩展模式](/controls/input/text-input/richtexteditor/extension-patterns)
