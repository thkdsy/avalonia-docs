---
id: cross-platform-architecture
title: 跨平台架构
description: 理解 Avalonia 如何在不同平台之间共享代码，以及如何处理平台差异。
doc-type: explanation
---

Avalonia 使用 Skia 渲染控件，而不是去包装各个平台的原生控件。这意味着你的 AXAML 视图、视图模型以及业务逻辑都可以在所有平台上产生一致的结果。本页将介绍哪些内容可以共享、哪些内容需要平台特定处理，以及应该如何选择合适的方案。

## Avalonia 的代码共享方式

由于 Avalonia 自己负责绘制控件，因此你可以在 Windows、macOS、Linux、iOS、Android 和浏览器上获得一致的外观和行为。在一个典型的 Avalonia 应用程序中，以下内容通常都可以完全共享：

- **视图**（AXAML 文件和 code-behind）
- **视图模型和业务逻辑**（普通的 C# 或 F# 类）
- **样式和主题**
- **平台服务**，例如 [文件选择器](/docs/services/storage/storage-provider)、[剪贴板](/docs/services/clipboard)、[启动器](/docs/services/launcher)、[深色模式检测](/docs/services/platform-settings) 和 [安全区域处理](/docs/services/insets-manager)

有些场景则可能需要平台特定代码，例如：

- 硬件传感器（GPS、加速度计、陀螺仪）
- 推送通知
- 蓝牙、摄像头和生物识别
- 系统托盘以及其他操作系统级的外壳集成能力

对于 Avalonia 没有抽象封装的设备 API，你可以考虑使用 [Microsoft.Maui.Essentials](https://www.nuget.org/packages/Microsoft.Maui.Essentials)，它在 .NET 8 及以上版本中可以与 Avalonia 协同工作。不过需要注意，Maui.Essentials 并不覆盖 Linux、浏览器或非 Catalyst 的 macOS 目标。

## 组织你的解决方案

标准的 Avalonia 跨平台模板会创建一组项目，以最大化代码共享：

| 项目 | 用途 |
|---|---|
| Core | 视图、视图模型、业务逻辑（由所有平台共享） |
| Desktop | Windows、macOS 和 Linux 的入口项目 |
| Android | Android 的入口项目 |
| iOS | iOS、iPadOS 和 Mac Catalyst 的入口项目 |
| Browser | WebAssembly 的入口项目 |

核心项目包含了绝大多数代码。各个平台特定项目通常只是很薄的一层入口，并引用核心项目。完整示例可参阅 [Setting up a cross-platform solution](/docs/app-development/cross-platform-solution-setup)。

## 处理平台差异

当你确实需要平台特定行为时，Avalonia 和 .NET 提供了四种处理方式，下面按从最简单到最灵活的顺序说明。

### OnPlatform 与 OnFormFactor

对于 UI 层面的调整，可以直接在 AXAML 中使用 `OnPlatform` 或 `OnFormFactor` 标记扩展：

```xml
<TextBlock Text="{OnPlatform 'Welcome', iOS='Welcome (iOS)', Android='Welcome (Android)'}"/>
```

`OnPlatform` 用于针对特定操作系统，`OnFormFactor` 则用于针对设备类别，例如 Desktop 或 Mobile。完整语法请参阅 [Platform-specific XAML](/docs/platform-specific-guides/xaml)。

### 运行时检测

如果你只是在 C# 中做简单分支判断，可以使用 `OperatingSystem` 类：

```csharp
if (OperatingSystem.IsWindows())
{
    // Windows-specific logic
}
```

这种方式无需修改项目结构，几乎在所有场景下都可使用。完整 API 说明请参阅 [Platform-specific .NET](/docs/platform-specific-guides/dotnet)。

### 条件编译

如果代码需要调用平台特定 API，可以结合操作系统目标框架使用 C# 预处理指令：

```csharp
#if ANDROID
    var orientation = GetAndroidOrientation();
#elif IOS
    var orientation = GetiOSOrientation();
#else
    var orientation = DeviceOrientation.Undefined;
#endif
```

这要求你的项目支持多目标编译（例如 `net8.0;net8.0-ios;net8.0-android`）。配置细节请参阅 [Platform-specific .NET](/docs/platform-specific-guides/dotnet)。

### 接口抽象

对于更复杂的平台特性，可以在共享项目中定义接口，并由各个平台分别实现：

```csharp
// In Core project
public interface IDeviceOrientation
{
    DeviceOrientation GetOrientation();
}

// In platform-specific project
public class AndroidDeviceOrientation : IDeviceOrientation
{
    public DeviceOrientation GetOrientation() { /* Android APIs */ }
}
```

将各个平台实现注册到依赖注入容器中，这样共享代码在使用它们时就不需要关心自己当前运行在哪个平台上。完整示例请参阅 [Platform-specific .NET](/docs/platform-specific-guides/dotnet)，DI 配置请参阅 [Dependency injection](/docs/app-development/dependency-injection)。

## 如何选择方案

| 方案 | 最适合的场景 | 代价 |
|---|---|---|
| OnPlatform / OnFormFactor | UI 微调（间距、文本、控件） | 仅限 XAML |
| 运行时检测 | 简单的运行时分支判断 | 所有平台代码都会进入每个二进制产物 |
| 条件编译 | 平台特定 API 调用 | 需要多目标编译 |
| 接口抽象 | 复杂的平台特性 | 文件更多，并且需要 DI |

建议先从能满足需求的最简单方案开始，只有在必要时再升级到更灵活的方式。

## 另请参阅

- [The MVVM pattern](/docs/fundamentals/the-mvvm-pattern)
- [Setting up a cross-platform solution](/docs/app-development/cross-platform-solution-setup)
- [Platform-specific .NET](/docs/platform-specific-guides/dotnet)
- [Platform-specific XAML](/docs/platform-specific-guides/xaml)
- [Dependency injection](/docs/app-development/dependency-injection)
- [Application lifetimes](/docs/fundamentals/application-lifetimes)
