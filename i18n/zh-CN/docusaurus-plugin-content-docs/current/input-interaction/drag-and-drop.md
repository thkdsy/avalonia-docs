---
id: drag-and-drop
title: 拖放
---

Avalonia 支持拖放操作，可用于在控件之间，或在你的应用与操作系统之间传输数据。拖放系统使用 [`DragDrop`](/api/avalonia/input/dragdrop) 静态类和 [`DataTransfer`](/api/avalonia/input/datatransfer) 类型来管理操作过程中的数据。

## 在目标上启用放置

若要接收放下的内容，元素必须将 `DragDrop.AllowDrop` 附加属性设为 `True`，并处理相关拖放事件：

```xml
<Border DragDrop.AllowDrop="True"
        Background="LightGray" Padding="40"
        DragEnter="OnDragEnter"
        DragLeave="OnDragLeave"
        DragOver="OnDragOver"
        Drop="OnDrop">
    <TextBlock Text="将文件拖放到这里" HorizontalAlignment="Center" />
</Border>
```

## 拖放事件

| 事件 | 触发时机 |
|---|---|
| `DragEnter` | 拖动过程中，指针进入目标元素时触发。 |
| `DragLeave` | 拖动过程中，指针离开目标元素时触发。 |
| `DragOver` | 拖动过程中，指针在目标元素上方移动时持续触发。 |
| `Drop` | 用户在目标元素上方释放指针时触发。 |

所有这些事件都会提供一个 `DragEventArgs`，其中包含以下属性：

| 属性 | 说明 |
|---|---|
| `DataTransfer` | 包含拖动数据的 `IDataTransfer` 对象。 |
| `DragEffects` | 允许和请求的拖放效果。你可以设置它来表明目标接受哪种操作。 |
| `KeyModifiers` | 当前键盘修饰键（Ctrl、Shift、Alt）。 |
| `GetPosition(Visual)` | 返回相对于指定可视元素的指针位置。 |

## 处理放置事件

```csharp
private void OnDragOver(object? sender, DragEventArgs e)
{
    // 检查是否可以接受该数据
    if (e.DataTransfer.Formats.Contains(DataFormat.File))
    {
        e.DragEffects = DragDropEffects.Copy;
    }
    else
    {
        e.DragEffects = DragDropEffects.None;
    }
}

private void OnDrop(object? sender, DragEventArgs e)
{
    if (e.DataTransfer.Formats.Contains(DataFormat.File))
    {
        var files = e.DataTransfer.GetFiles();
        if (files != null)
        {
            foreach (var file in files)
            {
                // 处理每个放下的文件
                Debug.WriteLine($"已放下：{file.Name}");
            }
        }
    }
}
```

## DragDropEffects

`DragDropEffects` 标志枚举用于表示允许哪些操作：

| 值 | 说明 |
|---|---|
| `None` | 放置目标不接受该数据。 |
| `Copy` | 数据被复制到目标。 |
| `Move` | 数据被移动到目标。 |
| `Link` | 创建指向原始数据的链接。 |

你可以在 `DragOver` 中设置 `e.DragEffects` 来控制光标反馈，也可以在 `Drop` 中设置它来表示操作结果。

## 开始拖动操作

若要从你的控件中发起拖放操作，请在响应指针事件时调用 `DragDrop.DoDragDropAsync`。首先创建一个包含待拖动数据的 `DataTransfer` 对象：

```csharp
private async void OnPointerPressed(object? sender, PointerPressedEventArgs e)
{
    var dragData = new DataTransfer();
    dragData.Set(DataFormat.Text, "来自拖动的问候！");

    var result = await DragDrop.DoDragDropAsync(
        e,
        dragData,
        DragDropEffects.Copy | DragDropEffects.Move);

    // result 表示放置目标执行了什么操作
    if (result == DragDropEffects.Move)
    {
        // 如果是移动操作，则移除源数据
    }
}
```

:::info
`DoDragDropAsync` 是异步方法。它会在用户完成或取消拖动操作时返回。返回值表示放置目标实际应用了哪种效果。
:::

## DataTransfer 与数据格式

`DataTransfer` 类是一个可变的数据容器，用于保存拖放过程中的数据。标准格式请使用 `DataFormat` 的静态属性：

| 格式 | 类型 | 说明 |
|---|---|---|
| `DataFormat.Text` | `string` | 纯文本。 |
| `DataFormat.Bitmap` | `Bitmap` | 位图图像数据。 |
| `DataFormat.File` | `IStorageItem` | 文件系统项。 |

你也可以创建自定义格式：

```csharp
var myFormat = DataFormat.CreateStringApplicationFormat("myapp-item");
```

### 设置数据

```csharp
var data = new DataTransfer();
data.Add(DataTransferItem.Create(DataFormat.Text, "Some text"));
```

### 读取数据

```csharp
// 在 DragOver 或 Drop 处理器中
if (e.DataTransfer.Formats.Contains(DataFormat.Text))
{
    var text = e.DataTransfer.TryGetText();
}

if (e.DataTransfer.Formats.Contains(DataFormat.File))
{
    var files = e.DataTransfer.GetFiles();
}
```

## 拖动过程中的视觉反馈

可以使用 `DragEnter` 和 `DragLeave` 事件来提供视觉反馈：

```csharp
private void OnDragEnter(object? sender, DragEventArgs e)
{
    if (sender is Border border)
    {
        border.BorderBrush = Brushes.Blue;
        border.BorderThickness = new Thickness(2);
    }
}

private void OnDragLeave(object? sender, DragEventArgs e)
{
    if (sender is Border border)
    {
        border.BorderBrush = null;
        border.BorderThickness = new Thickness(0);
    }
}
```

## 完整示例

下面的示例创建了一个可接收文本和文件的放置区域：

```xml title="XAML"
<Border x:Name="DropZone"
        DragDrop.AllowDrop="True"
        Background="#F5F5F5" CornerRadius="8"
        Padding="40" Margin="20"
        BorderBrush="DarkGray" BorderThickness="1">
    <StackPanel Spacing="8" HorizontalAlignment="Center">
        <TextBlock Text="将文本或文件拖放到这里"
                   HorizontalAlignment="Center" />
        <TextBlock x:Name="StatusText" Foreground="Gray"
                   HorizontalAlignment="Center" />
    </StackPanel>
</Border>
```

```csharp title="Code-behind"
public partial class MainWindow : Window
{
    public MainWindow()
    {
        InitializeComponent();

        DragDrop.AddDragOverHandler(DropZone, OnDragOver);
        DragDrop.AddDropHandler(DropZone, OnDrop);
        DragDrop.AddDragEnterHandler(DropZone, OnDragEnter);
        DragDrop.AddDragLeaveHandler(DropZone, OnDragLeave);
    }

    private void OnDragEnter(object? sender, DragEventArgs e)
    {
        DropZone.Background = Brushes.LightBlue;
    }

    private void OnDragLeave(object? sender, DragEventArgs e)
    {
        DropZone.Background = new SolidColorBrush(Color.Parse("#F5F5F5"));
    }

    private void OnDragOver(object? sender, DragEventArgs e)
    {
        e.DragEffects = e.DataTransfer.Formats.Contains(DataFormat.Text)
                     || e.DataTransfer.Formats.Contains(DataFormat.File)
            ? DragDropEffects.Copy
            : DragDropEffects.None;
    }

    private void OnDrop(object? sender, DragEventArgs e)
    {
        DropZone.Background = new SolidColorBrush(Color.Parse("#F5F5F5"));

        if (e.DataTransfer.Formats.Contains(DataFormat.Text))
        {
            StatusText.Text = $"已放下文本：{e.DataTransfer.TryGetText()}";
        }
        else if (e.DataTransfer.Formats.Contains(DataFormat.File))
        {
            var files = e.DataTransfer.GetFiles();
            if (files != null)
            {
                StatusText.Text = $"已放下 {files.Count()} 个文件";
            }
        }
    }
}
```

## 在 XAML 与代码中处理事件

拖放事件是定义在 `DragDrop` 类上的附加事件。你可以像上面那样在 XAML 中使用事件属性语法处理它们，也可以在代码中显式注册：

```csharp
// 在代码中注册
DragDrop.AddDropHandler(myBorder, OnDrop);

// 移除处理器
DragDrop.RemoveDropHandler(myBorder, OnDrop);
```

`DragDrop` 类为每个事件都提供了 `Add*Handler` 和 `Remove*Handler` 静态方法，包括 `DragEnter`、`DragLeave`、`DragOver` 和 `Drop`。

## 另请参阅

- [指针事件](/docs/input-interaction/pointer)：检测用于发起拖动的指针移动。
- [剪贴板](/docs/services/clipboard)：通过剪贴板共享数据。
- [存储提供程序](/docs/services/storage/storage-provider)：处理文件和文件夹。
