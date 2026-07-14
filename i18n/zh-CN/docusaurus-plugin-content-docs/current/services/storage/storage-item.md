---
id: storage-item
title: 存储项
---

## StorageFile 与 StorageFolder 的公共成员

### Name

获取项目的短名称；如果存在文件扩展名，则会包含扩展名。

### Path

获取该项目的文件系统路径。

:::note
Android 后端可能会返回带有 `content:` 协议的文件路径。
浏览器和 iOS 后端则可能返回相对 URI。
:::

:::caution
**不要**使用 `Path` 属性来保存对文件或文件夹的访问权限。关于如何持续访问存储项，请改看 [书签](/docs/services/storage/bookmarks) 页面。

**不要**使用 `Path` 属性直接按路径读取文件，因为这种方式在大多数移动端和浏览器平台上都无法工作。请改用 [OpenReadAsync](#openreadasync) 和 [OpenWriteAsync](#openwriteasync)。
:::

### CanBookmark

如果该项目可以保存为书签并在以后复用，则返回 true。

### SaveBookmarkAsync

将项目保存为书签。
返回书签标识符；如果操作系统拒绝请求，则可能返回 null。

### GetBasicPropertiesAsync

获取当前项目的基本属性。
当前可用的属性包括：
- `Size`
- `DateCreated`
- `DateModified`

### GetParentAsync

获取当前存储项的父文件夹。

### DeleteAsync

删除当前存储项及其内容。

### MoveAsync

将当前存储项及其内容移动到一个 `IStorageFolder` 中。

## StorageFile 成员

### OpenReadAsync

打开一个用于读取的流。

### OpenWriteAsync

打开一个用于写入文件的流。

## StorageFolder 成员

### GetItemsAsync

获取当前文件夹中的文件和子文件夹。
当该方法成功完成时，会返回当前文件夹中的文件和文件夹列表。列表中的每个项目都由一个 `IStorageItem` 实现对象表示。

:::note
该方法采用惰性求值，并且是异步的。
:::

### CreateFileAsync

在当前存储文件夹下创建一个指定名称的子文件。

### CreateFolderAsync

在当前存储文件夹下创建一个指定名称的子文件夹。

### TryGetSingleFileAsync

根据名称从当前存储文件夹中获取单个文件。如果找到匹配项，则返回对应的 `IStorageFile`；如果不存在指定名称的文件，则返回 null。这是一个扩展方法，用于简化“从文件夹中获取某个特定文件”的常见写法。

```csharp
IStorageFile? file = await folder.TryGetSingleFileAsync("config.json");
```

### TryGetSingleFolderAsync

根据名称从当前存储文件夹中获取单个子文件夹。如果找到匹配项，则返回对应的 `IStorageFolder`；如果不存在指定名称的文件夹，则返回 null。这是一个扩展方法，用于简化“从文件夹中获取某个特定子文件夹”的常见写法。

```csharp
IStorageFolder? subFolder = await folder.TryGetSingleFolderAsync("images");
```

## 扩展方法

### TryGetLocalPath

以字符串形式获取该项目在本地文件系统中的路径。
Android 平台通常使用 `content:` 虚拟文件路径，而浏览器平台的访问环境是隔离的、没有完整路径，因此在这些平台上该方法会返回 null。

:::note
如果你希望保存文件路径并在以后复用（例如配合 `TryGetFileFromPathAsync`），请优先考虑使用 [书签](/docs/services/storage/bookmarks)。它们专门为沙箱环境设计，而在这类环境中，应用通常并不能直接访问物理文件系统。
:::

## 另请参阅

- [存储提供程序](/docs/services/storage/storage-provider)：完整的存储提供程序 API 参考。
- [书签](/docs/services/storage/bookmarks)：持久化对已选文件和文件夹的访问权限。
- [文件对话框](/docs/services/file-dialogs)：使用文件打开、保存和文件夹选择对话框。
