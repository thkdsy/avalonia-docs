---
id: launcher
title: 启动器
description: 了解如何使用 Avalonia 的 Launcher 服务，在用户默认应用中打开文件、文件夹和 URI。
doc-type: concept
---

`Launcher` 服务允许你在与目标项关联的默认应用中打开文件、文件夹或 URI。例如，你可以用它在用户默认浏览器中打开一个 URL，或者在已注册处理该文件类型的应用中打开某个文档。

你可以通过 `TopLevel` 或 `Window` 的实例来访问 `Launcher`。关于如何访问 `TopLevel`，请参阅 [TopLevel](/docs/fundamentals/top-level) 页面。

```csharp
var launcher = TopLevel.GetTopLevel(control).Launcher;
```

## 方法

### `LaunchUriAsync`

启动与指定 URI 协议关联的默认应用。

```csharp
Task<bool> LaunchUriAsync(Uri uri)
```

:::note
输入 URI 可以使用任意协议，包括自定义协议。不过，是否允许该启动请求最终仍由操作系统决定。
:::

**示例：在默认浏览器中打开 URL**

```csharp
var success = await launcher.LaunchUriAsync(new Uri("https://avaloniaui.net"));
```

### `LaunchFileAsync`

启动与指定存储文件或文件夹关联的默认应用。

```csharp
Task<bool> LaunchFileAsync(IStorageItem storageItem);
```

:::note
`IStorageItem` 是从 `IStorageProvider` 或 `IClipboard` 这类沙箱 API 中获取的文件或文件夹对象。如果你的目标平台仅限于非沙箱桌面平台，可以考虑改用接收 `FileInfo` 或 `DirectoryInfo` 的扩展方法。
:::

**示例：打开用户选择的文件**

```csharp
var files = await storageProvider.OpenFilePickerAsync(new FilePickerOpenOptions());
if (files.Count > 0)
{
    await launcher.LaunchFileAsync(files[0]);
}
```

## 扩展方法

当你的目标平台是非沙箱桌面平台（Windows、macOS、Linux）时，可以使用下面这些更方便的扩展方法。

### `LaunchFileInfoAsync`

启动与指定文件关联的默认应用。

```csharp
Task<bool> LaunchFileInfoAsync(FileInfo fileInfo)
```

**示例**

```csharp
var file = new FileInfo("/path/to/document.pdf");
var success = await launcher.LaunchFileInfoAsync(file);
```

### `LaunchDirectoryInfoAsync`

启动与指定目录（文件夹）关联的默认应用。通常情况下，这会在系统文件管理器中打开该文件夹。

```csharp
Task<bool> LaunchDirectoryInfoAsync(DirectoryInfo directoryInfo);
```

**示例**

```csharp
var folder = new DirectoryInfo("/path/to/folder");
var success = await launcher.LaunchDirectoryInfoAsync(folder);
```

## 返回值

这些方法都会返回一个 `bool`，表示操作系统是否能够处理该请求。返回 `true` 并不保证某个应用一定真的打开了该项目，它只表示操作系统已接受该请求且未报错。

## 平台兼容性

| 功能 | Windows | macOS | Linux | Browser | Android | iOS |
|---------------|-------|-------|-------|-------|-------|-------|
| `LaunchUriAsync` | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| `LaunchFileAsync` | ✓ | ✓ | ✓ | ✗ | ✓ | ✓ |
| `LaunchFileInfoAsync` | ✓ | ✓ | ✓ | ✗ | ✗ | ✗ |
| `LaunchDirectoryInfoAsync` | ✓ | ✓ | ✓ | ✗ | ✗ | ✗ |

## 另请参阅

- [存储提供程序](/docs/services/storage/storage-provider)：文件和文件夹管理 API。
- [剪贴板](/docs/services/clipboard)：读取和写入剪贴板数据。
- [TopLevel](/docs/fundamentals/top-level)：从控件访问平台服务。
