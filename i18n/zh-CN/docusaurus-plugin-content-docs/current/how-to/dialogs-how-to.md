---
id: dialogs-how-to
title: "如何：使用对话框"
description: 在 Avalonia 中创建和显示模态对话框、返回结果，并构建自定义对话框窗口。
doc-type: how-to
---

本指南介绍如何创建和显示模态对话框、返回结果，以及构建自定义对话框窗口。

## 显示对话框窗口

创建一个对话框窗口，并使用 `ShowDialog<T>` 以模态方式显示：

```csharp
var dialog = new ConfirmDialog();
dialog.DataContext = new ConfirmDialogViewModel("Delete this item?");

// ShowDialog 会在对话框关闭时返回结果
bool? result = await dialog.ShowDialog<bool?>(parentWindow);

if (result == true)
{
    DeleteItem();
}
```

`parentWindow` 参数用于设置所有者窗口。在桌面平台上，对话框会居中显示在所有者窗口之上，并在关闭前阻止用户与该窗口交互。

## 创建对话框窗口

对话框本质上就是一个带有若干常见设置的普通 `Window`：

```xml
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        x:Class="MyApp.Views.ConfirmDialog"
        Title="Confirm"
        Width="400" Height="200"
        WindowStartupLocation="CenterOwner"
        CanResize="False"
        ShowInTaskbar="False">
    <Grid RowDefinitions="*,Auto" Margin="20">
        <TextBlock Grid.Row="0" Text="{Binding Message}"
                   TextWrapping="Wrap" VerticalAlignment="Center" />

        <StackPanel Grid.Row="1" Orientation="Horizontal"
                    HorizontalAlignment="Right" Spacing="8">
            <Button Content="Cancel" Command="{Binding CancelCommand}" />
            <Button Content="OK" Command="{Binding ConfirmCommand}" />
        </StackPanel>
    </Grid>
</Window>
```

### 带结果关闭对话框

你可以使用 `Window.Close(result)` 关闭对话框并返回一个值：

```csharp
public partial class ConfirmDialog : Window
{
    public ConfirmDialog()
    {
        InitializeComponent();
    }
}
```

```csharp
public partial class ConfirmDialogViewModel : ObservableObject
{
    private readonly Window _dialog;

    public string Message { get; }

    public ConfirmDialogViewModel(Window dialog, string message)
    {
        _dialog = dialog;
        Message = message;
    }

    [RelayCommand]
    private void Confirm() => _dialog.Close(true);

    [RelayCommand]
    private void Cancel() => _dialog.Close(false);
}
```

初始化对话框及其视图模型：

```csharp
var dialog = new ConfirmDialog();
var vm = new ConfirmDialogViewModel(dialog, "Delete this item?");
dialog.DataContext = vm;

bool? result = await dialog.ShowDialog<bool?>(this);
```

### 另一种方式：在 code-behind 中关闭

如果你更希望把关闭逻辑放在视图层中：

```csharp
public partial class ConfirmDialog : Window
{
    public ConfirmDialog()
    {
        InitializeComponent();
    }

    private void OnOkClick(object? sender, RoutedEventArgs e)
    {
        Close(true);
    }

    private void OnCancelClick(object? sender, RoutedEventArgs e)
    {
        Close(false);
    }
}
```

## 返回复杂结果

对话框可以返回任意对象：

```csharp
// 返回所选颜色的对话框
var dialog = new ColorPickerDialog();
Color? selectedColor = await dialog.ShowDialog<Color?>(parentWindow);

if (selectedColor is not null)
{
    ApplyColor(selectedColor.Value);
}
```

在对话框内部：

```csharp
[RelayCommand]
private void Select()
{
    _dialog.Close(SelectedColor);
}

[RelayCommand]
private void Cancel()
{
    _dialog.Close(null);
}
```

## 获取父窗口

如果你是在视图模型或 UserControl 中显示对话框，而手头没有父窗口的直接引用，可以这样做：

```csharp
// 在 UserControl 的 code-behind 中
var window = TopLevel.GetTopLevel(this) as Window;
if (window is not null)
{
    var result = await dialog.ShowDialog<bool?>(window);
}
```

如果是在视图模型中，通常通过服务或参数把窗口传入：

```csharp
public interface IDialogService
{
    Task<bool> ShowConfirmAsync(string message);
    Task<string?> ShowInputAsync(string prompt);
}

public class DialogService : IDialogService
{
    private readonly Window _mainWindow;

    public DialogService(Window mainWindow)
    {
        _mainWindow = mainWindow;
    }

    public async Task<bool> ShowConfirmAsync(string message)
    {
        var dialog = new ConfirmDialog();
        dialog.DataContext = new ConfirmDialogViewModel(dialog, message);
        var result = await dialog.ShowDialog<bool?>(_mainWindow);
        return result == true;
    }
}
```

## 文件和文件夹对话框

使用 `IStorageProvider` 服务来打开文件和文件夹选择对话框：

```csharp
var topLevel = TopLevel.GetTopLevel(this);
if (topLevel is null) return;

var storage = topLevel.StorageProvider;

// 打开文件选择器
var files = await storage.OpenFilePickerAsync(new FilePickerOpenOptions
{
    Title = "Select a File",
    AllowMultiple = false,
    FileTypeFilter = new[]
    {
        new FilePickerFileType("Text Files") { Patterns = new[] { "*.txt" } },
        new FilePickerFileType("All Files") { Patterns = new[] { "*" } },
    }
});

if (files.Count > 0)
{
    var file = files[0];
    await using var stream = await file.OpenReadAsync();
    // 读取文件内容
}
```

```csharp
// 保存文件选择器
var file = await storage.SaveFilePickerAsync(new FilePickerSaveOptions
{
    Title = "Save File",
    SuggestedFileName = "document.txt",
    FileTypeChoices = new[]
    {
        new FilePickerFileType("Text Files") { Patterns = new[] { "*.txt" } },
    }
});

if (file is not null)
{
    await using var stream = await file.OpenWriteAsync();
    // Write file contents
}
```

完整 API 请参阅 [Storage Provider](/docs/services/storage/storage-provider)。

## 阻止对话框关闭

你可以处理 `Closing` 事件来阻止对话框关闭（例如存在未保存更改时）：

```csharp
dialog.Closing += (sender, e) =>
{
    if (HasUnsavedChanges)
    {
        e.Cancel = true;
        // Optionally show a confirmation
    }
};
```

## 覆盖式对话框（窗口内）

如果你想让对话框显示在窗口内部，而不是作为独立的操作系统窗口出现，可以使用一个覆盖层面板：

```xml
<Grid>
    <!-- Main content -->
    <StackPanel Margin="20">
        <Button Content="Show Dialog" Command="{Binding ShowDialogCommand}" />
    </StackPanel>

    <!-- Dialog overlay -->
    <Border Background="#80000000"
            IsVisible="{Binding IsDialogVisible}">
        <Border Background="White" CornerRadius="8"
                HorizontalAlignment="Center" VerticalAlignment="Center"
                Padding="24" MinWidth="300" BoxShadow="0 8 16 0 #40000000">
            <StackPanel Spacing="16">
                <TextBlock Text="Confirm Action" FontWeight="Bold" FontSize="18" />
                <TextBlock Text="Are you sure?" />
                <StackPanel Orientation="Horizontal" Spacing="8"
                            HorizontalAlignment="Right">
                    <Button Content="Cancel" Command="{Binding HideDialogCommand}" />
                    <Button Content="OK" Command="{Binding ConfirmCommand}" />
                </StackPanel>
            </StackPanel>
        </Border>
    </Border>
</Grid>
```

这种方式适用于所有平台，包括不支持独立窗口的 WebAssembly。

## 另请参阅

- [Window Management](/docs/app-development/window-management): Show, ShowDialog, and window lifecycle.
- [Storage Provider](/docs/services/storage/storage-provider): File and folder picker dialogs.
- [Commanding](/docs/input-interaction/commanding): Binding buttons to commands.
