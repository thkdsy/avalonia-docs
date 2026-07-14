---
id: embedded-linux
title: 嵌入式 Linux
---

Avalonia 支持在没有桌面环境的嵌入式 Linux 设备上运行。这包括 Raspberry Pi 一类的单板计算机、工业面板、信息亭以及收银终端等。Avalonia 不再通过 X11 之类的窗口系统进行渲染，而是直接使用 Linux framebuffer 或 Direct Rendering Manager（DRM）输出到显示硬件。

## 显示输出：framebuffer 与 DRM

在桌面 Linux 系统中，应用通常绘制到由合成器管理的窗口中（例如 X11 或 Wayland）。而嵌入式设备通常没有合成器，因此应用会通过两种内核接口之一，直接把像素写入显示硬件。

### Linux framebuffer (`/dev/fb0`)

Linux framebuffer（fbdev）是这两种接口中较老也较简单的一种。它把显示设备暴露为一块可以写入的连续内存区域。内核会把显存映射到用户空间，对这块内存的写入就会显示到屏幕上。

**工作方式：**
1. 内核驱动把 GPU 的扫描输出缓冲区映射到 `/dev/fb0`。
2. 应用打开该设备，调用 `mmap()`，并获得指向像素数据的指针。
3. 向这段内存写入 RGB 值后，显示器会在下一次刷新时更新画面。

**优点：**
- 编程模型简单。
- 几乎适用于所有 Linux 系统，包括非常老的内核。
- 不要求 GPU 加速。

**局限：**
- 没有硬件加速渲染（不支持 OpenGL/Vulkan）。
- 默认是单缓冲，更新时可能出现明显撕裂。
- 同一时间只能有一个应用使用 framebuffer。
- fbdev 子系统在 Linux 内核中已处于维护模式，新硬件驱动通常都转向 DRM。

### Direct Rendering Manager (DRM/KMS)

DRM 是 Linux 内核中用于管理 GPU 和显示控制器的现代子系统。DRM 中的 Kernel Mode Setting（KMS）组件负责显示配置，例如分辨率、刷新率以及当前正在输出的缓冲区。

**工作方式：**
1. 应用打开 `/dev/dri/card0`（或其他显卡设备）。
2. 它通过 KMS API 查询可用的连接器（HDMI、DSI 等）、编码器和 CRTC。
3. 它分配 GPU 缓冲区（称为 GEM 对象），并创建一个指向该内存的 framebuffer 对象。
4. 它调用 `drmModeSetCrtc` 或 `drmModePageFlip` 来显示缓冲区。
5. 双缓冲是内建的：应用在后台缓冲区中绘制，同时前台缓冲区正在显示，然后再执行翻页。

**优点：**
- 可通过 OpenGL ES 或 Vulkan 实现硬件加速渲染（前提是 GPU 支持）。
- 通过 page-flip 实现无撕裂显示。
- 支持多平面叠加合成。
- Linux 内核仍在积极开发，并为所有现代 SoC 提供驱动支持。

**局限：**
- API 比 framebuffer 更复杂。
- 需要支持 KMS 的 GPU 驱动（现代 ARM SoC 通常都支持，但某些非常老旧或冷门的硬件可能不支持）。

### 应该选哪个

| 因素 | Framebuffer | DRM |
|---|---|---|
| 硬件加速 | 否 | 是（取决于 GPU 支持） |
| 撕裂 | 可能出现 | 通过 page-flip 实现无撕裂 |
| 内核支持状态 | 维护模式 | 持续活跃开发 |
| 配置复杂度 | 较低 | 中等 |
| 是否推荐用于新项目 | 否 | 是 |

**新项目应优先使用 DRM。** framebuffer 接口虽然能工作，但已经被视为旧方案。DRM 提供更好的性能、更平滑无撕裂的渲染，也是 Linux 内核的发展方向。只有在硬件缺少支持 KMS 的 GPU 驱动时，才建议退回到 framebuffer。

## 单视图应用生命周期

桌面 Avalonia 应用使用 `IClassicDesktopStyleApplicationLifetime`，它会创建窗口。而在嵌入式 Linux 上没有窗口管理器，因此 Avalonia 改用 `ISingleViewApplicationLifetime`。这种生命周期只提供一个填满整个显示设备的根视图。

你的 `App.axaml.cs` 应同时处理这两种生命周期，这样应用既能在桌面环境中运行（便于开发），也能在嵌入式目标设备上运行：

```csharp
public override void OnFrameworkInitializationCompleted()
{
    if (ApplicationLifetime is IClassicDesktopStyleApplicationLifetime desktop)
        desktop.MainWindow = new MainWindow();
    else if (ApplicationLifetime is ISingleViewApplicationLifetime singleView)
        singleView.MainView = new MainSingleView();

    base.OnFrameworkInitializationCompleted();
}
```

一个常见模式是先创建一个承载实际 UI 的 `MainView` UserControl，然后同时在 `MainWindow`（桌面）和 `MainSingleView`（嵌入式）中复用它。这样你就可以先在工作站上开发和调试，再把同一套 UI 部署到目标设备。

## 启用 DRM 输出

向项目中添加 `Avalonia.LinuxFramebuffer` 包：

```bash
dotnet add package Avalonia.LinuxFramebuffer
```

在 `Program.cs` 中检查是否传入 `--drm` 命令行参数，并启动对应的生命周期：

```csharp
public static int Main(string[] args)
{
    var builder = BuildAvaloniaApp();
    if (args.Contains("--drm"))
    {
        SilenceConsole();
        // Avalonia 会自动检测输出显卡。
        // 如果要显式指定，可使用：card: "/dev/dri/card1"
        return builder.StartLinuxDrm(args, card: null, options: new DrmOutputOptions
        {
            Scaling = 1.0,
        });
    }

    return builder.StartWithClassicDesktopLifetime(args);
}

private static void SilenceConsole()
{
    new Thread(() =>
    {
        Console.CursorVisible = false;
        while (true)
            Console.ReadKey(true);
    })
    { IsBackground = true }.Start();
}
```

`SilenceConsole` 方法会接管控制台输入并隐藏光标。否则，闪烁的文本光标会显示在应用输出之上。

## 显示缩放

`DrmOutputOptions` 上的 `Scaling` 属性用于控制 DPI 缩放因子，也支持小数值。例如，在 1920x1080 显示器上设置 `Scaling = 1.5`，会让控件看起来更大，效果类似于渲染到一个 1280x720 的逻辑表面。

```csharp
return builder.StartLinuxDrm(args, card: null, options: new DrmOutputOptions
{
    Scaling = 1.5,
});
```

为了获得最清晰的效果，建议选择一个能把物理分辨率整除成整数结果的缩放值。对于 1920x1080 显示器，像 `1.25`（1536x864）和 `1.5`（1280x720）这类值通常效果不错。非整数结果（例如 `1.3`）虽然也能工作，但边缘可能会不够锐利，因为 `UseLayoutRounding` 无法在所有边界上都精确对齐。

## 屏幕方向

许多嵌入式显示器会以竖屏或旋转方向安装。如果 Linux 显示驱动不支持硬件旋转，Avalonia 可以通过 `DrmOutputOptions` 上的 `Orientation` 属性，以软件方式旋转渲染输出。

```csharp
return builder.StartLinuxDrm(args, card: null, options: new DrmOutputOptions
{
    Scaling = 1.0,
    Orientation = SurfaceOrientation.Rotation90,
});
```

可用值包括：

| 值 | 旋转 |
|---|---|
| `SurfaceOrientation.Rotation0` | 不旋转（默认） |
| `SurfaceOrientation.Rotation90` | 顺时针旋转 90 度 |
| `SurfaceOrientation.Rotation180` | 旋转 180 度 |
| `SurfaceOrientation.Rotation270` | 顺时针旋转 270 度 |

触摸输入坐标会自动适配所配置的方向。旋转方向只能在启动时设置，应用运行期间无法动态更改。

:::note
旋转会使用离屏 framebuffer 和 OpenGL shader 来变换图像。当使用 `Rotation0` 时，不会产生额外性能开销。
:::

## 验证 DRM 配置

在运行 Avalonia 应用前，你可以使用 `kmscube` 来验证 DRM 是否正常工作：

```bash
sudo apt-get install kmscube
sudo kmscube
```

如果屏幕上出现一个旋转立方体，就说明 DRM 工作正常，Avalonia 也应该能够正常渲染。

## 触摸输入

嵌入式设备通常会把触摸屏作为主要输入方式。Avalonia 通过 `libinput` 读取触摸事件，而它已包含在上面列出的依赖库中。通过 DRM 运行时，触摸输入会自动工作。

如果你的应用需要屏幕键盘（例如信息亭、收银系统或任何没有物理键盘的设备），请参阅 [虚拟键盘](/docs/platform-specific-guides/embedded-linux/virtual-keyboard) 指南。

## 另请参阅

- [在 Raspberry Pi 上运行](/docs/platform-specific-guides/embedded-linux/raspberry-pi)：分步骤的硬件实践教程
- [虚拟键盘](/docs/platform-specific-guides/embedded-linux/virtual-keyboard)：了解屏幕键盘支持
- [部署到嵌入式 Linux](/docs/deployment/embedded-linux)
- [桌面 Linux](/docs/platform-specific-guides/linux)：了解 X11 和 Wayland 环境
