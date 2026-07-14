---
id: media-playback
title: 实现媒体播放
sidebar_label: 实现媒体播放
tags:
  - avalonia pro
  - avalonia enterprise
---

这是在 Avalonia 应用程序中使用 Avalonia Pro [`MediaPlayer`](/controls/media/mediaplayer) 实现媒体播放的实用指南。

:::info
此控件作为 [Avalonia Pro](https://avaloniaui.net/pricing) 或更高版本的一部分提供。
:::

## 初始化时机

:::caution
在 Avalonia UI 完全加载之前，`MediaPlayer` 尚未准备好接受媒体源。过早设置 `Source` 属性（例如，在 Window 或 UserControl 构造函数中）会静默失败，因为底层平台后端尚未初始化。
:::

始终在控件的 `Loaded` 事件触发后设置 `Source` 属性。您可以通过两种方式之一实现：

### 选项 1：使用 OnLoaded 重写

```csharp
public partial class MainView : UserControl
{
    private Model _vm;

    public MainView()
    {
        InitializeComponent();
        _vm = new Model();
        DataContext = _vm;
    }

    protected override void OnLoaded(RoutedEventArgs e)
    {
        base.OnLoaded(e);
        _vm.SetSource(new UriSource("file:///C:/Videos/sample.mp4"));
    }
}
```

### 选项 2：使用 Dispatcher.UIThread.Post

如果您需要从中无法重写 `OnLoaded` 的上下文设置源，请使用 `Dispatcher.UIThread.Post` 将调用延迟到 UI 线程准备就绪：

```csharp
protected override void OnLoaded(RoutedEventArgs e)
{
    base.OnLoaded(e);
    Dispatcher.UIThread.Post(() =>
    {
        mediaPlayer.Source = new UriSource("file:///C:/Videos/sample.mp4");
    });
}
```

:::tip
当使用 XAML 绑定的 `MediaPlayerControl`（例如 `Source="{Binding MediaSource}"`）时，绑定系统会自动处理时机，因为绑定是在控件附加到可视化树后评估的。仅在通过代码后置设置 `Source` 时需要显式管理时机。
:::

## 加载媒体源

### 从文件或 URL 使用 UriSource

```csharp
// 本地文件
mediaPlayer.Source = new UriSource("file:///C:/videos/sample.mp4");
// 或
mediaPlayer.Source = new UriSource(new Uri("file:///C:/videos/sample.mp4"));

// 远程 URL
mediaPlayer.Source = new UriSource("https://example.com/video.mp4");
```

**注意**：如果可能，请始终在本地文件 URI 中添加 `file://` 方案。这确保播放器识别文件的路径为本地路径。

### 从流使用 StreamSource

```csharp
// 从文件流
var fileStream = File.OpenRead("path/to/video.mp4");
mediaPlayer.Source = new StreamSource(fileStream);

// 从内存流
var memoryStream = new MemoryStream(byteArray);
mediaPlayer.Source = new StreamSource(memoryStream);
```

**注意**：确保不要控制传递给 `StreamSource` 的流的释放，因为播放器会管理其生命周期。

### 使用文件选择器和 StorageFileSource

```csharp
public async void OpenFile_Click(object sender, RoutedEventArgs e)
{
    var storageProvider = TopLevel.GetTopLevel(this)?.StorageProvider;
    if (storageProvider == null) return;

    var files = await storageProvider.OpenFilePickerAsync(new FilePickerOpenOptions
    {
        AllowMultiple = false
    });

    if (files.Count != 1) return;
    if (files[0].Path is not { } path) return;

    mediaPlayer.Source = new StorageFileSource(files[0]);
}
```

## 常见操作

### 播放控制

```csharp
// 播放/暂停
await mediaPlayer.PlayAsync();
await mediaPlayer.PauseAsync();

// 停止
await mediaPlayer.StopAsync();

// 搜索位置
mediaPlayer.Position = TimeSpan.FromSeconds(30);

// 更改音量（0.0 到 1.0）
mediaPlayer.Volume = 0.75;

// 静音/取消静音
mediaPlayer.IsMuted = true;
```

### 媒体信息

```csharp
// 获取时长
TimeSpan? duration = mediaPlayer.Duration;

// 检查媒体是否有视频
bool hasVideo = mediaPlayer.HasVideo;

// 检查媒体是否可搜索
bool isSeekable = mediaPlayer.IsSeekable;

// 获取当前位置
TimeSpan position = mediaPlayer.Position;
```

### 错误处理

```csharp
mediaPlayer.ErrorOccurred += (sender, args) =>
{
    Console.WriteLine($"媒体错误：{args.Message}");
    args.Handled = true; // 阻止抛出异常。
};
```

**注意**：此回调让您有机会优雅地重置 `MediaPlayer` 的状态。

### 基本示例

```xml
<Window xmlns="https://github.com/avaloniaui"
        Width="800" Height="450">

    <Grid RowDefinitions="*, Auto">
        <MediaPlayerControl Name="mediaPlayer" Grid.Row="0" />
        <Button Grid.Row="1" Content="打开文件" Click="OpenFile_Click" />
    </Grid>

</Window>
```

```csharp
public partial class MainWindow : Window
{
    public MainWindow()
    {
        InitializeComponent();
    }

    public async void OpenFile_Click(object sender, RoutedEventArgs e)
    {
        var storageProvider = TopLevel.GetTopLevel(this)?.StorageProvider;
        if (storageProvider == null) return;

        var files = await storageProvider.OpenFilePickerAsync(new FilePickerOpenOptions
        {
            AllowMultiple = false
        });

        if (files.Count != 1) return;
        if (files[0].Path is not { } path) return;

        mediaPlayer.Source = new StorageFileSource(files[0]);
    }
}
```

## 平台先决条件

`MediaPlayer` 组件在每个支持的平台上依赖于原生媒体播放框架：

### Windows

`MediaPlayer` 使用 Windows 的 Media Foundation 渲染多媒体内容，
同时尽可能在最终用户的安装上使用 Vulkan Graphics API。

对于 Windows 10/11：

- 无需额外设置。

对于 Windows 10N/11N 或 10KN/11KN：

- 请参阅[故障排除](/troubleshooting/controls/mediaplayer)。

### macOS/iOS

`MediaPlayer` 使用 Apple 的 AVFoundation 在 macOS 和 iOS 上渲染多媒体内容。

需要 macOS 10.15 或 iOS 12.0 或更高版本。

- 无需额外设置。

### Android

`MediaPlayer`    使用 Android 的 ExoPlayer 组件渲染多媒体内容，如果
终端设备支持，还会使用 Vulkan Graphics API。

需要 Android API 21（Android 5.0）或更高版本。

- 在应用程序构建器中调用 `UseAndroidPlayer`；
```csharp
protected override AppBuilder CustomizeAppBuilder(AppBuilder builder)
{
    return base.CustomizeAppBuilder(builder)
        ...
        .UseAndroidPlayer(this)
        ...
        .LogToTrace();
}
  ```
- 对于 Vulkan 支持；
```csharp
protected override AppBuilder CustomizeAppBuilder(AppBuilder builder)
{
    return base.CustomizeAppBuilder(builder)
        .UseAndroidPlayer(this)
        .With(new VulkanOptions()
        {
            VulkanDeviceCreationOptions = new VulkanDeviceCreationOptions()
            {
                DeviceExtensions = new[] { "VK_ANDROID_external_memory_android_hardware_buffer", "VK_EXT_queue_family_foreign" }
            }
        })
        ...
        .LogToTrace();
}
```

### Linux

`MediaPlayer` 使用系统安装的 LibVLC 库为 Linux 发行版渲染多媒体内容。

需要 LibVLC 3.0.21 或更高版本。

Debian/Ubuntu：

```bash
apt install libvlc
```

Fedora：

```bash
dnf install libvlc
```

### 嵌入式 Linux（直接渲染管理器）

与常规 Linux 上的要求类似，`MediaPlayer` 使用系统安装的 LibVLC 库为嵌入式 Linux 设备渲染多媒体内容。

请遵循[在 Linux DRM Framebuffer 上设置 Avalonia 的指南](https://avaloniaui.net/blog/unleashing-net-on-embedded-linux)。

之后，按照上述说明安装 VLC 依赖项。

Linux DRM 设置无需特殊要求，您可以像在常规 Linux 上一样继续使用 `MediaPlayer` 控件。

## 编解码器支持

`MediaPlayer` 支持的媒体编解码器取决于目标平台的内置编解码器和额外的插件。

最安全的假设是，大多数平台支持的视频格式是 `MPEG-4 Part 10 - Advanced Video Coding`，
更常被称为 `H.264`，视频容器为 `MPEG-4 Part 14` 或 `MP4`。

至于音频，安全假设支持的编解码器是 `MP3`、`AAC` 和 `WAV`。

关于平台特定支持哪些编解码器的资源，请查看以下链接：

### Windows

- https://support.microsoft.com/en-us/windows/codecs-in-media-player-d5c2cdcd-83a2-4805-abb0-c6888138e456

### Android

- https://developer.android.com/media/platform/supported-formats

### Linux

- https://www.videolan.org/vlc/features.html

### macOS 和 iOS

- 尚未找到关于 macOS/iOS 默认支持的编解码器的权威主要来源。

## 另请参阅

- [MediaPlayer 控件](/controls/media/mediaplayer)
- [MediaPlayer 类](/controls/media/mediaplayer/mediaplayer-class)
- [MediaSource 类](/controls/media/mediaplayer/mediasource)
- [安装 Avalonia Pro](/tools/installing-avalonia-pro)
- [故障排除](/troubleshooting/controls/mediaplayer)
