---
id: bookmarks
title: 书签
---

在具有严格安全与隐私控制的现代操作系统中，书签对于持续访问文件和文件夹尤为重要。例如，在 iOS 和较新的 macOS 版本上，直接访问文件系统通常受到严格限制。应用需要先通过系统提供的文件选择器让用户选中某个文件或文件夹，然后操作系统会向应用授予一个带安全作用域的书签，供应用在未来再次访问该文件或文件夹时使用。

在 Avalonia 的 `StorageProvider` 中，这些书签分别由 `IStorageBookmarkFile` 和 `IStorageBookmarkFolder` 接口表示。

## Avalonia.Platform.Storage
### `IStorageBookmarkItem` 接口
`IStorageBookmarkItem` 接口表示一个已书签化的存储项。它继承自 `IStorageItem` 和 `IDisposable`。这个接口不能由客户端自行实现，也就是说，没有特殊权限时你不能创建自己的实现类。

它提供的关键属性和方法如下：

#### 属性：

`CanBookmark`：指示该项目是否可以被保存为书签并在稍后复用。

`Name`：项目名称。

`Path`：项目的文件系统路径。

#### 方法：
`CreateFileAsync(String)`,`CreateFolderAsync(String)`,`DeleteAsync()`,`Dispose()`,`GetBasicPropertiesAsync()`,`GetFileAsync(String)`,`GetFolderAsync(String)`,`GetItemsAsync()`,`GetParentAsync()`,`MoveAsync(IStorageFolder)`,`ReleaseBookmarkAsync()`,`SaveBookmarkAsync()`.

### `IStorageBookmarkFolder` 接口

#### 属性：
与 IStorageBookmarkItem 相同。

#### 方法：
`DeleteAsync()`,`Dispose()`,`GetBasicPropertiesAsync()`,`GetBasicPropertiesAsync()`,`GetParentAsync()`,`MoveAsync(IStorageFolder)`,`OpenReadAsync()`,`OpenWriteAsync()`,`ReleaseBookmarkAsync()`,`SaveBookmarkAsync()`.



## 如何使用书签方法
本节提供一个关于 `bookmark` 用法的实用指南。

### 保存与加载书签
若要为某个特定文件或文件夹获取书签 ID，请在存储项上调用异步方法 `SaveBookmarkAsync()`。拿到书签 ID 后，你可以把它保存到本地数据库中，以便未来直接复用，而无需每次都要求用户重新选择文件夹。

`SaveBookmarkAsync()`：用于获取某个已选文件或文件夹的 `bookmark ID`，并可将其保存起来以备后续使用。

```csharp
// 示例用法
private async Task SaveBookmarksAsync(Control control)
{
    // 必须先选择一个文件夹
    if (_lastSelectedFolder == null) return;

    var bookmarkId = await _lastSelectedFolder.SaveBookmarkAsync();

    if (bookmarkId != null)
    {
        // 将 bookmarkId 保存到本地文件中，以便后续使用。
        // ...（保存 bookmarkId 的代码）
    }
}
```

你可以使用 `OpenFolderBookmarkAsync()` 方法通过 `bookmark ID` 打开一个已书签化的文件夹。如果操作系统拒绝请求，则会返回 null。

```csharp
// 示例用法
private async Task LoadFolderByBookmarkAsync(Control control, string bookmarkId)
{
    if (string.IsNullOrEmpty(bookmarkId)) return;

    var toplevel = TopLevel.GetTopLevel(control);
    if (toplevel?.StorageProvider != null)
    {
        var folder = await toplevel.StorageProvider.OpenFolderBookmarkAsync(bookmarkId);

        if (folder != null)
        {
            // 成功打开已书签化的文件夹。
            // ...（将 folder 保存到变量中的代码）
            LastSelectedFolder = folder;
        }
    }
}
```

`ReleaseBookmarkAsync()`：用于撤销操作系统授予的安全作用域访问权限。当你不再需要访问该已书签化项目时，应调用它。

```csharp
// 示例用法
private async Task ReleaseBookmarkAsync(Control control, string bookmarkId)
{
    if (string.IsNullOrEmpty(bookmarkId)) return;

    // 首先，尝试释放操作系统层面的书签。
    var toplevel = TopLevel.GetTopLevel(control);
    if (toplevel?.StorageProvider != null)
    {
        var folder = await toplevel.StorageProvider.OpenFolderBookmarkAsync(bookmarkId);
        if (folder is IStorageBookmarkItem storageBookmark)
        {
            await storageBookmark.ReleaseBookmarkAsync();
            storageBookmark.Dispose();
        }
    }
        
    // 然后，从本地存储中移除该 ID。
    // ...（从文件中移除 bookmarkId 的代码）

}
```


### 通过书签读取和写入文件内容

`OpenFileBookmarkAsync()`：用于通过已保存的 `bookmark ID` 打开一个已书签化文件。如果操作系统拒绝请求，则返回 null。


通过 `OpenFileBookmarkAsync()` 获取到已书签化文件后，你可以使用 `OpenReadAsync()` 读取其内容，或者通过 `OpenWriteAsync()` 修改其内容。

`OpenReadAsync()`：打开一个读取该已书签化文件内容的流。

```csharp
// 示例用法
private async Task LoadFileByBookmarkAsync(Control control, string bookmarkId)
{
    if (string.IsNullOrEmpty(bookmarkId)) return;
    var toplevel = TopLevel.GetTopLevel(control);
    if (toplevel?.StorageProvider != null)
    {
        IStorageFile bookmarkedFile = await toplevel.StorageProvider.OpenFileBookmarkAsync(bookmarkId);
        if (bookmarkedFile != null)
        {
            // 读取 bookmarkedFile 的内容
            // ...（使用读取流的代码）
            await using var readStream = await bookmarkedFile.OpenReadAsync();
            using var reader = new StreamReader(readStream, Encoding.UTF8);
            FileContent = await reader.ReadToEndAsync();
        }
    }
}
```

`OpenWriteAsync()`：打开一个用于写入该已书签化文件的流。

```csharp
// 示例用法
private async Task SaveFileByBookmarkAsync(Control control, string bookmarkId)
{
    if (string.IsNullOrEmpty(bookmarkId)) return;
    var toplevel = TopLevel.GetTopLevel(control);
    if (toplevel?.StorageProvider != null)
    {
        IStorageFile bookmarkedFile = await toplevel.StorageProvider.OpenFileBookmarkAsync(bookmarkId);
        if (bookmarkedFile != null)
        {
            // 写入 bookmarkedFile 的内容
            // ...（使用写入流的代码）
            await using var writeStream = await bookmarkedFile.OpenWriteAsync();
            await using var writer = new StreamWriter(writeStream, Encoding.UTF8);
            await writer.WriteAsync(FileContent);
        }
    }
}
```


### 管理已书签化的文件和文件夹
书签加载完成后，你就可以使用从 `IStorageItem` 继承而来的方法来操作该文件或文件夹。

`DeleteAsync()`：异步删除当前存储项及其内容。

```csharp
// 示例用法
private async Task DeleteFileAsync()
{
    if (SelectedFile != null && LastSelectedFolder != null)
    {
        await SelectedFile.DeleteAsync();
        // 然后刷新 UI。
    }
}
```

`MoveAsync(IStorageFolder)`：异步将该已书签化项目移动到新位置。

```csharp
IStorageFile bookmarkedFile = ...;
IStorageFolder newDestinationFolder = ...;
await bookmarkedFile.MoveAsync(newDestinationFolder);
```

`GetBasicPropertiesAsync()`：异步获取存储项的基本属性，例如大小和修改日期。

```csharp
// 示例用法
IStorageFile bookmarkedFile = ...;
var properties = await bookmarkedFile.GetBasicPropertiesAsync();
long size = properties.Size;
```

`GetParentAsync()`：异步获取当前存储项的父文件夹。

```csharp
// 示例用法
IStorageFile bookmarkedFile = ...;
var parentFolder = await bookmarkedFile.GetParentAsync();
string parentName = parentFolder.Name;
```

`TryGetLocalPath()`：这个扩展方法尝试以字符串形式获取本地文件系统路径。在某些需要本地路径的平台特定操作中，它会很有用。

```csharp
// 这在 Windows 上可用，但在其他平台上可能返回 null
IStorageFile bookmarkedFile = ...;
string? localPath = bookmarkedFile.TryGetLocalPath();
```

## 各平台中的书签表示形式
`bookmark ID` 的表示方式会因平台而异：

**Windows**：书签通常只是一个简单的绝对路径字符串，因此它可能长这样：`C:\Documents\Avalonia\bookmarks.pdf`

**Android**：可以把 content provider 想象成一个“服务员”，应用通过 Content URI 向它请求某个文件/文件夹。URI 格式大致是 `content://[Authority]/[path]/[id]`。例如，`com.android.externalstorage.documents` 是一个用于访问外部存储提供程序的 `Authority`，因此一个书签可能长这样：`content://com.android.externalstorage.documents/tree/[your folder path]`（参考：[Create a content provider | Android Developers](https://developer.android.com/guide/topics/providers/content-provider-creating)）。

:::note
具体行为和能力取决于目标操作系统及其安全策略。例如，在某些平台上，如果用户移动或重命名了书签指向的文件或文件夹，该书签可能会失效。
:::

:::note
不建议将书签 ID 存储到远程数据库中，因为书签未必具有长期稳定性，而且其中可能包含敏感的文件路径信息。
:::

## 另请参阅

- [存储提供程序](/docs/services/storage/storage-provider)：完整的存储提供程序 API 参考。
- [存储项](/docs/services/storage/storage-item)：处理文件和文件夹。
- [文件对话框](/docs/services/file-dialogs)：使用文件打开、保存和文件夹选择对话框。
