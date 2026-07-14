---
id: dotnet
title: 平台特定 .NET
---

## 概述

.NET 中的条件编译允许根据特定条件决定某些代码是否参与编译。这在处理需要在不同平台或不同开发环境下表现不同行为的代码时尤其有用。

这些方案都不是 Avalonia 特有的，它们可以用于任何类型的项目。

## 运行时条件

.NET 6 及更高版本提供了一组可在运行时判断操作系统的 API，即 [OperatingSystem](https://learn.microsoft.com/en-us/dotnet/api/system.operatingsystem)。

这个类型中常用的静态方法包括：
| 方法 | 说明 |
| --- | --- |
| IsWindows() | 指示当前应用是否运行在 Windows 上。 |
| IsLinux() | 指示当前应用是否运行在 Linux 上。 |
| IsMacOS() | 指示当前应用是否运行在 macOS 上。 |
| IsAndroid() | 指示当前应用是否运行在 Android 上。 |
| IsIOS() | 指示当前应用是否运行在 iOS 或 MacCatalyst 上。 |
| IsBrowser() | 指示当前应用是否作为浏览器中的 WASM 运行。 |
| IsOSPlatform(String) | 指示当前应用是否运行在指定平台上。 |

这些方法不需要改动项目结构，并且可以在任何位置使用。
它们的缺点是：无法在编译期隔离平台特定 API。否则就意味着公共程序集必须引用平台专属依赖。

这种方式适合较简单的场景，或者你希望保持项目结构尽可能简单时使用。

:::note
对于 Linux 来说，这也是编写条件化 .NET 代码的唯一通用方式，因为 .NET 目前并没有专门针对 Linux 的 Target Framework。
:::

## 条件编译

C# 本身支持通过 `#if`、`#elif`、`#else`、`#endif` 进行条件编译，参见 [C# 预处理器指令](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/preprocessor-directives#conditional-compilation)。

`DEBUG` 是一个众所周知的编译期常量，但它对于编写平台特定代码帮助不大。
根据项目使用的 [操作系统特定 Target Framework](https://learn.microsoft.com/en-us/dotnet/standard/frameworks#net-5-os-specific-tfms)，C# 编译器还可能定义额外常量：

| Target Framework | 常量 |
|----|----|
| net8.0 | - |
| net8.0-windows | WINDOWS |
| net8.0-macos | MACOS |
| net8.0-browser | BROWSER |
| net8.0-ios | IOS |
| net8.0-android | ANDROID |
从这张表可以看出几点：
1. 如果项目没有使用任何操作系统特定 Target Framework，那么这些常量一个都不会被定义。
2. **没有 LINUX 常量**，因为目前还不存在 `net8.0-linux` 这样的 Target Framework。未来版本的 .NET 可能会改变这一点。
3. 此外，`net8.0-browser` 只从 .NET 8 SDK 开始提供；其他这些 Target Framework 在 .NET 6 或更高版本中即可使用。

:::note
如果有需要，类似的方法也可以用于为 .NET Framework 或 .NET Standard 项目定义特殊的条件编译逻辑。更多信息请参阅微软文档 [Cross-platform targeting](https://learn.microsoft.com/en-us/dotnet/standard/library-guidance/cross-platform-targeting)。
:::

### 实际示例

假设我们想在 C# 代码中使用平台 API。它可以是 Avalonia API、Xamarin API，或者其他任何平台相关 API。
首先，需要在项目中定义预期的 Target Framework。为简化示例，我们在 `.csproj` 中使用三个目标框架：`net8.0`（默认）、`net8.0-ios` 和 `net8.0-android`：

```xml
<PropertyGroup>
    <TargetFrameworks>net8.0;net8.0-ios;net8.0-android</TargetFrameworks>
</PropertyGroup>
```

然后就可以写出如下方法：
```csharp
public enum DeviceOrientation
{
    Undefined,
    Landscape,
    Portrait
}

public static DeviceOrientation GetOrientation()
{
#if ANDROID
            IWindowManager windowManager = Android.App.Application.Context.GetSystemService(Context.WindowService).JavaCast<IWindowManager>();
            SurfaceOrientation orientation = windowManager.DefaultDisplay.Rotation;
            bool isLandscape = orientation == SurfaceOrientation.Rotation90 || orientation == SurfaceOrientation.Rotation270;
            return isLandscape ? DeviceOrientation.Landscape : DeviceOrientation.Portrait;
#elif IOS
            UIInterfaceOrientation orientation = UIApplication.SharedApplication.StatusBarOrientation;
            bool isPortrait = orientation == UIInterfaceOrientation.Portrait || orientation == UIInterfaceOrientation.PortraitUpsideDown;
            return isPortrait ? DeviceOrientation.Portrait : DeviceOrientation.Landscape;
#else
            return DeviceOrientation.Undefined;
#endif
}
```

:::note
这段示例代码改编自微软文档：https://learn.microsoft.com/en-us/dotnet/maui/platform-integration/invoke-platform-code?view=net-maui-8.0#conditional-compilation
:::


## 平台特定项目

和前一种方式类似，你也可以为每个平台创建单独的启动项目，并保留一个包含主要逻辑与布局的共享项目。
例如，默认的 Avalonia.Xplat 模板会创建如下项目结构：

| 项目 | Target Framework |
| --- | --- |
| Project.Shared | net8.0 |
| Project.Desktop | net8.0 |
| Project.Android | net8.0-android |
| Project.iOS | net8.0-ios |
| Project.Browser | net8.0-browser |

Desktop 项目会统一承载 Windows、macOS 和 Linux，而移动端与浏览器平台则各自拥有独立项目。
这是 Avalonia 项目的默认做法。如果有需要，开发者也可以把 Desktop 项目进一步拆分。
不过需要注意，.NET SDK 目前依然没有 Linux 专属的 target framework，因此 Linux 仍然只能使用通用的 `net8.0`。

通常在需要平台特定代码时，会先在共享项目中定义一个接口，再由各个平台项目分别实现。
如果把前面的示例改造成这种结构，大致会像下面这样：
```csharp title='Project.Shared IDeviceOrientation.cs'
public interface IDeviceOrientation
{
    DeviceOrientation GetOrientation();
}
```

```csharp title='Project.Android AndroidDeviceOrientation.cs'
public class AndroidDeviceOrientation : IDeviceOrientation
{
    public DeviceOrientation GetOrientation()
    {
        IWindowManager windowManager = Android.App.Application.Context.GetSystemService(Context.WindowService).JavaCast<IWindowManager>();
        SurfaceOrientation orientation = windowManager.DefaultDisplay.Rotation;
        bool isLandscape = orientation == SurfaceOrientation.Rotation90 || orientation == SurfaceOrientation.Rotation270;
        return isLandscape ? DeviceOrientation.Landscape : DeviceOrientation.Portrait;
    }
}
```

```csharp title='Project.iOS iOSDeviceOrientation.cs'
public class iOSDeviceOrientation : IDeviceOrientation
{
    public DeviceOrientation GetOrientation()
    {
        UIInterfaceOrientation orientation = UIApplication.SharedApplication.StatusBarOrientation;
        bool isPortrait = orientation == UIInterfaceOrientation.Portrait || orientation == UIInterfaceOrientation.PortraitUpsideDown;
        return isPortrait ? DeviceOrientation.Portrait : DeviceOrientation.Landscape;
    }
}
```

随后可以使用你选择的依赖注入库来注册这些实现，或者使用静态注册表属性进行注册。

## 另请参阅

- [平台特定 XAML](/docs/platform-specific-guides/xaml)
- [部署到 Android](/docs/deployment/android)
- [部署到 iOS](/docs/deployment/ios)
