---
id: performance
title: 性能优化
---

## 使用 ReadyToRun 减少启动时间

XPF 应用可以从 ReadyToRun（R2R）编译中显著受益，它会将程序集预编译为本机代码。Microsoft 提供的 WPF 库通常默认已经以 R2R 形式预编译，但 XPF 库默认不是。

在你的 `.csproj` 中启用 ReadyToRun：

```xml
<PropertyGroup>
    <PublishReadyToRun>true</PublishReadyToRun>
</PropertyGroup>
```

然后使用运行时标识符发布：

```bash
dotnet publish -r linux-x64 -c Release
```

这可以显著减少应用启动时间，尤其是在 Linux 嵌入式设备上。

:::note
在 Linux 上，ReadyToRun 可能会改变本机 `.so` 库的解析方式。详情请参见 [Linux: Native Library Resolution](/xpf/platforms/linux#native-library-resolution-with-readytorun)。
:::

## 渲染性能

### 配置 Skia 和合成选项

XPF 使用 Skia 作为渲染引擎。你可以通过 [自定义初始化](/xpf/configuration/customizing-initialization) 中的 `SkiaOptions` 和 `CompositionOptions` 来调优渲染性能：

```csharp
using Avalonia;
using Avalonia.Controls;
using Avalonia.Controls.ApplicationLifetimes;
using Avalonia.Skia;
using AvaloniaUI.Xpf;

AppBuilder.Configure<AvaloniaUI.Xpf.Helpers.DefaultXpfAvaloniaApplication>()
    .UsePlatformDetect()
    .With(new SkiaOptions
    {
        MaxGpuResourceSizeBytes = 512 * 1024 * 1024 // 512 MB GPU resource cache
    })
    .With(new CompositionOptions
    {
        UseRegionDirtyRectClipping = true
    })
    .WithAvaloniaXpf()
    .SetupWithLifetime(new ClassicDesktopStyleApplicationLifetime
    {
        ShutdownMode = ShutdownMode.OnExplicitShutdown
    });
```

- `MaxGpuResourceSizeBytes`：增大 GPU 纹理缓存，减少具有大量可视元素的应用重新上传资源的次数。
- `UseRegionDirtyRectClipping`：仅限制在屏幕上发生变化的区域进行渲染，从而提升局部更新场景下的性能。

### 模糊效果

模糊效果（`BlurEffect`、带模糊的 `DropShadowEffect`）在 Skia 中计算开销很大。启用了模糊效果的复杂 UI 可能会将帧率从 60fps 降低到 30fps 或更低。如果你关注渲染性能：

- 尽可能移除或降低模糊半径
- 考虑使用纯色背景替代 acrylic/模糊效果
- 尽早在目标硬件上进行测试

## 动态加载 XAML

`XamlReader.Load` 会在运行时解析并实例化 XAML。对于大型 XAML 文档，这可能会阻塞 UI 线程数秒。这是 WPF 共有的一项根本性限制。

提升动态 XAML 性能的策略：

- **将 XAML 预编译到程序集**：如果 XAML 内容在构建时已知，可将其编译到单独的程序集，并在运行时动态加载该程序集。编译后的 XAML（BAML）加载速度明显快于直接解析原始 XAML。
- **拆分大型 XAML**：将大型 XAML 文档拆分为更小的部分，并逐步加载。
- **在后台线程加载**：在后台线程中解析 XAML 字符串，然后在 UI 线程上实例化生成的对象树。

:::note
BAML（编译后的 XAML）提供最佳加载性能，但目前没有受支持的公共 API 用于创建 loose BAML 文件。请改为将 XAML 编译到程序集。
:::

## 嵌入高性能内容

对于对性能要求较高的渲染场景（例如实时仪表、音频可视化或 3D 内容），可以考虑使用 [AvaloniaHost](/xpf/interop/embedding-avalonia-in-xpf) 将 Avalonia 控件嵌入到你的 XPF 应用中。Avalonia 的 `CompositionCustomVisuals` API 允许直接在合成线程上进行渲染，完全绕过 WPF 分发器。

对于 OpenGL 内容，请参见 [OpenGL sample](https://github.com/AvaloniaUI/Avalonia-XPF-Samples/tree/master/src/OpenGLSample)，其中演示了如何使用 `ICompositionGpuInterop` 在 XPF 窗口中嵌入 OpenGL 渲染。