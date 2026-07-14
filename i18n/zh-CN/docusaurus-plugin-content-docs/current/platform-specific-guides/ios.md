---
id: ios
title: iOS
---

import IOSOpenXcodeScreenshot from '/img/guides/platform-specific-guides/ios/ios-open-xcode.png';
import IOSCreateXcodeProjectScreenshot from '/img/guides/platform-specific-guides/ios/ios-create-xcode-project.png';
import IOSSelectProjectOptionsScreenshot from '/img/guides/platform-specific-guides/ios/ios-select-project-options.png';
import IOSSelectAnyDeviceScreenshot from '/img/guides/platform-specific-guides/ios/ios-select-any-device.png';
import IOSAddAdditionalSimulatorsScreenshot from '/img/guides/platform-specific-guides/ios/ios-add-additional-simulators.png';
import IOSProvisionPhoneScreenshot from '/img/guides/platform-specific-guides/ios/ios-provision-phone.png';
import IOSSelectDeviceScreenshot from '/img/guides/platform-specific-guides/ios/ios-select-device.png';
import IOSCertScreenshot from '/img/guides/platform-specific-guides/ios/ios-cert.png';

## 配置开发环境

### 前置条件

在 Mac 上，你需要安装最新版本的 macOS 和 Xcode。

### 安装 SDK

首先，安装正确版本的 [dotnet SDK](https://dotnet.microsoft.com/en-us/download/dotnet/6.0) 非常重要。在撰写本文时，可用的最低 SDK 版本是 6.0.200。

### 安装 workload

```bash
dotnet workload install ios
```

:::info
你可能需要使用 `sudo` 运行该命令。\
\
你也可能需要卸载旧版本：`dotnet workload remove ios`
:::

这样你就可以在任意平台上构建 iOS 应用。但只有在你能够访问安装了 Xcode 的真实 macOS 硬件时，才能实际测试和运行这些应用。

## 使用 Xcode 为设备配置签名

如果要部署到真实的 iPhone 或 iPad，你必须先使用 Xcode 为设备完成配置。这会创建签名证书，并将设备与开发 provisioning profile 关联起来。

继续之前，请先按照这份指南创建免费的 Apple 开发者签名证书：[guide to create a free Apple developer signing certificate](https://docs.microsoft.com/en-us/xamarin/ios/get-started/installation/device-provisioning/free-provisioning)。

你需要创建一个 Xcode 应用项目，并确保它使用与你的 Avalonia 应用相同的 `bundle identifier`。

1. 打开 Xcode

<Image light={IOSOpenXcodeScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

2. 选择 Create a new Xcode project

<Image light={IOSCreateXcodeProjectScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

3. 选择 iOS 和 App，然后点击 Next。

<Image light={IOSSelectProjectOptionsScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

4. 为项目和 Organisation 输入名称，其余信息保持默认即可。

5. 选择一个目录保存项目。这个项目之后不一定需要保留，因此存放位置不用太在意。

6. 在顶部状态栏中点击 “Any device (arm64)”

<Image light={IOSSelectAnyDeviceScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

7. 在列表底部点击 “Add Additional Simulators...”

<Image light={IOSAddAdditionalSimulatorsScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

8. 点击 devices，并通过 USB 连接你的 iPhone 或 iPad。Xcode 将开始为设备配置开发环境。

<Image light={IOSProvisionPhoneScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

9. 在设备列表中选择你的 iPhone 或 iPad。

<Image light={IOSSelectDeviceScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

10. 点击运行按钮，应用将被安装并在你的手机上启动。

如果成功，你的设备现在就已经完成开发配置。若要找到代码签名密钥，请打开 **Keychain Access** 应用，并搜索 “development”。

<Image light={IOSCertScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

在你选中的开发证书窗口顶部，粗体显示的文本就是签名密钥值（例如 `Apple Development: dan@walms.co.uk (3L323F7VSS)`）。

## Mac Catalyst

Mac Catalyst 允许你在不单独编写 macOS 项目的情况下，在 macOS 上运行 Avalonia iOS 应用。当你的应用高度依赖 UIKit API，或者你希望把 Avalonia 嵌入到 MAUI 混合应用中（MAUI 在 macOS 桌面支持上使用 Catalyst）时，这会非常有用。

:::note
对于原生 macOS 开发，仍然推荐使用 Avalonia 基于 AppKit 的后端。它可以直接访问 macOS 窗口系统、Metal 渲染和原生菜单，而无需经过 Catalyst 转换层。详见 [macOS 平台指南](/docs/platform-specific-guides/macos)。
:::

### 配置 Mac Catalyst

1. 安装 Mac Catalyst workload：

```bash
dotnet workload install maccatalyst
```

2. 在 iOS 项目的 target frameworks 中加入 `net10.0-maccatalyst`：

```xml
<PropertyGroup>
    <TargetFrameworks>net10.0-ios;net10.0-maccatalyst</TargetFrameworks>
</PropertyGroup>
```

3. 以 Mac Catalyst 为目标构建并运行：

```bash
dotnet build -f net10.0-maccatalyst
dotnet run -f net10.0-maccatalyst
```

应用会通过 Apple 的 Catalyst 转换层以原生 macOS 应用形式运行。它会显示在 Dock 中，支持 macOS 窗口管理，并且可以通过 Mac App Store 分发。

### 何时使用 Mac Catalyst，何时使用默认 macOS 后端

对于大多数 Avalonia 应用来说，默认的 macOS 后端（Avalonia Native）是更好的选择。它使用一个轻量级原生库（`libAvaloniaNative.dylib`）来提供窗口、输入、渲染、菜单和无障碍支持，而无需依赖 .NET macOS 或 Catalyst workload。这意味着你可以在 Windows 或 Linux 上完成 macOS 应用的构建、打包和签名，开发阶段不需要 Mac。详见 [Avalonia 如何在 macOS 上运行](/docs/platform-specific-guides/macos#how-avalonia-runs-on-macos)。

Mac Catalyst 更适用于一些特定场景：例如与 UIKit API 深度耦合的应用，或者嵌入到 MAUI 混合项目中的应用（其 macOS 目标使用 Catalyst）。在这些情况下，Catalyst 可以让你直接复用 iOS 项目，而不必维护单独的 macOS 入口。

| 对比项 | 默认 macOS 后端 | Mac Catalyst |
|---|---|---|
| 可从 Windows 或 Linux 构建 | 是 | 否（需要 macOS） |
| 所需 .NET workload | 无（`net10.0` 即可） | `maccatalyst` workload |
| 原生 API 能力范围 | 较少（仅 UI 基础能力） | 通过 Catalyst 提供 UIKit 子集 |
| 是否与 iOS 共用项目 | 否（单独的 Desktop 项目） | 是（同一个 iOS 项目） |
| MAUI 混合嵌入 | 否 | 是 |
| 是否推荐用于新的 Avalonia 应用 | 是 | 否 |

## 深链接与通用链接

iOS 支持通过两种机制从 URL 打开你的应用：

### 自定义 URL scheme

在 `Info.plist` 中添加 `CFBundleURLTypes`，即可注册一个自定义 URL scheme（例如 `myapp://`）：

```xml
<key>CFBundleURLTypes</key>
<array>
    <dict>
        <key>CFBundleURLName</key>
        <string>MyApp</string>
        <key>CFBundleURLSchemes</key>
        <array>
            <string>myapp</string>
        </array>
    </dict>
</array>
```

### 通用链接

通用链接会将你的应用与某个 Web 域名关联起来，从而允许标准 `https://` URL 直接打开你的应用。配置步骤如下：

1. 为应用添加 Associated Domains entitlement。创建或更新 `Entitlements.plist`：
   ```xml
   <key>com.apple.developer.associated-domains</key>
   <array>
       <string>applinks:example.com</string>
   </array>
   ```
2. 在你的 Web 服务器上托管 `apple-app-site-association` 文件，路径为 `https://example.com/.well-known/apple-app-site-association`。

### 处理激活事件

无论是自定义 URL scheme 还是通用链接，都会在 `IActivatableLifetime` 上触发 `ActivationKind.OpenUri` 类型的 `Activated` 事件：

```csharp
if (Application.Current.TryGetFeature<IActivatableLifetime>() is { } activatableLifetime)
{
    activatableLifetime.Activated += (s, a) =>
    {
        if (a is ProtocolActivatedEventArgs protocolArgs
            && protocolArgs.Kind == ActivationKind.OpenUri)
        {
            // 处理 URI
            var uri = protocolArgs.Uri;
        }
    };
}
```

这与 Avalonia 12 中基于 scene 的生命周期兼容。完整 API 参考请参阅 [Activatable Lifetime](/docs/services/activatable-lifetime)。

## 另请参阅

- [部署到 iOS](/docs/deployment/ios)（模拟器、真机与发布）
- [macOS 平台指南](/docs/platform-specific-guides/macos)（原生 AppKit 后端）
- [Activatable Lifetime](/docs/services/activatable-lifetime)：用于处理 URI、文件和后台激活
