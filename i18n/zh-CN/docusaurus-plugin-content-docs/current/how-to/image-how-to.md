---
id: image-how-to
title: "如何：显示和处理图像"
description: 在 Avalonia 中从多种来源加载图像、处理宽高比，并操作位图。
doc-type: how-to
---

本指南介绍常见的 Image 控件使用模式：从不同来源加载图像、处理宽高比、显示动态图像以及操作位图。

## 从资源中加载图像

将图像作为嵌入资源包含到项目中，并通过 `avares://` URI 方案来引用：

```xml
<Image Source="avares://MyApp/Assets/logo.png" Width="200" />
```

请确保在 `.csproj` 中把该文件设置为 `AvaloniaResource`：

```xml
<ItemGroup>
    <AvaloniaResource Include="Assets\**" />
</ItemGroup>
```

## 从文件路径加载图像

将磁盘中加载得到的 `Bitmap` 绑定到界面：

```csharp
[ObservableProperty]
private Bitmap? _photo;

[RelayCommand]
private async Task LoadPhoto()
{
    var topLevel = TopLevel.GetTopLevel(App.Current?.ApplicationLifetime
        is IClassicDesktopStyleApplicationLifetime desktop ? desktop.MainWindow : null);
    if (topLevel is null) return;

    var files = await topLevel.StorageProvider.OpenFilePickerAsync(
        new FilePickerOpenOptions
        {
            Title = "Select an image",
            FileTypeFilter = new[] { FilePickerFileTypes.ImageAll }
        });

    if (files.Count > 0)
    {
        await using var stream = await files[0].OpenReadAsync();
        Photo = new Bitmap(stream);
    }
}
```

```xml
<Image Source="{Binding Photo}" MaxWidth="400" />
```

## 从 URL 加载图像

你可以使用 `AsyncImageLoader`，或者直接在视图模型中加载位图：

```csharp
[RelayCommand]
private async Task LoadFromUrl(string url)
{
    using var client = new HttpClient();
    var bytes = await client.GetByteArrayAsync(url);
    using var stream = new MemoryStream(bytes);
    Photo = new Bitmap(stream);
}
```

## 拉伸模式

[`Stretch`](/api/avalonia/media/stretch) 属性用于控制图像如何填充其可用边界：

```xml
<!-- 保持宽高比，并完整适配边界 -->
<Image Source="{Binding Photo}" Stretch="Uniform" />

<!-- 填满整个区域，可能会裁剪 -->
<Image Source="{Binding Photo}" Stretch="UniformToFill" />

<!-- 拉伸填满，忽略宽高比 -->
<Image Source="{Binding Photo}" Stretch="Fill" />

<!-- 不缩放，以原始尺寸显示 -->
<Image Source="{Binding Photo}" Stretch="None" />
```

| Stretch | 说明 |
|---|---|
| `Uniform` | 按比例缩放以适配边界（默认）。 |
| `UniformToFill` | 按比例缩放并填满区域，必要时进行裁剪。 |
| `Fill` | 完全拉伸填满区域，不保持宽高比。 |
| `None` | 按原始像素尺寸显示。 |

## 圆形图像（头像）

可以借助 `Clip` 将图像裁剪为圆形：

```xml
<Border CornerRadius="50" ClipToBounds="True"
        Width="100" Height="100">
    <Image Source="{Binding Avatar}" Stretch="UniformToFill" />
</Border>
```

## 带回退显示的图像

当没有图像可用时，显示一个占位内容：

```xml
<Panel Width="200" Height="200">
    <!-- 当 Source 为 null 时显示回退内容 -->
    <Border Background="#F3F4F6" IsVisible="{Binding Photo, Converter={x:Static ObjectConverters.IsNull}}">
        <TextBlock Text="No Image" HorizontalAlignment="Center" VerticalAlignment="Center"
                   Foreground="Gray" />
    </Border>
    <Image Source="{Binding Photo}" Stretch="Uniform" />
</Panel>
```

## 图像插值

控制图像缩放时的渲染质量：

```xml
<!-- 适合像素风图像的清晰像素显示 -->
<Image Source="{Binding PixelArt}"
       RenderOptions.BitmapInterpolationMode="None" />

<!-- 适合照片的平滑缩放 -->
<Image Source="{Binding Photo}"
       RenderOptions.BitmapInterpolationMode="HighQuality" />
```

| 模式 | 说明 |
|---|---|
| `None` | 最近邻插值，像素边缘清晰。 |
| `LowQuality` | 双线性过滤。 |
| `MediumQuality` | 做了一些优化的双线性过滤。 |
| `HighQuality` | 双三次或更高质量的重采样。 |
| `Default` | 使用平台默认值。 |

## DrawingImage（矢量图形）

使用 `DrawingImage` 可以显示与分辨率无关的矢量图像：

```xml
<Image Width="48" Height="48">
    <Image.Source>
        <DrawingImage>
            <GeometryDrawing Brush="Red"
                             Geometry="M12,21.35L10.55,20.03C5.4,15.36 2,12.27 2,8.5
                                       C2,5.41 4.42,3 7.5,3C9.24,3 10.91,3.81 12,5.08
                                       C13.09,3.81 14.76,3 16.5,3C19.58,3 22,5.41 22,8.5
                                       C22,12.27 18.6,15.36 13.45,20.03L12,21.35Z" />
        </DrawingImage>
    </Image.Source>
</Image>
```

## PathIcon

对于简单的单色图标，建议使用 `PathIcon` 而不是 `Image`：

```xml
<PathIcon Data="{StaticResource home_regular}" Width="24" Height="24"
          Foreground="{DynamicResource SystemAccentColor}" />
```

`PathIcon` 会从父级样式继承 `Foreground`，因此非常容易参与主题统一。

## RenderTargetBitmap（截图）

将某个控件渲染到位图中：

```csharp
var renderTarget = new RenderTargetBitmap(new PixelSize(800, 600));
renderTarget.Render(myControl);
renderTarget.Save("screenshot.png");
```

目标控件必须附加在一个可见窗口上。如果你想在不显示窗口的情况下进行渲染，请使用启用了 Skia 渲染器的 [headless platform](/docs/testing/setting-up-the-headless-platform#visual-regression-testing)。

## 关键属性

| 属性 | 类型 | 说明 |
|---|---|---|
| `Source` | `IImage` | 要显示的图像（例如 `Bitmap`、`DrawingImage` 等）。 |
| `Stretch` | `Stretch` | 图像填充边界的方式。 |
| `StretchDirection` | `StretchDirection` | 可选 `Both`、`UpOnly`、`DownOnly`。 |

## 另请参阅

- [Image Control Reference](/controls/media/image): Property tables.
- [PathIcon Control Reference](/controls/media/pathicon): Vector icon control.
- [DrawingImage Control Reference](/controls/media/drawingimage): Vector image source.
- [How to Bind Image Files](/docs/data-binding/how-to-bind-image-files): Binding images in data templates.
- [Image Interpolation](/docs/graphics-animation/image-interpolation): Bitmap rendering quality.
- [Assets](/docs/fundamentals/including-assets): Asset loading and URI schemes.
