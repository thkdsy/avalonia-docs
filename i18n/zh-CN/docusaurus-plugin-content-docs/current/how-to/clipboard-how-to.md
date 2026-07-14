---
id: clipboard-how-to
title: "如何：使用剪贴板"
description: 使用 Avalonia 剪贴板 API 复制和粘贴文本、图像以及自定义数据。
doc-type: how-to
---

本指南将介绍如何使用 Avalonia 剪贴板 API 来复制和粘贴文本、图像以及自定义数据。你将学习如何获取剪贴板引用、传输常见数据类型、注册自定义格式，以及绑定键盘快捷键。

## 获取剪贴板

你需要通过 `TopLevel` 访问剪贴板。在 code-behind 中，可以对树中的任意视觉对象调用 `GetTopLevel`：

```csharp
var clipboard = TopLevel.GetTopLevel(this)?.Clipboard;
```

如果你需要在视图模型中访问剪贴板，建议通过构造函数注入 `IClipboard`，这样视图模型依然保持可测试性：

```csharp
public class MainViewModel
{
    private readonly IClipboard? _clipboard;

    public MainViewModel(IClipboard? clipboard)
    {
        _clipboard = clipboard;
    }
}
```

:::tip
所有剪贴板方法都是异步的，因为底层平台 API 可能需要用户授权，或者需要进行跨进程通信。请始终使用 `await` 调用，并处理可能出现的 `null` 返回值。
:::

## 复制文本到剪贴板

使用 `SetTextAsync` 把纯文本字符串放入剪贴板：

```csharp
[RelayCommand]
private async Task CopyText()
{
    var clipboard = TopLevel.GetTopLevel(this)?.Clipboard;
    if (clipboard is not null)
    {
        await clipboard.SetTextAsync("Hello, clipboard!");
    }
}
```

## 从剪贴板粘贴文本

使用 `TryGetTextAsync` 读取纯文本。如果当前没有文本可用，该方法会返回 `null`：

```csharp
[RelayCommand]
private async Task PasteText()
{
    var clipboard = TopLevel.GetTopLevel(this)?.Clipboard;
    if (clipboard is not null)
    {
        var text = await clipboard.TryGetTextAsync();
        if (text is not null)
        {
            Content = text;
        }
    }
}
```

## 检查剪贴板内容

在执行粘贴之前，你可以先查询剪贴板当前包含哪些格式。当应用支持多种数据类型，并且你希望优先选取最合适的格式时，这非常有用：

```csharp
var clipboard = TopLevel.GetTopLevel(this)?.Clipboard;
if (clipboard is not null)
{
    using var data = await clipboard.TryGetDataAsync();
    if (data is not null)
    {
        if (data.Formats.Contains(DataFormat.Text))
        {
            var text = await data.TryGetTextAsync();
            // 使用文本值
        }
    }
}
```

:::note
`TryGetDataAsync` 返回的 `DataTransfer` 对象是可释放的。请用 `using` 语句包裹它，以便及时释放平台资源。
:::

## 复制图像到剪贴板

加载一个 `Bitmap`，然后把它传给 `SetBitmapAsync`：

```csharp
var clipboard = TopLevel.GetTopLevel(this)?.Clipboard;
if (clipboard is not null)
{
    var bitmap = new Bitmap("assets/photo.png");
    await clipboard.SetBitmapAsync(bitmap);
}
```

## 从剪贴板粘贴图像

使用 `TryGetBitmapAsync` 获取图像。如果没有图像数据可用，该方法会返回 `null`：

```csharp
var clipboard = TopLevel.GetTopLevel(this)?.Clipboard;
if (clipboard is not null)
{
    var bitmap = await clipboard.TryGetBitmapAsync();
    if (bitmap is not null)
    {
        MyImage.Source = bitmap;
    }
}
```

## 复制自定义数据

使用 `DataTransfer` 和 `DataTransferItem` 可以把结构化数据放入剪贴板。通常你需要先创建一个带唯一标识符的应用级格式，再把一种或多种表示形式写入 `DataTransferItem`。同时包含一个 `DataFormat.Text` 项，可以为其他应用提供纯文本回退内容：

```csharp
var myFormat = DataFormat.CreateBytesApplicationFormat("mycompany-myapp-mydata");

var item = new DataTransferItem();
item.Set(DataFormat.Text, "Plain text fallback");
item.Set(myFormat, mySerializedBytes);

var data = new DataTransfer();
data.Add(item);

await clipboard.SetDataAsync(data);
```

## 粘贴自定义数据

如果要读取你自己的自定义格式，需要创建相同的 `DataFormat`，然后调用 `TryGetValueAsync`：

```csharp
var myFormat = DataFormat.CreateBytesApplicationFormat("mycompany-myapp-mydata");

using var data = await clipboard.TryGetDataAsync();
if (data is not null)
{
    var bytes = await data.TryGetValueAsync(myFormat);
    if (bytes is not null)
    {
        // 将字节数组反序列化为你的对象
    }
}
```

:::tip
复制和粘贴两端必须使用相同的格式标识符字符串（`"mycompany-myapp-mydata"`）。剪贴板正是通过这个标识符来把数据匹配到你的应用格式上的。
:::

## 键盘快捷键

标准剪贴板快捷键（`Ctrl+C`、`Ctrl+V`、`Ctrl+X`）在 `TextBox`、`TextPresenter` 等内置文本控件中会自动生效。对于自定义控件，则需要显式地把按键手势绑定到命令上：

```xml
<UserControl.KeyBindings>
    <KeyBinding Gesture="Ctrl+C" Command="{Binding CopyCommand}" />
    <KeyBinding Gesture="Ctrl+V" Command="{Binding PasteCommand}" />
    <KeyBinding Gesture="Ctrl+X" Command="{Binding CutCommand}" />
</UserControl.KeyBindings>
```

在 macOS 上，当你在 XAML 中指定 `Ctrl` 修饰键时，Avalonia 会自动映射为 `Cmd+C`、`Cmd+V` 和 `Cmd+X`，因此通常不需要再写平台专属绑定。

## 平台说明

剪贴板 API 在所有 Avalonia 目标平台上都可用，但并不是每个平台都支持所有数据类型。下表总结了当前支持情况：

| 平台 | 文本 | 图像 | 文件 | 自定义格式 |
|---|---|---|---|---|
| Windows | 支持 | 支持 | 支持 | 支持 |
| macOS | 支持 | 支持 | 支持 | 支持 |
| Linux | 支持 | 支持 | 视桌面环境而定 | 支持 |
| Browser (WASM) | 支持（需要授权） | 支持 | 不支持 | 有限制 |
| iOS | 支持 | 支持 | 不支持 | 有限制 |
| Android | 支持 | 只读 | 不支持 | 有限制 |

在 **Browser/WASM** 上，浏览器可能会在应用第一次调用剪贴板方法时请求用户授权。你的代码应当处理用户拒绝授权、方法返回 `null` 的情况。

在 **Linux** 上，文件剪贴板支持取决于桌面环境及其剪贴板管理器。文本和图像操作通常可以在 GNOME、KDE 等主流环境中可靠运行。

## 另请参阅

- [Clipboard service](/docs/services/clipboard)
- [Drag and drop how-to](/docs/how-to/drag-and-drop-how-to)
- [Hotkeys](/docs/input-interaction/keyboard-and-hotkeys)
