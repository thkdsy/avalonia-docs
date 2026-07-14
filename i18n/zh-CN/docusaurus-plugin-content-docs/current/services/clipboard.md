---
id: clipboard
title: 剪贴板
---

## 数据格式

在访问剪贴板之前，先理解由 `DataFormat<T>` 类表示的数据格式非常重要。它表示某个数据项的格式（例如文本、HTML、PNG 等），并被多种剪贴板和拖放 API 共同使用。

一个数据格式由类型（`Universal`、`Platform` 或 `Application`）、标识符以及数据类型组成。

如果两个数据格式具有相同的类型和标识符，则它们会被视为相等。

### 通用格式

通用格式是 Avalonia 可直接理解的跨平台格式。  
当前共有三种通用格式：

| 格式 | 标识符 | 类型 | 说明 |
| --------------------|------------|----------------|---------------------|
| `DataFormat.Text`   | "Text"     | `string`       | 纯文本数据 |
| `DataFormat.File`   | "File"     | [`IStorageItem`](/api/avalonia/platform/storage/istorageitem) | 文件或目录 |
| `DataFormat.Bitmap` | "Bitmap"   | [`Bitmap`](/api/avalonia/media/imaging/bitmap)       | 位图图像 |

### 平台格式

平台格式**仅与应用当前运行的平台兼容**（例如 Windows、Linux、iOS 等）。它的标识符应当是底层平台能够识别的名称。只有在你需要与目标平台直接互操作，并且清楚了解其编码或序列化方式时，才应使用这种格式。

:::caution
**不要**假设同一个标识符能在所有平台上通用！  
例如，HTML 格式在 Windows 上名为 `HTML format`，在 Linux 和 Android 上名为 `text/html`，而在 macOS 和 iOS 上则名为 `public.html`。在使用平台特定格式之前，务必确认目标操作系统。
:::

平台格式可以分别通过 `DataFormat.CreateBytesPlatformFormat` 或 `DataFormat.CreateStringPlatformFormat` 定义，它们分别使用 `byte[]` 和 `string` 作为值类型。Avalonia 不会自动执行序列化。

示例：
```csharp
if (OperatingSystem.IsMacOS())
{
    var macOSHtmlFormat = DataFormat.CreateStringPlatformFormat("public.html");
}
```

### 应用格式

应用格式是应用自身特有的格式，并可在所有允许自定义数据格式的平台上使用。它的标识符仅允许包含 ASCII 字母、数字，以及点号和短横线（`A`-`Z`、`a`-`z`、`0`-`9`、`.`、`-`）。该标识符不会直接暴露给底层平台；系统会在内部为它加上前缀，以避免与平台格式发生冲突。

当你需要在剪贴板上放置应用专属数据时，应使用这种格式，例如用于在程序的多个实例之间共享数据。

应用格式可以分别通过 `DataFormat.CreateBytesApplicationFormat` 或 `DataFormat.CreateStringApplicationFormat` 定义，它们分别使用 `byte[]` 和 `string` 作为值类型。Avalonia 不会自动执行序列化。

```csharp
var myFormat = DataFormat.CreateBytesApplicationFormat("mycompany-myapp-myformat");
```

## IClipboard

`IClipboard` 接口用于与系统剪贴板交互，可设置和读取文本、图像以及自定义数据格式。

可以通过 [`TopLevel`](/docs/fundamentals/top-level) 对象访问 `IClipboard` 实例：
```csharp
var clipboard = window.Clipboard;
```

### 读取

#### TryGetDataAsync()

剪贴板内容通过获取 [`IAsyncDataTransfer`](/api/avalonia/input/iasyncdatatransfer) 的实例来读取。这个对象负责按需提供各种格式的数据项（参见下文的 [`IAsyncDataTransfer`](#iasyncdatatransfer--iasyncdatatransferitem) 章节）。

`TryGetDataAsync` 方法会异步获取一个表示剪贴板内容的 `IAsyncDataTransfer` 对象。如果剪贴板为空，则返回 `null`。

```csharp
using var data = await clipboard.TryGetDataAsync();
```

:::caution
由于剪贴板内容可能随时发生变化，建议尽快使用返回的 `IAsyncDataTransfer` 实例，不要将它保存起来以备后用。完成所有必要操作后，调用方必须负责释放该对象，因此推荐使用 `using` 语句。
:::

#### 扩展方法

为了按特定格式读取数据，还提供了多个扩展方法。对于常见格式，你可以直接使用下列方法，而不必调用 `TryGetDataAsync()`：

- `TryGetValueAsync(DataFormat<T>)`：返回剪贴板中与指定数据格式匹配的单个 `T` 类型值；若不存在则返回 `null`。
- `TryGetValuesAsync(DataFormat<T>)`：返回剪贴板中与指定数据格式匹配的多个 `T` 类型值；若不存在则返回空数组。
- `TryGetTextAsync()`：返回与 `DataFormat.Text` 匹配的单个 `string` 值；若不存在则返回 `null`。
- `TryGetFileAsync()`：返回与 `DataFormat.File` 匹配的单个 `IStorageItem` 对象；若不存在则返回 `null`。
- `TryGetFilesAsync()`：返回与 `DataFormat.File` 匹配的多个 `IStorageItem` 对象；若不存在则返回空数组。
- `TryGetBitmapAsync()`：返回与 `DataFormat.Bitmap` 匹配的单个位图图像；若不存在则返回 `null`。

如果调用返回单个值的方法时，剪贴板中存在多个匹配值，则会使用第一个。

示例：

```csharp
var text = await clipboard.TryGetTextAsync();
Console.WriteLine($"剪贴板文本：{text}");

var file = await clipboard.TryGetFileAsync();
Console.WriteLine($"剪贴板文件：{file?.Path}");

var bitmap = await clipboard.TryGetBitmapAsync();
Console.WriteLine($"剪贴板图像：{bitmap?.PixelSize}");
```

:::tip
这些扩展方法在底层都会调用 `TryGetDataAsync()`。如果你打算连续使用多个此类方法，建议先调用一次 [`TryGetDataAsync()`](#trygetdataasync) 获取 `IAsyncDataTransfer`，然后再从该对象中读取所需值。
:::

#### TryGetInProcessDataAsync()

如果此前通过 `SetDataAsync()` 放入剪贴板的 `IAsyncDataTransfer` 实例仍然存在，那么这个方法会返回它本身。如果剪贴板内容已经变化、已经被 flush，或者平台不支持惰性提供的值，那么该方法会返回 `null`。

调用这个方法可以避免访问底层平台的剪贴板；在需要更细粒度控制时，这会很有帮助。但在大多数场景下，仍然更推荐使用 `TryGetDataAsync()`。

该方法在 Windows、macOS 和 X11 上受支持。

### 写入

#### SetDataAsync()

若要将数据放入剪贴板，请调用 `IClipboard.SetDataAsync(IAsyncDataTransfer)` 方法。它接收一个 `IAsyncDataTransfer` 的实现，该对象负责按需提供各种格式的数据项（参见下文的 [`IAsyncDataTransfer`](#iasyncdatatransfer--iasyncdatatransferitem) 章节）。

示例：
```csharp
var data = new DataTransfer();
data.Add(DataTransferItem.CreateText("已从 Avalonia 复制！"));
await clipboard.SetDataAsync(data);
```

:::note
向剪贴板放入一个新对象时，总会清除之前的所有数据。
:::

:::caution
**不要**对传给 `SetDataAsync()` 的 `IAsyncDataTransfer` 对象调用 `Dispose()`！只要它仍在剪贴板中，该实例就必须保持有效。Avalonia 会接管其生命周期，并在不再使用时自动释放。
:::

#### 扩展方法

为了更方便地写入单个特定格式，还提供了多个扩展方法。对于常见格式，你可以使用以下方法：

- `SetValueAsync(DataFormat<T>, T)`：以指定数据格式将一个 `T` 类型值写入剪贴板。
- `SetValuesAsync(DataFormat<T>, IEnumerable<T>)`：以指定数据格式将多个 `T` 类型值写入剪贴板。
- `SetTextAsync(string)`：以 `DataFormat.Text` 格式写入单个 `string` 值。
- `SetFileAsync(IStorageItem)`：以 `DataFormat.File` 格式写入单个 `IStorageItem` 对象。
- `SetFilesAsync(IEnumerable<IStorageItem>)`：以 `DataFormat.File` 格式写入多个 `IStorageItem` 对象。
- `SetBitmapAsync(Bitmap)`：以 `DataFormat.Bitmap` 格式写入单个位图图像。

#### ClearAsync()

调用 `ClearAsync()` 方法会清除剪贴板中的所有内容。

#### FlushAsync()

在 Windows、macOS 和 X11 上，数据会从放入剪贴板的 `IAsyncDataTransfer` 中按需惰性读取。
如果 Avalonia 应用在未执行 flush 的情况下退出，这些数据就会变得不可用。

调用 `FlushAsync()` 会强制系统读取所有数据并将其持久化，从而保证应用关闭后这些数据仍然可用。该功能在 Windows 和 X11（Linux）上受支持；在不支持 flush 的平台上，此方法不会执行任何操作。

## IAsyncDataTransfer 与 IAsyncDataTransferItem

`IAsyncDataTransfer` 接口表示剪贴板内容，并公开以下属性：
- `Formats`：返回一个 `DataFormat` 实例列表，表示对象内部包含的所有格式。
- `Items`：返回一个 [`IAsyncDataTransferItem`](/api/avalonia/input/iasyncdatatransferitem) 实例列表，表示对象内部包含的所有项目。

`IAsyncDataTransferItem` 接口表示 `IAsyncDataTransfer` 中的单个项目，并公开以下成员：
- `Formats`：返回一个 `DataFormat` 实例列表，表示该对象中存在的所有格式。
- `TryGetRawAsync(DataFormat)`：用于异步获取指定数据格式下的单个值。

:::info
单个项目可以同时具有多种格式。  
例如，一段富文本可以同时以 RTF、HTML 和纯文本形式表示。
:::

### 获取值

#### 原始值

要从 `IAsyncDataTransferItem` 对象中读取值，请调用 `TryGetRawAsync(DataFormat)` 方法，并指定所需的数据格式。

:::tip
返回值是无类型的（`object`）。  
如果你希望获得强类型结果，请考虑使用下文介绍的扩展方法。
:::

#### 强类型值

系统提供了多个扩展方法，用于从 `IAsyncDataTransfer` 和 `IAsyncDataTransferItem` 中读取强类型值：
- `TryGetValueAsync(DataFormat<T>)`：返回与指定数据格式匹配的单个 `T` 类型值；若不存在则返回 `null`。
- `TryGetTextAsync()`：返回与 `DataFormat.Text` 匹配的单个 `string` 值；若不存在则返回 `null`。
- `TryGetFileAsync()`：返回与 `DataFormat.File` 匹配的单个 `IStorageItem` 对象；若不存在则返回 `null`。
- `TryGetBitmapAsync()`：返回与 `DataFormat.Bitmap` 匹配的单个位图图像；若不存在则返回 `null`。

如果这些方法是对 `IAsyncDataTransfer` 调用的，并且有多个项目匹配所请求的格式，那么会使用第一个。

此外，下列扩展方法仅适用于 `IAsyncDataTransfer`：
- `TryGetValuesAsync(DataFormat<T>)`：返回与指定数据格式匹配的多个 `T` 类型值；若不存在则返回空数组。
- `TryGetFilesAsync()`：返回与 `DataFormat.File` 匹配的多个 `IStorageItem` 对象；若不存在则返回空数组。

### 实现

#### DataTransfer 与 DataTransferItem

若要提供待写入剪贴板的值，就必须实现 `IAsyncDataTransfer` 和 `IAsyncDataTransferItem`。

Avalonia 分别通过 `DataTransfer` 和 [`DataTransferItem`](/api/avalonia/input/datatransferitem) 类型提供了这两个接口的实现：
- `DataTransfer` 类本质上是一个项目列表，它提供 `Add(DataTransferItem)` 方法用于添加新项目。
- `DataTransferItem` 类可以视作一个“格式-值”对的字典，它提供 `Set<T>(DataFormat, T)` 方法用于为指定格式设置值。

示例：

```csharp
// 创建一个同时包含文本和 HTML 格式的项目。
var item = new DataTransferItem();
item.Set(DataFormat.Text, "来自 Avalonia！");
item.Set(DataFormat.CreateStringPlatformFormat("text/html"), "来自 <b>Avalonia</b>！");

// 将该项目添加到 DataTransfer 中。
var data = new DataTransfer();
data.Add(item);
```

`Set<T>` 方法还有一个重载 `Set<T>(DataFormat, Func<T>)`，允许以惰性方式提供值。

为方便使用，`DataTransferItem` 提供了若干用于设置常见格式值的方法：
- `SetText(string)`：为 `DataFormat.Text` 格式设置一个 `string` 值。
- `SetFile(IStorageItem)`：为 `DataFormat.File` 格式设置一个 `IStorageItem` 对象。
- `SetBitmap(Bitmap)`：为 `DataFormat.Bitmap` 格式设置一个 `Bitmap` 图像。

此外，还提供了若干静态工厂方法，用于创建只包含单一格式的项目：
- `DataTransferItem.Create<T>(DataFormat<T>, T)`：针对任意给定格式。
- `DataTransferItem.CreateText(string)`：针对 `DataFormat.Text` 格式。
- `DataTransferItem.CreateFile(IStorageItem)`：针对 `DataFormat.File` 格式。
- `DataTransferItem.CreateBitmap(Bitmap)`：针对 `DataFormat.Bitmap` 格式。

#### 自定义实现

在高级场景中，你可以手动实现 `IAsyncDataTransfer` 和 `IAsyncDataTransferItem`。一种常见用途是：为某个给定项目动态提供多种不同格式。

在编写实现时，请务必确认：
- [`DataTransfer` 和 `DataTransferItem`](#datatransfer--datatransferitem) 已不足以满足你的需求。
- `IAsyncDataTransfer.Formats` 包含了所有项目的格式，并且没有重复项。
- 你清楚 `IAsyncDataTransferItem.TryGetRawAsync()` 可能会在任意线程上被调用，并且不会使用 dispatcher。
- 你已经评估过是否需要同时实现 `IDataTransfer` 和 `IDataTransferItem`，因为某些平台只能以同步方式访问剪贴板。

:::caution
`IAsyncDataTransferItem.TryGetRawAsync()` 是否在 UI 线程上调用，取决于底层平台。**不要**在其中执行任何 UI 线程调用，包括 `Dispatcher.Invoke/InvokeAsync`，否则会导致死锁！
:::

## 另请参阅

- [拖放](/docs/input-interaction/drag-and-drop)：使用类似数据格式 API 的拖放数据传输。
- [TopLevel](/docs/fundamentals/top-level)：从控件访问平台服务。
