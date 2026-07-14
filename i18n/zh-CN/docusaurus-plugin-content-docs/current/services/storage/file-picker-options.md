---
id: file-picker-options
title: 文件选择器选项
---

## 常见选择器选项

### 标题

获取或设置显示在选择器标题栏中的文本。

### 建议起始位置

获取或设置文件打开选择器初始查找文件的位置。
这个位置可以来自先前选择过的文件夹，或者通过 `StorageProvider.TryGetFolderFromPathAsync`、`StorageProvider.TryGetWellKnownFolderAsync` 获取。

:::note
这只是对系统的一个建议。如果应用无权访问该文件夹，或者文件夹本身不存在，系统可以忽略这个参数。
:::
:::note
在 Linux 上，某些 DBus 文件选择器并不支持起始位置。如果你想使用 GTK Free Desktop 方案，请在 `X11PlatformOptions` 中禁用 `UseDBusFilePicker`。
:::

## FilePickerOpenOptions

### AllowMultiple

获取或设置一个选项，用于指示打开选择器是否允许用户选择多个文件。

### FileTypeFilter

获取或设置文件打开选择器所显示的文件类型集合。

### SuggestedFileType

获取或设置对话框在文件类型筛选下拉框中默认选中的 `FilePickerFileType`。该值必须是 `FileTypeFilter` 中的某一项。

如需为文件选择器创建文件类型列表：

```csharp
// 这同样适用于 FilePickerSaveOptions。
var files = await _target.StorageProvider.OpenFilePickerAsync(new FilePickerOpenOptions()
{
 Title = title,
// 你既可以添加自定义类型，也可以使用内置文件类型。如何创建自定义类型，请参阅“定义自定义文件类型”。
 FileTypeFilter = new[] { ImageAll, FilePickerFileTypes.TextPlain }
});
```

## FilePickerSaveOptions

### SuggestedFileName

获取或设置文件保存选择器建议给用户的文件名。

### DefaultExtension

获取或设置保存文件时使用的默认扩展名。

### FileTypeChoices

获取或设置用户可以为文件选择的合法文件类型集合。

### SuggestedFileType

获取或设置对话框在文件类型筛选下拉框中默认选中的 `FilePickerFileType`。该值必须是 `FileTypeChoices` 中的某一项。

### ShowOverwritePrompt

获取或设置一个值，用于指示当用户指定的文件名已存在时，文件保存选择器是否显示警告。

## FolderPickerOpenOptions

### AllowMultiple

获取或设置一个选项，用于指示打开选择器是否允许用户选择多个文件夹。

## 平台兼容性

| 功能 | Managed | Windows | macOS | Linux | Browser | Android | iOS |
|---------------|-------|-------|-------|-------|-------|-------|-------|
| `Title` | ✓ | ✓ | ✓ | ✓ | ✗ | ✓ | ✓ |
| `SuggestedStartLocation` | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| `AllowMultiple` | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| `FileTypeFilter` | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| `SuggestedFileType` | ✓ | ✓ | ✓ | ✓ | ✗ | ✗ | ✗ |
| `SuggestedFileName` | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✗ |
| `DefaultExtension` | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✗ |
| `FileTypeChoices` | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✗ |
| `ShowOverwritePrompt` | ✓ | ✓ | ✗ | ✓ | ✗ | ✗ | ✗ |

## 定义自定义文件类型

Avalonia 提供了一组内置文件类型：

- FilePickerFileTypes.All - 所有文件
- FilePickerFileTypes.TextPlain - txt 文件
- FilePickerFileTypes.ImageAll - 所有图像
- FilePickerFileTypes.ImageJpg - jpg 图像
- FilePickerFileTypes.ImagePng - png 图像
- FilePickerFileTypes.ImageWebP - webp 图像
- FilePickerFileTypes.Pdf - pdf 文档

当然，你也可以定义供选择器使用的自定义文件类型。

例如，内置的 `ImageAll` 类型定义如下：

```csharp
public static FilePickerFileType ImageAll { get; } = new("所有图像")
{
    Patterns = new[] { "*.png", "*.jpg", "*.jpeg", "*.gif", "*.bmp", "*.webp" },
    AppleUniformTypeIdentifiers = new[] { "public.image" },
    MimeTypes = new[] { "image/*" }
};
```

每种文件类型都可以包含以下提示信息，供不同平台使用：

- `Patterns`：用于大多数 Windows、Linux 和浏览器平台，是一种可用于匹配类型的基础 GLOB 模式。
- `AppleUniformTypeIdentifiers`：Apple 定义的标准标识符，用于 macOS 和 iOS 平台。你可以在 macOS 终端中通过 `mdls -name kMDItemContentType yourfile.ext` 查找某个文件对应的正确值。
- `MimeTypes`：文件的 Web 标识符，适用于大多数平台，但不包括 Windows 和 iOS。

如果这些信息是已知的，建议尽可能完整地定义所有提示字段。

:::note
如果你不知道某个特定提示的正确值，请不要随意填写，也不要使用 `*.*` 这样的通配符；应保持该集合为 null。这样平台会忽略这一组提示，并尝试使用其他可用信息。
:::

## 在选项中加入 WebP

请注意，`FilePickerFileTypes.ImageWebP` 以及在“所有图像”类型中加入 `*.webp`，是从 11.1 版本开始引入的。在旧版本中，你仍然可以通过自定义文件选择器类型来支持 WebP 图像。例如，如果只允许选择 WebP 图像，可以这样写：

```csharp
var customWebPFileType = new FilePickerFileType("仅 WebP 图像")
{
    Patterns = new[] { "*.webp" },
    AppleUniformTypeIdentifiers = new[] { "org.webmproject.webp" },
    MimeTypes = new[] { "image/webp" }
};
```

如果你希望把 WebP 也视作“图像”类型的一部分，可以直接参考上面展示的 `ImageAll` 示例。

## 另请参阅

- [存储提供程序](/docs/services/storage/storage-provider)：完整的存储提供程序 API 参考。
- [文件对话框](/docs/services/file-dialogs)：使用文件打开、保存和文件夹选择对话框。
- [书签](/docs/services/storage/bookmarks)：持久化对已选文件和文件夹的访问权限。
