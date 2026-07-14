---
id: storage-provider
title: 存储提供程序
---

`StorageProvider` 是文件和文件夹管理的核心服务。它提供文件/文件夹选择、平台能力检查以及与已保存书签交互的方法。

你可以通过 `TopLevel` 或 `Window` 的实例访问 `StorageProvider`。关于如何访问 `TopLevel`，请参阅 [TopLevel](/docs/fundamentals/top-level) 页面。

```csharp
var storage = window.StorageProvider;
```

## 属性

### CanOpen
表示当前平台是否支持打开“文件选择器”。

```csharp
bool CanOpen { get; }
```

### CanSave
表示当前平台是否支持打开“保存文件选择器”。

```csharp
bool CanSave { get; }
```

### CanPickFolder
表示当前平台是否支持打开“文件夹选择器”。

```csharp
bool CanPickFolder { get; }
```

## 方法

### OpenFilePickerAsync
打开文件选择对话框。

```csharp
Task<IReadOnlyList<IStorageFile>> OpenFilePickerAsync(FilePickerOpenOptions options);
```
该方法返回用户所选的 `IStorageFile` 实例数组；如果用户取消对话框，则返回空集合。

### SaveFilePickerAsync
打开保存文件对话框。

```csharp
Task<IStorageFile?> SaveFilePickerAsync(FilePickerSaveOptions options);
```
该方法返回保存后的 `IStorageFile` 实例；如果用户取消对话框，则返回 null。

### SaveFilePickerWithResultAsync
打开保存文件对话框，并同时返回所选文件和用户选中的文件类型筛选器。

```csharp
Task<SaveFilePickerResult> SaveFilePickerWithResultAsync(FilePickerSaveOptions options);
```
该方法返回一个 `SaveFilePickerResult` 结构体。其中 `StorageFile` 属性包含保存后的文件（若已取消则为 `null`），`SelectedFileType` 则包含用户在对话框中选择的 `FilePickerFileType`。

### OpenFolderPickerAsync
打开文件夹选择对话框。

```csharp
Task<IReadOnlyList<IStorageFolder>> OpenFolderPickerAsync(FolderPickerOpenOptions options);
```
该方法返回用户所选的 `IStorageFolder` 实例数组；如果用户取消对话框，则返回空集合。

### OpenFileBookmarkAsync
通过书签 ID 打开一个 `IStorageBookmarkFile`。

```csharp
Task<IStorageBookmarkFile?> OpenFileBookmarkAsync(string bookmark);
```
如果操作系统允许访问，该方法会返回一个已书签化的文件；如果请求被拒绝，则返回 null。

### OpenFolderBookmarkAsync
通过书签 ID 打开一个 `IStorageBookmarkFolder`。

```csharp
Task<IStorageBookmarkFolder?> OpenFolderBookmarkAsync(string bookmark);
```
如果操作系统允许访问，该方法会返回一个已书签化的文件夹；如果请求被拒绝，则返回 null。

### TryGetFileFromPathAsync
尝试根据路径从文件系统中读取一个文件。

```csharp
Task<IStorageFile?> TryGetFileFromPathAsync(Uri filePath);
```
如果文件存在，则该方法返回文件；如果不存在，则返回 null。`filePath` 参数通常应为带有 `file` 协议的绝对路径，但在 Android 上也可以是带有 `content` 协议的 URI。

### TryGetFolderFromPathAsync
尝试根据路径从文件系统中读取一个文件夹。

```csharp
Task<IStorageFolder?> TryGetFolderFromPathAsync(Uri folderPath);
```
如果文件夹存在，则该方法返回文件夹；如果不存在，则返回 null。`folderPath` 参数通常应为带有 `file` 协议的绝对路径，但在 Android 上也可以是带有 `content` 协议的 URI。

### TryGetWellKnownFolderAsync
尝试根据众所周知的文件夹标识符从文件系统中读取一个文件夹。

```csharp
Task<IStorageFolder?> TryGetWellKnownFolderAsync(WellKnownFolder wellKnownFolder);
```
如果文件夹存在，则该方法返回文件夹；如果不存在，则返回 null。

## 扩展方法

### TryGetFileFromPathAsync
尝试根据路径从文件系统中读取一个文件。

```csharp
Task<IStorageFile?> TryGetFileFromPathAsync(this IStorageProvider provider, string filePath);
```
如果文件存在，则该方法返回文件；如果不存在，则返回 null。
该方法接收一个不带协议的本地文件路径字符串作为参数。
它只在支持物理文件路径的操作系统上受支持，主要是桌面平台。

### TryGetFolderFromPathAsync
尝试根据路径从文件系统中读取一个文件夹。

```csharp
Task<IStorageFolder?> TryGetFolderFromPathAsync(this IStorageProvider provider, string folderPath);
```
如果文件夹存在，则该方法返回文件夹；如果不存在，则返回 null。 
该方法接收一个不带协议的本地文件夹路径字符串作为参数。
它只在支持物理文件路径的操作系统上受支持，主要是桌面平台。

## 平台兼容性

| 功能 | Managed | Windows | macOS | Linux | Browser | Android | iOS |
|---------------|-------|-------|-------|-------|-------|-------|-------|
| `OpenFileBookmarkAsync` | ✓* | ✓* | ✓* | ✓* | ✓ | ✓ | ✓ |
| `OpenFolderBookmarkAsync` | ✓* | ✓* | ✓* | ✓* | ✓ | ✓ | ✓ |
| `OpenFilePickerAsync` | ✓** | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| `SaveFilePickerAsync` | ✓** | ✓ | ✓ | ✓ | ✓*** | ✓ | ✓ |
| `SaveFilePickerWithResultAsync` | ✓** | ✓ | ✓ | ✓ | ✓*** | ✓ | ✓ |
| `OpenFolderPickerAsync` | ✓** | ✓ | ✓ | ✓ | ✓*** | ✓ | ✓ |
| `TryGetFileFromPathAsync` | ✓ | ✓ | ✓ | ✓ | ✗ | ✗ | ✗ |
| `TryGetFolderFromPathAsync` | ✓ | ✓ | ✓ | ✓ | ✗ | ✗ | ✗ |
| `TryGetWellKnownFolderAsync` | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |

\* 书签在桌面平台上尚未得到完整支持，因此通常返回的是文件路径本身。后续计划在 macOS 上改进，以支持沙箱化的 Apple Store 应用。

** Managed 文件选择器仅在允许打开自定义窗口的桌面平台上可用。

*** 只有基于 Chromium 的浏览器对文件选择器提供了较完整的支持。

## 另请参阅

- [文件对话框](/docs/services/file-dialogs)：常见文件对话框用法示例。
- [文件选择器选项](/docs/services/storage/file-picker-options)：配置文件类型筛选器和对话框选项。
- [书签](/docs/services/storage/bookmarks)：持久化对已选文件和文件夹的访问权限。
- [存储项](/docs/services/storage/storage-item)：处理由存储提供程序返回的文件和文件夹。
