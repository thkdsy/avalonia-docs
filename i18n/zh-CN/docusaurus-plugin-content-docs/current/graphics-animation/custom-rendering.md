---
id: custom-rendering
title: 自定义渲染
description: 通过重写 Render 方法并使用 DrawingContext 来绘制自定义图形。
doc-type: how-to
---

Avalonia 提供了 `DrawingContext` API，用于在控件内部渲染自定义图形。当内置的形状和几何控件无法满足你的需求时，这会非常有用。

## 重写 Render

如果要绘制自定义内容，可以在任意 `Control` 上重写 `Render` 方法：

```csharp
public class SimpleCircle : Control
{
    public override void Render(DrawingContext context)
    {
        var center = new Point(Bounds.Width / 2, Bounds.Height / 2);
        var radius = Math.Min(Bounds.Width, Bounds.Height) / 2 - 4;

        context.DrawEllipse(
            Brushes.CornflowerBlue,         
            new Pen(Brushes.Navy, 2),       
            center,
            radius,                          
            radius);                         
    }
}
```

在 XAML 中这样使用该控件：

```xml
<local:SimpleCircle Width="100" Height="100" />
```

每当控件需要重绘时，都会调用 `Render` 方法。当数据发生变化时，可以调用 `InvalidateVisual()` 来请求重新绘制。

## DrawingContext 操作

`DrawingContext` 提供了以下绘制操作：

| 方法 | 说明 |
|---|---|
| `DrawRectangle(brush, pen, rect, radiusX, radiusY)` | 绘制矩形，也可以是圆角矩形 |
| `DrawEllipse(brush, pen, center, radiusX, radiusY)` | 绘制椭圆 |
| `DrawLine(pen, p1, p2)` | 在两个点之间绘制直线 |
| `DrawGeometry(brush, pen, geometry)` | 绘制任意几何图形 |
| `DrawText(formattedText, origin)` | 在指定位置绘制格式化文本 |
| `DrawImage(bitmap, sourceRect, destRect)` | 绘制位图图像 |
| `DrawGlyphRun(brush, glyphRun)` | 绘制预排版的文字字形 |

### 带状态的绘制

可以使用 `PushClip`、`PushOpacity`、`PushTransform` 等方法来修改绘制状态。这些方法会返回一个 `IDisposable`，在释放时恢复到之前的状态：

```csharp
public override void Render(DrawingContext context)
{
    // 裁剪到一个圆角矩形区域
    using (context.PushClip(new RoundedRect(new Rect(Bounds.Size), 8)))
    {
        // 填充背景
        context.DrawRectangle(Brushes.White, null, new Rect(Bounds.Size));

        // 对一组绘制操作应用透明度
        using (context.PushOpacity(0.5))
        {
            context.DrawEllipse(Brushes.Red, null,
                new Point(30, 30), 20, 20);
        }

        // 应用变换
        using (context.PushTransform(Matrix.CreateRotation(0.2)))
        {
            context.DrawRectangle(Brushes.Blue, null,
                new Rect(10, 10, 50, 50));
        }
    }
}
```

### 绘制文本

使用 [`FormattedText`](/api/avalonia/media/formattedtext) 可以测量并渲染单段文本：

```csharp
public override void Render(DrawingContext context)
{
    var text = new FormattedText(
        "Hello, Avalonia!",
        CultureInfo.CurrentCulture,
        FlowDirection.LeftToRight,
        new Typeface("Segoe UI"),
        16,
        Brushes.Black);

    context.DrawText(text, new Point(10, 10));
}
```

`FormattedText` 适合简单的、单行的或较短的文本。如果需要多行文本、自动换行、两端对齐，或者逐行度量信息，请改用 `TextLayout`。`TextLayout` 支持一些 `FormattedText` 不具备的能力，例如 `TextAlignment.Justify`：

```csharp
var layout = new TextLayout(
    "Multi-line text with wrapping and justification support.",
    new Typeface("Segoe UI"),
    16,
    Brushes.Black,
    textAlignment: TextAlignment.Justify,
    maxWidth: 200);

// 访问每一行的度量信息
foreach (var line in layout.TextLines)
{
    // line.Height, line.Width, line.Start, and other metrics
}

layout.Draw(context, new Point(10, 10));
```

### 绘制图像

加载并绘制位图图像：

```csharp
private IImage? _image;

protected override void OnLoaded(RoutedEventArgs e)
{
    base.OnLoaded(e);
    var uri = new Uri("avares://MyApp/Assets/photo.png");
    _image = new Bitmap(AssetLoader.Open(uri));
    InvalidateVisual();
}

public override void Render(DrawingContext context)
{
    if (_image is null) return;

    var destRect = new Rect(0, 0, Bounds.Width, Bounds.Height);
    var sourceRect = new Rect(0, 0, _image.Size.Width, _image.Size.Height);

    context.DrawImage(_image, sourceRect, destRect);
}
```

## 使视觉内容失效并重绘

框架会缓存 `Render` 的结果。当控件数据发生变化时，你必须显式请求重绘：

```csharp
public static readonly StyledProperty<double> ProgressProperty =
    AvaloniaProperty.Register<ProgressRing, double>(nameof(Progress));

public double Progress
{
    get => GetValue(ProgressProperty);
    set => SetValue(ProgressProperty, value);
}

static ProgressRing()
{
    // 当 Progress 改变时自动使视觉内容失效
    AffectsRender<ProgressRing>(ProgressProperty);
}
```

`AffectsRender` 会注册一个回调，使得 `Progress` 的任何变化都会自动触发 `InvalidateVisual()`。当然，你也可以在需要时手动调用 `InvalidateVisual()`。

## RenderTargetBitmap

如果你想把控件的渲染结果捕获为位图（例如保存到文件，或做图像处理），可以这样做：

```csharp
var pixelSize = new PixelSize(
    (int)myControl.Bounds.Width,
    (int)myControl.Bounds.Height);

var renderTarget = new RenderTargetBitmap(pixelSize, new Vector(96, 96));
renderTarget.Render(myControl);

// 保存到文件
renderTarget.Save("output.png");
```

:::note
`RenderTargetBitmap` 使用的是软件渲染。那些依赖 GPU 特定渲染路径的控件（例如 `OpenGlControlBase` 或自定义 GPU 互操作内容）在用这种方式捕获时，可能无法正确渲染。
:::

:::tip[Offscreen rendering]
`RenderTargetBitmap.Render` 要求目标控件必须附加到一个可见窗口上。如果你需要在不显示窗口的情况下渲染控件（例如服务端生成图片或批量导出），请使用启用了 Skia 渲染器的 [headless platform](/docs/testing/setting-up-the-headless-platform)。Headless 平台会在内存中提供完整的布局与渲染管线，而无需真正打开窗口。
:::

## 用于 SkiaSharp 的 ICustomDrawOperation

如果你需要直接访问 SkiaSharp 画布（例如绘制复杂图表、3D 内容或游戏图形），可以实现 `ICustomDrawOperation`：

```csharp
using Avalonia.Rendering.SceneGraph;
using Avalonia.Skia;
using SkiaSharp;

public class ChartControl : Control
{
    public override void Render(DrawingContext context)
    {
        context.Custom(new ChartDrawOperation(new Rect(Bounds.Size)));
    }

    private class ChartDrawOperation : ICustomDrawOperation
    {
        public ChartDrawOperation(Rect bounds) => Bounds = bounds;

        public Rect Bounds { get; }

        public void Render(ImmediateDrawingContext context)
        {
            var feature = context.TryGetFeature<ISkiaSharpApiLeaseFeature>();
            if (feature is null) return;

            using var lease = feature.Lease();
            var canvas = lease.SkCanvas;

            // 使用完整的 SkiaSharp API
            using var paint = new SKPaint
            {
                IsAntialias = true,
                Style = SKPaintStyle.Stroke,
                StrokeWidth = 2,
                Color = SKColors.DodgerBlue
            };

            var path = new SKPath();
            path.MoveTo(0, (float)Bounds.Height);
            path.LineTo((float)Bounds.Width * 0.25f, (float)Bounds.Height * 0.6f);
            path.LineTo((float)Bounds.Width * 0.5f, (float)Bounds.Height * 0.8f);
            path.LineTo((float)Bounds.Width * 0.75f, (float)Bounds.Height * 0.2f);
            path.LineTo((float)Bounds.Width, (float)Bounds.Height * 0.4f);

            canvas.DrawPath(path, paint);
        }

        public bool HitTest(Point p) => Bounds.Contains(p);
        public bool Equals(ICustomDrawOperation? other) => false;
        public void Dispose() { }
    }
}
```

添加所需的 NuGet 包：

```xml
<PackageReference Include="Avalonia.Skia" Version="11.2.*" />
<PackageReference Include="SkiaSharp" Version="2.88.*" />
```

## 通过合成表面进行 GPU 互操作

对于视频播放、3D 引擎集成、跨进程 GPU 纹理共享等高级场景，Avalonia 的 composition API 支持将外部 GPU 资源导入到 `CompositionDrawingSurface` 中。

### 导入 GPU 图像

你可以使用 compositor 导入外部 GPU 纹理（例如 Vulkan 图像或 macOS 上的 IOSurface），并把它们显示在 composition surface 中：

```csharp
var compositor = ElementComposition.GetElementVisual(this)!.Compositor;

// 导入外部 GPU 图像句柄
var image = compositor.ImportGpuImage(
    PlatformGraphicsExternalImageProperties.CreateForVulkan(
        pixelSize, format),
    new PlatformHandle(handle, handleType));

// 导入用于 GPU 同步的信号量
var waitSemaphore = compositor.ImportGpuSemaphore(
    new PlatformHandle(semaphoreHandle, semaphoreType));
var signalSemaphore = compositor.ImportGpuSemaphore(
    new PlatformHandle(semaphoreHandle2, semaphoreType));

// 使用导入的图像更新 surface
await surface.UpdateWithKeyedMutexAsync(image);
// or with timeline semaphores (macOS Metal):
await surface.UpdateWithTimelineSemaphoresAsync(
    image,
    waitSemaphore, waitValue,
    signalSemaphore, signalValue);
```

### 支持的句柄类型

GPU 图像和信号量的句柄类型因平台而异。你可以使用 `KnownPlatformGraphicsExternalImageHandleTypes` 和 `KnownPlatformGraphicsExternalSemaphoreHandleTypes` 来查看可用类型：

| 平台 | 图像句柄类型 | 信号量句柄类型 |
|---|---|---|
| Windows | `D3D11TextureNtHandle`, `VulkanOpaqueNtHandle` | `D3D11Fence`, `VulkanOpaqueNtHandle` |
| macOS | `IOSurfaceRef` | `MetalSharedEvent` |
| Linux | `DmaBuf`, `VulkanOpaqueFd` | `VulkanOpaqueFd` |

请检查导入图像上的 `CompositionGpuImportedImageSynchronizationCapabilities`，以确定可用的同步方式（`KeyedMutex`、`Semaphores`、`TimelineSemaphores`）。

## CompositionCustomVisualHandler

[`CompositionCustomVisualHandler`](/api/avalonia/rendering/composition/compositioncustomvisualhandler) 提供逐帧回调，并且这些回调直接运行在渲染线程上，而不会阻塞 UI 线程。这对于平滑连续动画，或对 UI 线程开销较敏感的实时可视化场景非常有用。

对于那些可以接受 UI 线程回调的简单场景，则可以改用 [`TopLevel.RequestAnimationFrame`](/docs/fundamentals/top-level#requestanimationframe)。

### 设置自定义 visual handler

创建一个 `CompositionCustomVisualHandler`，并将它注册到控件的 composition visual 上：

```csharp
public class RenderThreadAnimationControl : Control
{
    private CompositionCustomVisualHandler? _handler;

    protected override void OnAttachedToVisualTree(VisualTreeAttachmentEventArgs e)
    {
        base.OnAttachedToVisualTree(e);

        var visual = ElementComposition.GetElementVisual(this);
        if (visual == null) return;

        var compositor = visual.Compositor;

        _handler = new CompositionCustomVisualHandler(
            OnRender, OnMessage);

        compositor.CreateCustomVisual(_handler);
    }

    private void OnRender(
        CompositionCustomVisualHandler sender,
        SkiaSharp.SKCanvas canvas,
        RenderBounds bounds)
    {
        // 这里运行在渲染线程上。
        // 直接使用 SkiaSharp 画布进行绘制。
        using var paint = new SkiaSharp.SKPaint
        {
            IsAntialias = true,
            Color = SkiaSharp.SKColors.CornflowerBlue
        };
        canvas.DrawCircle(
            (float)bounds.Width / 2,
            (float)bounds.Height / 2,
            50f, paint);

        // 请求下一帧，以持续渲染
        sender.RequestNextFrameRendering();
    }

    private void OnMessage(
        CompositionCustomVisualHandler sender,
        object message)
    {
        // 处理通过 sender.SendHandlerMessage(data)
        // 从 UI 线程发来的消息
    }
}
```

### 在线程之间通信

由于 `OnRender` 运行在渲染线程上，因此你不能直接访问 UI 线程状态。你可以使用 `SendHandlerMessage`，把数据从 UI 线程传递给渲染回调：

```csharp
// 在 UI 线程上：把更新后的数据发送到渲染线程
_handler?.SendHandlerMessage(new AnimationData(progress: 0.5));
```

消息会在 `OnMessage` 回调中到达，你可以把它保存下来，并在下一次渲染过程中使用。这种模式可以让 UI 线程保持响应，同时由渲染线程负责绘制。

### 何时使用 CompositionCustomVisualHandler

| 方式 | 所在线程 | 适用场景 |
|---|---|---|
| `TopLevel.RequestAnimationFrame` | UI 线程 | 简单的逐帧更新、属性动画循环 |
| `CompositionCustomVisualHandler` | 渲染线程 | 实时可视化、游戏循环、视频渲染 |
| `Render()` override | UI 线程 | 标准自定义控件绘制 |

## 性能注意事项

- `Render` 运行在 UI 线程上，因此绘制操作应尽可能快，并尽量避免额外分配。
- 当参数不变时，请复用 `Pen`、`Brush` 和 `FormattedText` 对象。可以把它们存为字段，并只在输入参数变化时重新创建。
- 对于需要频繁重绘的控件（例如图表、仪表盘），请使用 `AffectsRender` 来避免不必要的重绘。
- 对于复杂场景，可以考虑把控件拆分成更小的控件，以便只有发生变化的部分需要重新绘制。
- `ICustomDrawOperation` 会绕过 Avalonia 的场景图缓存。只有在你确实需要 SkiaSharp 级别控制时，才应使用它。

## 另请参阅

- [TopLevel.RequestAnimationFrame](/docs/fundamentals/top-level#requestanimationframe)：UI 线程上的逐帧回调。
- [Composition Animations](/docs/graphics-animation/composition-animations)：使用 composition API 的渲染线程属性动画。
- [Shapes and Geometries](/docs/graphics-animation/shapes-and-geometries)：内置形状控件与几何类型。
- [绘制控件](/docs/custom-controls/drawing-custom-controls)：创建能够自行绘制的自定义控件。
- [Brushes](/docs/graphics-animation/brushes)：用于填充和描边的可用画刷类型。
- [Effects](/docs/graphics-animation/effects)：阴影、模糊与其他视觉效果。
