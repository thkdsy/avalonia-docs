---
id: imageloader
title: 图像加载器
description: 通过实现自定义 MarkdownImageLoader 并通过 MarkdownImage 上的样式进行分配，自定义 Markdown 控件加载和解析图像的方式。
doc-type: reference
tags:
  - avalonia pro
  - avalonia enterprise
---

`Markdown` 控件通过 `MarkdownImage` 上的 `ImageLoader` 属性支持自定义图像加载。由于 Markdown 控件构建在共享文档模型之上，每个图像都是一个 `MarkdownImage` 元素（一个 `StyledElement`），您可以使用标准的 Avalonia 样式选择器来分配加载器。

:::info
此控件作为 [Avalonia Pro](https://avaloniaui.net/pricing) 或更高版本的一部分提供。
:::

## 默认行为

当您未设置自定义 `ImageLoader` 时，图像不会自动加载。要启用图像加载，请创建一个 `MarkdownImageLoader` 子类，并通过针对 `MarkdownImage` 的样式进行分配。默认的 `MarkdownImageLoader` 基类支持 `http://`、`https://` 和 `file://` 方案，成功时返回 `Bitmap`，失败时返回 `null`。

## 示例：加载 SVG 图像

### 所需包

要使用下面的自定义图像加载器示例，您需要安装以下 NuGet 包：

```bash
 dotnet add package Avalonia.Svg.Skia
```

### 实现

下面是一个支持 SVG 图像的自定义图像加载器示例：

```csharp
using Avalonia.Controls;
using Avalonia.Media.Imaging;
using Avalonia.Svg.Skia;
using System;
using System.IO;
using System.Net.Http;
using System.Text;
using System.Threading.Tasks;

public class CustomImageLoader : MarkdownImageLoader
{
    public override async Task<IImage?> LoadImageAsync(string url)
    {
        IImage? image = null;

        if (Uri.TryCreate(url, UriKind.Absolute, out var uri))
        {
            Stream? stream = null;

            if (uri.Scheme == "http" || uri.Scheme == "https")
            {
                stream = await DownloadImage(uri);
            }
            else if (uri.Scheme == "file" && File.Exists(uri.LocalPath))
            {
                stream = File.OpenRead(uri.LocalPath);
            }

            if (stream is null)
            {
                return null;
            }

            using (stream)
            {
                if (IsSvgFile(stream))
                {
                    var svg = new SvgImage
                    {
                        Source = SvgSource.LoadFromStream(stream)
                    };

                    image = svg;
                }
                else
                {
                    image = new Bitmap(stream);
                }
            }
        }

        return image;
    }

    private static async Task<Stream> DownloadImage(Uri url)
    {
        using var client = new HttpClient();
        using var response = await client.GetAsync(url).ConfigureAwait(false);
        using var stream = await response.Content.ReadAsStreamAsync().ConfigureAwait(false);
        var memoryStream = new MemoryStream();
        await stream.CopyToAsync(memoryStream).ConfigureAwait(false);
        memoryStream.Position = 0;
        return memoryStream;
    }

    private static bool IsSvgFile(Stream stream)
    {
        if (stream == null || stream.Length == 0)
            return false;
        try
        {
            const int bufferSize = 512;
            byte[] buffer = new byte[Math.Min(bufferSize, stream.Length)];
            int bytesRead = stream.Read(buffer, 0, buffer.Length);
            string header = Encoding.UTF8.GetString(buffer, 0, bytesRead);
            return header.Contains("<svg", StringComparison.OrdinalIgnoreCase);
        }
        catch
        {
            return false;
        }
        finally
        {
            stream.Position = 0;
        }
    }
}
```

## 用法

通过样式将自定义加载器分配给 `MarkdownImage` 元素。这是推荐的方法，因为图像元素是由文档模型动态创建的：

### XAML

```xml
<Window xmlns="https://github.com/avaloniaui"
        xmlns:local="using:MarkdownSample">
  <Window.Resources>
    <local:CustomImageLoader x:Key="CustomImageLoader" />
  </Window.Resources>

  <Window.Styles>
    <Style Selector="MarkdownImage">
      <Setter Property="ImageLoader" Value="{StaticResource CustomImageLoader}" />
    </Style>
  </Window.Styles>

  <Markdown Text="![SVG 图像](https://example.com/image.svg)" />
</Window>
```

### 代码后置

```csharp
var loader = new CustomImageLoader();

var style = new Style(x => x.OfType<MarkdownImage>());
style.Setters.Add(new Setter(MarkdownImage.ImageLoaderProperty, loader));
myMarkdownControl.Styles.Add(style);
```

图像加载会延迟到 `ImageSource`（从 Markdown 源自动设置）和 `ImageLoader`（通过样式设置）都可用时才会进行。这使文档模型与图像解析解耦。

## 何时使用

当默认的图像解析不能满足您的需求时，您应该实现自定义的 `MarkdownImageLoader`。例如，您可能需要渲染 SVG 图像、从需要身份验证的远程服务器加载图像，或应用缓存策略以避免重复下载。自定义加载器让您完全控制图像 URI 的解析方式以及 `Markdown` 控件可以显示的图像类型。

## 另请参阅
- [Markdown 控件](/controls/data-display/text-display/markdown)
