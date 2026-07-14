---
id: drag-and-drop-how-to
title: "如何：实现拖放"
description: 在 Avalonia 中发起拖拽、处理放下、提供视觉反馈，并接受文件拖放。
doc-type: how-to
---

本指南介绍常见的拖放场景：发起拖拽、处理放下、提供视觉反馈，以及接受文件拖放。

## 接受拖放文件

最常见的拖放场景之一，就是接收用户从操作系统文件管理器中拖过来的文件。

### XAML 设置

在目标元素上把 `DragDrop.AllowDrop` 设为 `True`，即可启用放下能力：

```xml
<Border Background="#F3F4F6" Padding="40"
        DragDrop.AllowDrop="True">
    <TextBlock Text="Drop files here"
               HorizontalAlignment="Center" VerticalAlignment="Center" />
</Border>
```

### Code-behind 处理器

为 `DragOver`（声明可接受的拖放效果）和 `Drop`（处理放下数据）注册处理器：

```csharp
public MainWindow()
{
    InitializeComponent();

    DragDrop.AddDropHandler(this, OnDrop);
    DragDrop.AddDragOverHandler(this, OnDragOver);
}

private void OnDragOver(object? sender, DragEventArgs e)
{
    // 只接受文件拖放；其余全部拒绝
    e.DragEffects = e.DataTransfer.Formats.Contains(DataFormat.File)
        ? DragDropEffects.Copy
        : DragDropEffects.None;
}

private void OnDrop(object? sender, DragEventArgs e)
{
    if (e.DataTransfer.GetFiles() is { } files)
    {
        foreach (var file in files)
        {
            var path = file.Path.LocalPath;
            // 处理文件
        }
    }
}
```

:::tip
务必在 `DragOver` 处理器中设置 `e.DragEffects`。否则即使你的控件实际上能接收放下，平台仍可能显示“禁止放下”的鼠标指针。
:::

## 接受拖放文本

你也可以接受纯文本拖放。使用 `TryGetText()` 获取其字符串值：

```csharp
private void OnDrop(object? sender, DragEventArgs e)
{
    if (e.DataTransfer.TryGetText() is { } text)
    {
        // 使用拖放进来的文本
        ViewModel.Content = text;
    }
}
```

## 发起拖拽操作

如果你想从自己的控件中发起拖拽（例如从列表项开始拖动），可以在指针按下处理器中调用 `DragDrop.DoDragDropAsync`：

```csharp
private async void OnPointerPressed(object? sender, PointerPressedEventArgs e)
{
    if (sender is not Control control) return;

    var dragData = new DataTransfer();
    dragData.Set(DataFormat.Text, "Dragged item text");

    var result = await DragDrop.DoDragDropAsync(e, dragData, DragDropEffects.Copy | DragDropEffects.Move);

    if (result == DragDropEffects.Move)
    {
        // 项目已被移动，应从源中移除
    }
}
```

:::warning
`DoDragDropAsync` 会捕获指针。因此不要在每次 `PointerPressed` 时都直接开始拖拽。更稳妥的做法是设置最小移动距离阈值，或者等到 `PointerMoved` 后再确认用户是真的想拖动而不是点击。
:::

## 列表之间拖放

一个常见模式，是在两个列表控件之间拖动项目。通常你需要一个处理器从源列表发起拖拽，再用另一个处理器在目标列表中接收放下。

### 源列表

```csharp
private async void SourceList_PointerPressed(object? sender, PointerPressedEventArgs e)
{
    if (sender is ListBox listBox && listBox.SelectedItem is ItemViewModel item)
    {
        var data = new DataTransfer();
        data.Set("application/x-my-item", item);

        var result = await DragDrop.DoDragDropAsync(e, data, DragDropEffects.Move);

        if (result == DragDropEffects.Move)
            ViewModel.SourceItems.Remove(item);
    }
}
```

### 目标列表

在放下处理器中，取回自定义对象并把它加入目标集合：

```csharp
private void TargetList_Drop(object? sender, DragEventArgs e)
{
    if (e.DataTransfer.Get("application/x-my-item") is ItemViewModel item)
    {
        ViewModel.TargetItems.Add(item);
        e.DragEffects = DragDropEffects.Move;
    }
}
```

## 拖拽过程中的视觉反馈

提供视觉反馈有助于用户理解哪些位置可以放下。你可以在用户拖过目标区域时，改变放置目标的外观：

```csharp
public MainWindow()
{
    InitializeComponent();

    var dropZone = this.FindControl<Border>("DropZone");

    DragDrop.AddDragEnterHandler(this, (s, e) =>
    {
        dropZone.BorderBrush = Brushes.Blue;
        dropZone.BorderThickness = new Thickness(2);
    });

    DragDrop.AddDragLeaveHandler(this, (s, e) =>
    {
        dropZone.BorderBrush = Brushes.Transparent;
        dropZone.BorderThickness = new Thickness(0);
    });

    DragDrop.AddDropHandler(this, (s, e) =>
    {
        dropZone.BorderBrush = Brushes.Transparent;
        dropZone.BorderThickness = new Thickness(0);
        // 处理放下...
    });
}
```

:::tip
请在 `DragLeave` 和 `Drop` 两个处理器中都重置视觉状态。如果你只在 `DragLeave` 中重置，那么当用户真正完成一次放下时，高亮可能会残留。
:::

## 设置拖拽光标

你可以通过设置拖拽期间显示的光标，告诉用户当前允许的操作。在 `DragOver` 处理器中设置 `e.DragEffects`：

```csharp
private void OnDragOver(object? sender, DragEventArgs e)
{
    if (e.DataTransfer.Formats.Contains(DataFormat.File))
    {
        e.DragEffects = DragDropEffects.Copy;
    }
    else
    {
        e.DragEffects = DragDropEffects.None;
    }
}
```

| DragDropEffects | 光标 | 含义 |
|---|---|---|
| `None` | 禁止放下光标 | 此处不允许放下。 |
| `Copy` | 复制光标 (+) | 项目将被复制。 |
| `Move` | 移动光标 | 项目将被移动。 |
| `Link` | 链接光标 | 将创建一个链接或快捷方式。 |

## 自定义数据格式

你可以使用字符串键传递自定义对象。建议使用 MIME 风格的标识符，以避免和其他应用冲突：

```csharp
// 设置
var data = new DataTransfer();
data.Set("application/x-my-custom-type", myObject);

// 获取
if (e.DataTransfer.Get("application/x-my-custom-type") is MyType obj)
{
    // 使用 obj
}
```

## 数据格式参考

| 格式 | 常量 | 说明 |
|---|---|---|
| Text | `DataFormat.Text` | 纯文本字符串。 |
| Bitmap | `DataFormat.Bitmap` | 位图图像数据。 |
| File | `DataFormat.File` | 文件系统项（返回 `IStorageItem` 实例）。 |
| Custom | 任意字符串键 | 应用程序自定义的任意类型数据。 |

## 边界情况与故障排查

- **Drop 处理器没有触发：** 确认目标元素上已设置 `DragDrop.AllowDrop="True"`，并且 `DragOver` 处理器把 `e.DragEffects` 设成了 `None` 以外的值。
- **单击就开始拖拽：** 在调用 `DoDragDropAsync` 前加入移动距离阈值。否则一次普通点击也可能触发拖拽，容易让用户困惑。
- **跨进程时自定义数据丢失：** 使用 `DataTransfer.Set` 设置的自定义对象类型，只能在同一个应用程序内部使用。跨进程拖放通常仅限于 `DataFormat.Text`、`DataFormat.File` 这类标准格式。
- **多种数据格式：** 你可以在同一个 `DataTransfer` 实例上多次调用 `DataTransfer.Set`，写入不同格式键。这样目标端就能选择它最能处理的那种格式。

## 平台说明

| 平台 | 支持级别 | 说明 |
|---|---|---|
| Windows | 完整支持 | 支持从资源管理器拖入文件、应用之间拖放文本和位图，以及在应用内部使用自定义格式。 |
| macOS | 完整支持 | 支持从 Finder 拖入文件，系统拖拽光标也会正确响应 `DragDropEffects`。 |
| Linux (X11/Wayland) | 完整支持 | 行为与 Windows 基本一致。部分 Wayland 合成器在光标渲染上可能略有差异。 |
| Browser (WebAssembly) | 有限支持 | 大多数浏览器支持从操作系统文件管理器拖入文件。应用内部元素之间的拖拽通常需要自定义实现，因为浏览器会处理指针捕获。 |
| iOS / Android | 不支持 | 当前不提供拖放功能。可以考虑使用长按手势或列表重排模式来实现类似交互。 |

## 另请参阅

- [Drag and Drop](/docs/input-interaction/drag-and-drop)：拖放系统的概念总览。
- [Gestures](/docs/input-interaction/gestures)：触摸与指针手势识别。
- [Storage Provider](/docs/services/storage/storage-provider)：与 `IStorageItem` 一起使用的文件访问 API。
