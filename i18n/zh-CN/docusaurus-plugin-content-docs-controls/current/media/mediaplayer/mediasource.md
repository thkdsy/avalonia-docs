---
id: mediasource
title: MediaSource 类
tags:
  - avalonia pro
  - avalonia enterprise
---

`MediaSource` 类层次结构为 Avalonia Pro MediaControls 中不同类型媒体内容源提供了抽象。这允许媒体播放系统通过统一的接口处理各种内容源（文件、URL、流）。

:::info
此控件作为 [Avalonia Pro](https://avaloniaui.net/pricing) 或更高版本的一部分提供。
:::

## MediaSource（抽象基类）

`MediaSource` 是一个抽象基类，定义了所有媒体源的公共接口。

### 方法

| 方法 | 返回类型 | 描述 |
|-----------|-------------|----------------------------------------------|
| Dispose() | void | 释放媒体源使用的资源。 |

## UriSource 类

`UriSource` 类表示由 URI 引用的媒体内容，可以指向本地文件或网络资源。

### 属性

| 属性 | 类型 | 描述 |
|----------|------|------------------------------------------------|
| Source | Uri | 获取指向媒体内容的 URI。 |

### 构造函数

| 构造函数 | 描述 |
|--------------------------|--------------------------------------------|
| UriSource(Uri source) | 使用指定的 URI 初始化。 |
| UriSource(string source) | 使用指定的 URI 字符串初始化。 |

### 方法

| 方法 | 返回类型 | 描述 |
|-------------------------|-------------|---------------------------------------------|
| Equals(UriSource other) | bool | 确定与另一个 UriSource 的相等性。 |
| Equals(object obj) | bool | 确定与对象的相等性。 |
| GetHashCode() | int | 返回此实例的哈希码。 |
| Dispose() | void | 释放资源（通常为空操作）。 |

### 使用示例

```csharp
// 从字符串 URL
var webSource = new UriSource("https://example.com/video.mp4");

// 从文件路径
var fileSource = new UriSource("file:///C:/Videos/sample.mp4");

// 从 Uri 对象
var uri = new Uri("rtsp://example.com/stream");
var streamSource = new UriSource(uri);
```

## StreamSource 类

`StreamSource` 类表示作为流提供的媒体内容，允许播放动态或内存中的内容。

### 属性

| 属性 | 类型 | 描述 |
|--------------|--------|------------------------------------------------------|
| TargetStream | Stream | 获取包含媒体数据的底层流。 |
| IsSeekable | bool | 获取底层流是否支持搜索。 |

### 构造函数

| 构造函数 | 描述 |
|-----------------------------------|----------------------------------------|
| StreamSource(Stream targetStream) | 使用指定的流初始化。 |

### 方法

| 方法 | 返回类型 | 描述 |
|----------------------------|-------------|----------------------------------------------|
| Equals(StreamSource other) | bool | 确定与另一个 StreamSource 的相等性。 |
| Dispose() | void | 释放资源，包括底层流。 |

### 使用示例

```csharp
// 从文件流
var fileStream = File.OpenRead("video.mp4");
var fileStreamSource = new StreamSource(fileStream);

// 从内存流
byte[] videoData = GetVideoData();
var memoryStream = new MemoryStream(videoData);
var memoryStreamSource = new StreamSource(memoryStream);

// 从网络流
var webRequest = WebRequest.Create("https://example.com/video.mp4");
var responseStream = webRequest.GetResponse().GetResponseStream();
var networkStreamSource = new StreamSource(responseStream);
```

## 在 `UriSource` 和 `StreamSource` 之间选择

### 何时使用 `UriSource`

- 本地媒体文件。
- 具有直接 URL 的网络流。
- 实时协议流（RTSP/RTMP/RDP）。
- 任何具有标准 URI 表示的媒体。

**优势**：

- 较低的开销。
- 媒体后端的原生处理。
- 无内存或生命周期管理问题。

### 何时使用 `StreamSource`

- 内存中的媒体内容。
- 运行时生成的动态内容。
- 从非标准源加载的内容。
- 需要在播放前预处理的内容。

**优势**：

- 自定义内容源的灵活性。
- 无需临时文件。
- 适用于加密或受保护的内容。

## 资源管理

`UriSource` 和 `StreamSource` 都实现了 `IDisposable`：

- 对于 `UriSource`，`Dispose` 方法通常为空操作。
- 对于 `StreamSource`，`Dispose` 方法会释放底层流。

`MediaPlayer` 自动管理生命周期：

- 设置新的 Source 时，先前的 Source 会被释放。
- 当播放器被释放或反初始化时，当前的 Source 会被释放。

## 最佳实践

1. **资源管理**：
   - 不要释放传递给 `StreamSource` 的流，因为它会获得所有权。

2. **源选择**：
   - 尽可能使用 `UriSource` 处理文件和网络媒体（更高效）。
   - 对内存中内容或预处理使用 `StreamSource`。

3. **错误处理**：
   - 在创建 `UriSource` 之前验证 URI。
   - 在创建 `StreamSource` 之前验证流是否可读。
   - 处理打开文件或网络资源时的异常。

4. **搜索考虑**：
   - 检查 `StreamSource.IsSeekable` 以确定是否支持搜索。
   - 如果需要搜索，确保流支持搜索（`CanSeek` = true）。

## 另请参阅

- [MediaPlayer 控件](/controls/media/mediaplayer)
- [MediaPlayer 类](/controls/media/mediaplayer/mediaplayer-class)
- [实现 MediaPlayer](/controls/media/mediaplayer/media-playback)
- [安装 Avalonia Pro](/tools/installing-avalonia-pro)
- [故障排除](/troubleshooting/controls/mediaplayer)
