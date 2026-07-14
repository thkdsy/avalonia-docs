---
id: richtexteditor
title: RichTextEditor 问题
description: 排查常见的 RichTextEditor 问题。
doc-type: troubleshooting
sidebar_label: RichTextEditor
tags:
  - avalonia pro
  - avalonia enterprise
---

:::info
此控件作为 [Avalonia Pro](https://avaloniaui.net/pricing) 或更高版本的一部分提供。
:::

有关此控件的参考信息，请参阅 [RichTextEditor](/controls/input/text-input/richtexteditor) 页面。

## Document 为空

确保已设置 `FlowDocument` 以供编辑器访问。

```csharp
if (editor.Document == null)
{
    editor.Document = new FlowDocument();
}
```

## 撤销功能无效

确保已在代码隐藏中创建 `UndoManager`。

```csharp
editor.Document.TextDocument.UndoManager = new UndoManager();
```

## 编辑内容未显示在编辑器中

尝试将多个操作包装为批量编辑。

```csharp
doc.BeginChange();
try
{
    // 你的编辑操作
}
finally
{
    doc.EndChange();
}
```

## 调试线程问题

### 启用线程断言

Avalonia 内置了线程检查功能：

```csharp
// 如果不在 UI 线程上则抛出异常
Dispatcher.UIThread.VerifyAccess();
```

### 常见异常

**InvalidOperationException**："调用线程无法访问此对象，因为另一个线程拥有它。"
- **原因**：从后台线程访问 UI 线程对象
- **修复**：使用 `Dispatcher.UIThread.InvokeAsync()`

**NullReferenceException**：访问 `Element` 时
- **原因**：弱引用已被回收或后台线程访问
- **修复**：检查 null 并确保在 UI 线程上

## 我需要其他帮助

## 另请参阅

- [RichTextEditor 控件](/controls/input/text-input/richtexteditor)
