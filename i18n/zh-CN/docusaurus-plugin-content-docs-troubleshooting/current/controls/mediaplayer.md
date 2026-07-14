---
id: mediaplayer
title: MediaPlayer 问题
description: 排查 Avalonia 中常见的 MediaPlayer 问题，包括播放失败、黑屏、内存泄漏以及平台特定的编解码器问题。
doc-type: troubleshooting
sidebar_label: MediaPlayer
tags:
  - avalonia pro
  - avalonia enterprise
---

## 在构造函数中设置 `Source` 时无法播放

在 Avalonia UI 完全加载之前，`MediaPlayer` 还未准备好接受媒体源。如果你在 `Window` 或 `UserControl` 的构造函数中设置 `Source`，底层平台后端尚未初始化，因此该源不会加载。

**修复：** 在 `OnLoaded` 重写中设置 `Source` 属性，或使用 `Dispatcher.UIThread.Post`：

```csharp
// Option 1: OnLoaded override
protected override void OnLoaded(RoutedEventArgs e)
{
    base.OnLoaded(e);
    mediaPlayer.Source = new UriSource("file:///C:/Videos/sample.mp4");
}

// Option 2: Dispatcher.UIThread.Post
protected override void OnLoaded(RoutedEventArgs e)
{
    base.OnLoaded(e);
    Dispatcher.UIThread.Post(() =>
    {
        mediaPlayer.Source = new UriSource("file:///C:/Videos/sample.mp4");
    });
}
```

更多详情请参阅 [初始化时机](/controls/media/media-playback#initialization-timing)。

## 黑屏且没有可见输出

如果 `MediaPlayer` 控件渲染出黑色矩形而不是视频内容，请检查以下内容：

- **验证你的源路径或 URL。** 确保传递给 `Source` 的文件路径或远程 URL 正确且可访问。
- **确认媒体格式受支持。** 并非每个平台都支持所有编解码器。请使用已知可正常播放的文件进行测试，例如使用 H.264 和 AAC 编码的 MP4。
- **检查是否已安装所需的编解码器。** 某些操作系统默认不包含特定编解码器。请参阅下面的平台特定部分以获取指导。

## Windows 上无法播放

Windows 的 “N” 和 “KN” 版本默认不包含媒体组件。如果你运行的是 Windows 10N、10KN、11N 或 11KN，请从 Microsoft 安装 [Media Feature Pack](https://support.microsoft.com/en-us/topic/media-feature-pack-list-for-windows-n-editions-c1c6fffa-d052-8338-7a79-a4bb980a700a)。

如果你使用的是标准 Windows 版本但播放仍然失败：

- 如果内置编解码器不支持你需要的格式，请安装替代编解码器包。
- 确认 Windows Media Player 或“电影和电视”应用可以播放同一文件。如果这些应用也失败，则问题出在操作系统或编解码器层面，而不是你的 Avalonia 应用中。

## Linux 上无法播放

Linux 上的 `MediaPlayer` 依赖 LibVLC。请确保你的系统已安装 VLC（或至少安装了 `libvlc-dev` 包）：

```bash
# Debian / Ubuntu
sudo apt install vlc libvlc-dev

# Fedora
sudo dnf install vlc vlc-devel
```

安装后，重启应用以加载新的库。

## 内存泄漏

不正确地清理 `MediaPlayer` 资源会导致内存随时间增长。请遵循以下做法以避免泄漏：

- 在完成使用后，始终对播放器调用 `UnInitialize()`，例如在视图的 `OnUnloaded` 重写中调用。
- 在处理 `StreamSource` 时使用 `using` 语句，以确保流被正确释放。

```csharp
protected override void OnUnloaded(RoutedEventArgs e)
{
    base.OnUnloaded(e);
    mediaPlayer.UnInitialize();
}
```

## 播放器卡在错误状态

当播放器遇到问题（例如不受支持的编解码器或网络超时）时，它会进入错误状态并停止响应新命令。要恢复：

1. 订阅 `ErrorOccurred` 事件，以便记录错误详情并做出相应处理。
2. 调用 `Player.ReleaseAsync()` 将播放器从错误状态重置。

```csharp
mediaPlayer.ErrorOccurred += (sender, args) =>
{
    // Log or display the error
    Console.WriteLine($"Playback error: {args.Message}");
};

// Reset the player after an error
await mediaPlayer.Player.ReleaseAsync();
```

释放后，你可以设置新的 `Source` 并再次尝试播放。

## 另请参阅

- [MediaPlayer 控件](/controls/media/mediaplayercontrol)
- [MediaPlayer 类](/controls/media/mediaplayer)
- [MediaSource 类](/controls/media/mediasource)