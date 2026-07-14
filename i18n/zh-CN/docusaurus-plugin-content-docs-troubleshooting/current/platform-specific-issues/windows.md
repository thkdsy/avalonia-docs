---
id: windows
title: Windows 问题
sidebar_label: Windows
description: 排查 Windows 特有的 Avalonia 问题，包括签名、渲染、缩放以及深色主题标题栏问题。
doc-type: troubleshooting
---

## 打包和签名

#### 已签名的可执行文件仍然触发 SmartScreen 警告

对于新证书和新应用程序来说，这是正常现象。不同类型的证书有不同的信任建立周期：

- **EV 证书和 Microsoft Trusted Signing**：可立即绕过 SmartScreen。
- **OV 证书**：需要建立信誉（通常需要 3 到 6 个月的持续分发）。

要使用 OV 证书加快信誉建立：

1. 将你签名后的应用程序提交到 [Microsoft Intelligent Security Graph (ISG)](https://www.microsoft.com/en-us/wdsi/filesubmission) 进行分析。
2. 通过知名下载来源分发你的应用程序，以便 Windows Defender 遥测可以建立信任。
3. 确保每个版本都使用同一张证书签名。切换证书会重置你的信誉。

如果 SmartScreen 在几个月后仍然阻止你的应用程序，请验证证书链是否完整，以及在签名期间你的时间戳服务器是否可访问。证书链不完整或缺少时间戳会阻止信誉累积。

#### Azure 身份验证失败

如果通过 Azure Trusted Signing 签名时出现身份验证错误：

1. 验证以下环境变量是否已正确设置：`AZURE_TENANT_ID`、`AZURE_CLIENT_ID`、`AZURE_CLIENT_SECRET`。
2. 确认服务主体已在 Azure 门户中分配 **Trusted Signing Certificate Profile Signer** 角色。
3. 检查你的 Azure Trusted Signing 账户和证书配置文件是否位于同一区域。
4. 如果你使用托管标识而不是服务主体，请确保托管环境（例如 Azure DevOps 或 GitHub Actions）支持它，并且该标识已正确关联。
5. 作为替代方案，安装 [Azure CLI](https://learn.microsoft.com/en-us/cli/azure/?view=azure-cli-latest) 并按照 [az login](https://learn.microsoft.com/en-us/cli/azure/reference-index?view=azure-cli-latest#az-login) 流程进行交互式身份验证，以排除凭据问题。

## 渲染

#### 应用程序显示空白或黑色窗口

如果你的应用程序窗口出现但没有内容：

1. 检查你是否在远程桌面会话或没有 GPU 直通的虚拟机下运行。Avalonia 在这些环境中会回退到软件渲染，但某些配置仍可能失败。
2. 强制使用软件渲染，以确认问题是否与 GPU 有关。在构建应用程序之前，将以下内容添加到你的 `Program.cs` 中：

```csharp
AppBuilder.Configure<App>()
    .UsePlatformDetect()
    .With(new Win32PlatformOptions
    {
        RenderingMode = new[]
        {
            Win32RenderingMode.Software
        }
    })
    .StartWithClassicDesktopLifetime(args);
```

3. 更新你的 GPU 驱动程序。过时或有问题的驱动程序是 Windows 上渲染失败最常见的原因。

#### 窗口调整大小时闪烁或出现视觉伪影

窗口调整大小时的闪烁是 Win32 窗口模型的一个已知限制。要减少它：

- 在顶层 `Window` 上设置 `Background`，使清除颜色与你的应用程序主题匹配。这样可以让闪烁不那么明显。
- 避免在调整大小时触发大量布局重新计算。尽可能使用 `LayoutTransformControl` 或固定大小的内部面板。

## 高 DPI 和缩放

#### 控件在高 DPI 显示器上看起来过小或过大

Avalonia 会尊重 Windows 上按显示器设置的 DPI。如果你的应用程序以意外的缩放比例渲染：

1. 验证你的应用程序清单没有覆盖 DPI 感知设置。如果你有 `app.manifest` 文件，请确保它声明了按显示器 DPI 感知（或者移除任何与 DPI 相关的条目，让 Avalonia 来处理）。
2. 检查 **设置 > 显示 > 缩放与布局** 中的显示缩放百分比。Avalonia 应该会自动匹配这个值。
3. 如果你将 Avalonia 嵌入到 WPF 或 WinForms 宿主中，宿主应用程序的 DPI 感知模式优先。确保宿主配置为 per-monitor V2 感知。

#### 位图或图像资源看起来模糊

当图像在高 DPI 屏幕上显得模糊时，请为资源提供多个分辨率变体。Avalonia 会为当前 DPI 选择最佳匹配。你可以将缩放后的变体与基础资源并排放置：

```text
/Assets/logo.png        (1x, base)
/Assets/logo@2x.png     (2x, for 200% scaling)
```

## 主题和外观

#### 在 Windows 10 上切换到深色主题时标题栏仍保持浅色

在 Windows 10 上，原生标题栏不会自动跟随应用程序的 `RequestedThemeVariant`。这是平台限制：Windows 10 没有提供用于将标题栏变为深色的官方 API。在 Windows 11 上，Avalonia 会自动处理这一点。

如果你需要在 Windows 10 上使用深色标题栏，你有两个选择：

- **使用自定义标题栏。** 设置 `ExtendClientAreaToDecorationsHint="True"` 和 `SystemDecorations="None"`，以便完全控制主题并绘制自己的标题栏。详情请参见 [自定义标题栏](/docs/platform-specific-guides/windows#custom-title-bars)。
- **使用未记录的 DWM API。** 通过 P/Invoke 调用 `DwmSetWindowAttribute`，并使用 `DWMWA_USE_IMMERSIVE_DARK_MODE`（属性 20）。这适用于 Windows 10 build 18985 及更高版本，但它没有文档记录，可能会在未来的 Windows 更新中发生变化。

```csharp
using System.Runtime.InteropServices;
using Avalonia;

[DllImport("dwmapi.dll", PreserveSig = true)]
private static extern int DwmSetWindowAttribute(
    IntPtr hwnd, int attr, ref int value, int size);

private void SetDarkTitleBar(Window window, bool isDark)
{
    if (!OperatingSystem.IsWindows()) return;

    var handle = window.TryGetPlatformHandle()?.Handle;
    if (handle is null) return;

    int value = isDark ? 1 : 0;
    // Attribute 20: DWMWA_USE_IMMERSIVE_DARK_MODE
    DwmSetWindowAttribute(handle.Value, 20, ref value, sizeof(int));
}
```

在窗口打开后以及主题发生变化时调用此方法（订阅 `ActualThemeVariantChanged`）。

## 窗口行为

#### 窗口位置或大小未正确恢复

如果你在会话之间保存并恢复窗口边界，请注意显示器配置可能会发生变化。在应用之前，始终使用 `Screens.All` 验证恢复的坐标是否与当前屏幕布局匹配。位于屏幕外的窗口对用户不可见。

#### 无边框或自定义 Chrome 窗口缺少任务栏图标

如果你的窗口使用 `SystemDecorations="None"` 或自定义标题栏，并且没有出现在任务栏中，请确保你没有无意中将 `ShowInTaskbar="False"`。Win32 应用的一些扩展窗口样式也可能会抑制任务栏条目。显式设置 `ShowInTaskbar="True"` 在大多数情况下都可以解决这个问题。

## 另请参见

- [Windows 平台指南](/docs/platform-specific-guides/windows) 中有关透明度、Mica 和 Win32 集成的详细信息
- [macOS 问题](/troubleshooting/platform-specific-issues/macos) 和 [WebAssembly 问题](/troubleshooting/platform-specific-issues/webassembly) 中的其他平台特定排查内容
- [日志记录错误和警告](/docs/app-development/logging-errors-and-warnings) 了解如何从应用程序中捕获诊断输出