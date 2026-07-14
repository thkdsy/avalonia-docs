---
id: file-dialogs
title: 文件对话框
description: 在 Avalonia 应用中使用 StorageProvider API 打开文件选择、保存文件和文件夹选择对话框。
doc-type: guide
---

文件对话框功能通过 [`StorageProvider`](/docs/services/storage/storage-provider) 服务 API 提供，你可以从 `Window` 或 `TopLevel` 类中访问它。本页只展示最基础的用法；若想了解这个 API 的更多细节，请访问存储提供程序页面。

<GitHubSampleLink title="文件对话框" link="https://github.com/AvaloniaUI/AvaloniaUI.QuickGuides/tree/main/FileOps"/>

## OpenFilePickerAsync

该方法会打开文件选择对话框，让用户选择一个文件。`FilePickerOpenOptions` 用于定义传递给操作系统对话框的选项。

```csharp
public class MyView : UserControl
{
    private async void OpenFileButton_Clicked(object sender, RoutedEventArgs args)
    {
        // 从当前控件获取 top level。也可以改用 Window 引用。
        var topLevel = TopLevel.GetTopLevel(this);

        // 启动异步操作以打开对话框。
        var files = await topLevel.StorageProvider.OpenFilePickerAsync(new FilePickerOpenOptions
        {
            Title = "打开文本文件",
            AllowMultiple = false
        });

        if (files.Count >= 1)
        {
            // 打开第一个文件的读取流。
            await using var stream = await files[0].OpenReadAsync();
            using var streamReader = new StreamReader(stream);
            // 将文件全部内容读取为文本。
            var fileContent = await streamReader.ReadToEndAsync();
        }
    }
}
```

---

## SaveFilePickerAsync

该方法会打开文件保存对话框，让用户保存一个文件。`FilePickerSaveOptions` 用于定义传递给操作系统对话框的选项。

### 示例

```csharp
public class MyView : UserControl
{
    private async void SaveFileButton_Clicked(object sender, RoutedEventArgs args)
    {
        // 从当前控件获取 top level。也可以改用 Window 引用。
        var topLevel = TopLevel.GetTopLevel(this);

        // 启动异步操作以打开对话框。
        var file = await topLevel.StorageProvider.SaveFilePickerAsync(new FilePickerSaveOptions
        {
            Title = "保存文本文件"
        });

        if (file is not null)
        {
            // 打开文件的写入流。
            await using var stream = await file.OpenWriteAsync();
            using var streamWriter = new StreamWriter(stream);
            // 向文件写入一些内容。
            await streamWriter.WriteLineAsync("Hello World!");
        }
    }
}
```

---

## SaveFilePickerWithResultAsync

该方法的行为类似 `SaveFilePickerAsync`，但还会额外返回用户选择了哪个文件类型筛选器。当文件扩展名取决于用户选择时（例如导出为 PNG 或 JPEG），这一点非常有用。

### 示例

```csharp
public class MyView : UserControl
{
    private async void ExportButton_Clicked(object sender, RoutedEventArgs args)
    {
        var topLevel = TopLevel.GetTopLevel(this);

        var result = await topLevel.StorageProvider.SaveFilePickerWithResultAsync(new FilePickerSaveOptions
        {
            Title = "导出图片",
            FileTypeChoices = new[]
            {
                new FilePickerFileType("PNG 图片") { Patterns = new[] { "*.png" } },
                new FilePickerFileType("JPEG 图片") { Patterns = new[] { "*.jpg", "*.jpeg" } },
            }
        });

        if (result.StorageFile is not null)
        {
            // result.SelectedFileType 包含用户选择的筛选器
            var format = result.SelectedFileType?.Name; // 例如 "PNG 图片"
            await using var stream = await result.StorageFile.OpenWriteAsync();
            // 按所选格式保存……
        }
    }
}
```

返回的 `SaveFilePickerResult` 结构体包含：

| 属性 | 类型 | 说明 |
|---|---|---|
| `StorageFile` | `IStorageFile?` | 保存后的文件；如果用户取消，则为 `null`。 |
| `SelectedFileType` | `FilePickerFileType?` | 用户在对话框中选择的文件类型筛选器。 |

如果你想了解 StorageProvider 服务的更多内容，例如如何持续保留对已选择文件的访问权限，以及支持哪些可选项，请参阅 [`StorageProvider`](/docs/services/storage/storage-provider) 文档页及其子页面。

:::note
为便于学习，这些示例直接在 ViewModel 中访问了 [`StorageProvider`](/docs/services/storage/storage-provider) API。在真实项目中，建议遵循 MVVM 原则，通过创建服务类并结合依赖注入 / 控制反转（DI/IoC）来使用它。有关示例，请参阅 [IoCFileOps](https://github.com/AvaloniaUI/AvaloniaUI.QuickGuides/tree/main/IoCFileOps) 和 DepInject 项目。
:::

## 另请参阅

- [`StorageProvider`](/docs/services/storage/storage-provider)：完整的存储提供程序 API 参考。
- [文件选择器选项](/docs/services/storage/file-picker-options)：配置文件类型筛选器和对话框选项。
- [书签](/docs/services/storage/bookmarks)：持久化对已选文件和文件夹的访问权限。















