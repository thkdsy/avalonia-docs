---
id: clipboard
title: 剪贴板
description: 了解 WPF 剪贴板 API 在 XPF 中如何跨 Windows、macOS 和 Linux 工作，包括对文本、位图和自定义数据格式的支持。
doc-type: guide
---

## 概述

XPF 在所有平台上实现了 WPF 剪贴板 API（`System.Windows.Clipboard`）。不过，它与 WPF 在 Windows 上的原生实现存在一些差异，需要你注意。

## 基本用法

标准的 WPF 剪贴板操作在 XPF 中都可正常使用。你可以使用 `Clipboard.SetText` 和 `Clipboard.GetText` 复制和获取文本，或者通过 `DataObject` 处理更丰富的数据：

```csharp
// 文本
Clipboard.SetText("Hello, World!");
string text = Clipboard.GetText();

// 数据对象
var data = new DataObject();
data.SetData(DataFormats.Text, "Hello");
Clipboard.SetDataObject(data);
```

在尝试读取之前，你也可以先检查剪贴板是否包含特定格式：

```csharp
if (Clipboard.ContainsText())
{
    string text = Clipboard.GetText();
}
```

## 位图支持

从 XPF 1.6.0 开始，支持将位图复制到剪贴板以及从剪贴板读取位图：

```csharp
// 将位图复制到剪贴板
Clipboard.SetImage(myBitmapSource);

// 从剪贴板获取位图
BitmapSource image = Clipboard.GetImage();
```

在 macOS 上，使用系统快捷键（Cmd+Shift+Ctrl+3）捕获的屏幕截图可能使用与 WPF 约定不同的像素格式。XPF 1.6.0 及更高版本会自动将这些格式转换为兼容格式。

## 自定义数据格式

自定义剪贴板数据格式在同一进程内可正常工作。对于使用自定义数据的跨进程剪贴板操作，XPF 会将你的数据序列化为字符串。请确保你的自定义数据类型可序列化：

```csharp
var data = new DataObject();
data.SetData("MyCustomFormat", mySerializableObject);
Clipboard.SetDataObject(data);
```

要获取自定义数据，请使用 `Clipboard.GetDataObject`，并通过相同的格式字符串调用 `GetData`：

```csharp
IDataObject clipboardData = Clipboard.GetDataObject();
if (clipboardData?.GetDataPresent("MyCustomFormat") == true)
{
    var result = clipboardData.GetData("MyCustomFormat");
}
```

## STA 线程（Windows）

在 Windows 上，剪贴板操作使用 COM，并且要求主线程标记为 STA。如果你遇到消息为 `CoInitialize was not called` 的 `COMException`，请确保你的入口点带有 `[STAThread]` 特性：

```csharp
[STAThread]
static void Main(string[] args)
{
    // 你的应用程序启动
}
```

或者，你也可以使用 [自定义初始化](/xpf/configuration/customizing-initialization)，它会自动处理 STA 线程。

## 平台差异

下表总结了各平台对剪贴板功能的支持情况：

| 功能 | Windows | macOS | Linux |
|---|---|---|---|
| 文本 | 支持 | 支持 | 支持 |
| 位图 | 支持（1.6.0+） | 支持（1.6.0+） | 支持（1.6.0+） |
| 自定义格式（同一进程） | 支持 | 支持 | 支持 |
| 自定义格式（跨进程） | 支持（1.6.0+） | 支持（1.6.0+） | 支持（1.6.0+） |
| `Clipboard.Flush()` | 支持 | 无效果 | 支持（X11） |

`Clipboard.Flush()` 会持久化剪贴板数据，使其在应用关闭后仍可继续使用。在不支持刷新功能的平台上，该方法不会执行任何操作。

## 另请参阅

- [已知差异](/xpf/migration/known-differences)
- [自定义初始化](/xpf/configuration/customizing-initialization)
- [故障排除](/xpf/troubleshooting)