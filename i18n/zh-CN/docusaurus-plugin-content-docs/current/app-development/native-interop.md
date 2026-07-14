---
id: native-interop
title: 原生平台互操作
description: 在 Avalonia 应用中访问原生平台 API、嵌入原生视图并使用 P/Invoke。
doc-type: overview
---

Avalonia 提供了多种机制，用于与原生平台 API 交互，并在应用中嵌入原生内容。

## 平台特定代码模式

对于简单的平台分支逻辑，可以使用运行时检测或条件编译。相关模式请参阅 [Cross-Platform Architecture](/docs/fundamentals/cross-platform-architecture)。

而对于更复杂的原生集成场景，Avalonia 还提供了对底层平台窗口句柄和原生 API 的直接访问能力。

## 访问原生窗口句柄

你可以获取原生窗口句柄，以便与平台 API 进行互操作：

```csharp
if (TopLevel.GetTopLevel(this)?.TryGetPlatformHandle() is { } handle)
{
    // handle.Handle 就是原生句柄
    IntPtr nativeHandle = handle.Handle;

    // handle.HandleDescriptor 告诉你句柄类型
    string kind = handle.HandleDescriptor;
}
```

返回的句柄类型取决于平台：

| 平台 | HandleDescriptor | 原生类型 |
|---|---|---|
| Windows | `HWND` | Win32 窗口句柄 |
| macOS | `NSWindow` | AppKit 窗口指针 |
| Linux (X11) | `X11` | X11 Window ID |
| iOS | `UIViewControlHandle` | UIKit 视图引用 |
| Android | `AndroidViewControlHandle` | Android 视图引用 |
| Browser | `JSObjectControlHandle` | 容器 `<div>` 元素引用 |

这在以下场景中很有用：
- 通过操作系统 API 注册全局热键
- 调用原生窗口函数（例如 Win32 的 `SetWindowPos`）
- 将窗口句柄传给需要父窗口的原生库
- 与平台特定的移动端 SDK 集成

## 嵌入原生视图

Avalonia 支持通过 [`NativeControlHost`](/api/avalonia/controls/nativecontrolhost) 在 Avalonia 视觉树中嵌入原生 UI 控件。这让你能够在 Avalonia 布局内部使用平台特定控件，例如原生浏览器、媒体播放器或地图视图。

### NativeControlHost

`NativeControlHost` 是一种控件，它会在 Avalonia 布局中预留一块区域，并在该区域中承载原生视图：

```csharp
public class NativeTextEditor : NativeControlHost
{
    protected override IPlatformHandle CreateNativeControlCore(
        IPlatformHandle parent)
    {
        if (OperatingSystem.IsWindows())
        {
            // 创建一个 Win32 EDIT 控件
            var hwnd = CreateWindowEx(0, "EDIT", "",
                WS_CHILD | WS_VISIBLE | ES_MULTILINE,
                0, 0, 100, 100,
                parent.Handle, IntPtr.Zero, IntPtr.Zero, IntPtr.Zero);

            return new PlatformHandle(hwnd, "HWND");
        }

        return base.CreateNativeControlCore(parent);
    }

    protected override void DestroyNativeControlCore(
        IPlatformHandle control)
    {
        if (OperatingSystem.IsWindows())
        {
            DestroyWindow(control.Handle);
        }
        else
        {
            base.DestroyNativeControlCore(control);
        }
    }
}
```

你可以像使用其他控件一样在 XAML 中使用 `NativeControlHost`：

```xml
<Border BorderBrush="Gray" BorderThickness="1">
    <local:NativeTextEditor MinHeight="200" />
</Border>
```

### 原生嵌入的限制

原生视图会位于 Avalonia 渲染表面的上层。这意味着：

- **不支持透明**：原生视图不能拥有透明背景来显示其后方的 Avalonia 内容。
- **不支持变换**：Avalonia 的渲染变换（旋转、缩放）不会作用到原生视图上。
- **Z 顺序受限**：原生视图始终渲染在 Avalonia 内容之上。你不能把 Avalonia 控件叠放到原生视图上方。
- **裁剪有限**：原生视图会被裁剪到宿主边界内，但不支持复杂的裁剪几何。

## P/Invoke 与原生库

调用原生 C 库时，可以使用标准的 .NET P/Invoke：

```csharp
using System.Runtime.InteropServices;

public static partial class NativeMethods
{
    // .NET 7+ 的 LibraryImport（兼容 AOT）
    [LibraryImport("user32")]
    public static partial int MessageBoxW(
        IntPtr hWnd,
        [MarshalAs(UnmanagedType.LPWStr)] string text,
        [MarshalAs(UnmanagedType.LPWStr)] string caption,
        uint type);
}
```

对于 Native AOT 部署，建议使用 `LibraryImport` 而不是 `DllImport`，以确保封送代码在编译时生成。更多细节请参阅 [Native AOT Deployment](/docs/deployment/native-aot)。

### 加载平台特定的原生库

把原生库放到平台对应的 `runtimes` 文件夹中：

```text
MyApp/
├── runtimes/
│   ├── win-x64/native/mylib.dll
│   ├── osx-arm64/native/libmylib.dylib
│   └── linux-x64/native/libmylib.so
```

然后在 `.csproj` 中引用它们：

```xml
<ItemGroup>
    <NativeLibrary Include="runtimes\win-x64\native\mylib.dll"
                   Pack="true"
                   PackagePath="runtimes/win-x64/native/" />
</ItemGroup>
```

.NET 运行时会自动为当前平台加载正确的库。

## 结合依赖注入的平台专属服务

对于复杂的原生集成场景，可以在共享项目中定义服务接口，再在各个平台分别实现：

```csharp
// 共享项目
public interface INativeNotification
{
    void ShowNotification(string title, string message);
}

// Windows 实现
public class WindowsNotification : INativeNotification
{
    public void ShowNotification(string title, string message)
    {
        // 使用 Windows Toast Notification API
    }
}

// macOS 实现
public class MacNotification : INativeNotification
{
    public void ShowNotification(string title, string message)
    {
        // 使用 NSUserNotificationCenter
    }
}
```

在启动时注册对应平台的实现：

```csharp
if (OperatingSystem.IsWindows())
    services.AddSingleton<INativeNotification, WindowsNotification>();
else if (OperatingSystem.IsMacOS())
    services.AddSingleton<INativeNotification, MacNotification>();
```

完整配置方式请参阅 [Dependency Injection](/docs/app-development/dependency-injection)。

## 使用 Microsoft.Maui.Essentials

对于常见的设备 API（传感器、网络连接、电池、权限等），`Microsoft.Maui.Essentials` 提供了可在 .NET 8+ 的 Avalonia 中使用的跨平台抽象：

```xml
<PackageReference Include="Microsoft.Maui.Essentials" Version="8.0.0" />
```

```csharp
using Microsoft.Maui.Devices;

var model = DeviceInfo.Model;
var platform = DeviceInfo.Platform;
```

请注意，Maui.Essentials 支持 Windows、macOS（通过 Catalyst）、Android 和 iOS。它不支持 Linux、WebAssembly，也不支持非 Catalyst 的 macOS 构建。

## 使用 SkiaSharp 进行自定义渲染

如果你希望在 Avalonia 控件内部直接进行 GPU 渲染，可以结合 `ICustomDrawOperation` 与 SkiaSharp：

```csharp
using Avalonia.Media;
using Avalonia.Platform;
using Avalonia.Rendering.SceneGraph;
using Avalonia.Skia;
using SkiaSharp;

public class SkiaCanvas : Control
{
    public override void Render(DrawingContext context)
    {
        var bounds = new Rect(0, 0, Bounds.Width, Bounds.Height);
        context.Custom(new SkiaDrawOperation(bounds));
    }

    private class SkiaDrawOperation : ICustomDrawOperation
    {
        public SkiaDrawOperation(Rect bounds) => Bounds = bounds;

        public Rect Bounds { get; }

        public void Render(ImmediateDrawingContext context)
        {
            var leaseFeature = context.TryGetFeature<ISkiaSharpApiLeaseFeature>();
            if (leaseFeature is null) return;

            using var lease = leaseFeature.Lease();
            var canvas = lease.SkCanvas;

            // 直接使用 SkiaSharp 绘制
            using var paint = new SKPaint
            {
                Color = SKColors.CornflowerBlue,
                IsAntialias = true,
                Style = SKPaintStyle.Fill
            };

            canvas.DrawCircle(
                (float)Bounds.Width / 2,
                (float)Bounds.Height / 2,
                50,
                paint);
        }

        public bool HitTest(Point p) => Bounds.Contains(p);
        public bool Equals(ICustomDrawOperation? other) => false;
        public void Dispose() { }
    }
}
```

添加 SkiaSharp NuGet 包：

```xml
<PackageReference Include="SkiaSharp" Version="2.88.*" />
```

## 另请参阅

- [Cross-Platform Architecture](/docs/fundamentals/cross-platform-architecture)：解决方案结构与平台分支模式。
- [Platform-Specific .NET](/docs/platform-specific-guides/dotnet)：运行时检测与条件编译。
- [Dependency Injection](/docs/app-development/dependency-injection)：注册平台服务。
- [Native AOT Deployment](/docs/deployment/native-aot)：原生互操作下的 AOT 注意事项。
