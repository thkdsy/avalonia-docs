---
id: attaching-applications
title: 附加应用程序
doc-type: how-to
tags:
  - avalonia plus
  - avalonia pro
  - avalonia enterprise
---

## 附加浏览器或移动端应用

本页假设两个应用都部署在同一局域网或同一台机器上。如需远程连接，请参阅[附加到远程工具](/tools/developer-tools/attaching-to-the-remote-tool)。

:::note

对于所有平台，都可以在共享项目中安装 `AvaloniaUI.DiagnosticsSupport` 包。同时，也可以将 `this.AttachDeveloperTools()` 代码保留在共享的 `Application` 类中。如果某个平台需要自定义配置，可以使用 `OperatingSystem.IsPlatform` API。

:::

## 附加浏览器应用

1. 按照[快速开始](/tools/developer-tools/installation)完成初始设置。
2. 运行 `Developer Tools` 应用。也可以在命令行中使用 `avdt` dotnet 工具。
3. 通过 `dotnet run` 运行浏览器应用，或对已发布项目使用 `dotnet serve`。更多细节请参阅 [Avalonia WebAssembly 文档](/docs/platform-specific-guides/webassembly)。
4. 默认情况下，浏览器项目会在启动时自动附加到 `Developer Tools`。详见 [DeveloperToolsOptions.ConnectOnStartup](/tools/developer-tools/options)。

![Browser with Developer Tools](/img/tools/dev-tools/attaching-to-browser.png)

:::note

为避免与 Chrome Developer Tools 冲突，可以将 Avalonia 工具的快捷键从 <kbd>F12</kbd> 改为自定义按键。详见 [DeveloperToolsOptions.Gesture](/tools/developer-tools/options)。

:::

## 附加 iOS 应用

1. 按照[快速开始](/tools/developer-tools/installation)完成初始设置。
2. 运行 `Developer Tools` 应用。也可以在命令行中使用 `avdt` dotnet 工具。
3. 从你的 IDE 运行 iOS 应用。更多细节请参阅 [Avalonia iOS 文档](/docs/platform-specific-guides/ios)。
4. 确保浏览器应用处于焦点状态，以便快捷键能够被拦截。按下 <kbd>F12</kbd>。

![iOS with Developer Tools](/img/tools/dev-tools/attaching-to-ios.png)

## 附加 Android 应用

1. 按照[快速开始](/tools/developer-tools/installation)完成初始设置。
2. 重要：Android 默认不允许任何 HTTP 流量：
   1. 创建 `Resources/xml/network_security_config.xml` 文件：
   ```xml
   <?xml version="1.0" encoding="utf-8"?>
   <network-security-config>
     <domain-config cleartextTrafficPermitted="true">
       <domain includeSubdomains="true">10.0.2.2</domain> <!-- Debug address -->
     </domain-config>
   </network-security-config>
   ```
   2. 在 `AndroidManifest.xml` 文件中，更新 `<application>` XML 节点：
   ```xml
   <application android:networkSecurityConfig="@xml/network_security_config">
   ```
   3. 更多细节请参阅 https://devblogs.microsoft.com/xamarin/cleartext-http-android-network-security/
   4. 本仓库中的 `SimpleToDoList.Android` 项目也包含这些改动，可供参考。
3. 运行 `Developer Tools` 应用。也可以在命令行中使用 `avdt` dotnet 工具。
4. 从你的 IDE 运行 Android 应用。更多细节请参阅 [Avalonia Android 文档](/docs/platform-specific-guides/android)。
5. 默认情况下，Android 项目会在启动时自动附加到 `Developer Tools`。详见 [DeveloperToolsOptions.ConnectOnStartup](/tools/developer-tools/options)。

![Android with Developer Tools](/img/tools/dev-tools/attaching-to-android.png)

:::note

`10.0.2.2` 是 Android 上用于替代 `localhost` 的默认 IP 地址。模拟器会将其映射到目标主机。如需覆盖该地址，请参阅 [DeveloperToolsOptions.Protocol](/tools/developer-tools/options)。

:::

## 附加 WSL2 应用

WSL2 允许你从 Windows 主机调试和运行 Linux 应用。
你也可以按照同样的步骤，在 WSL2 系统中安装完整的 Developer Tools 进程，但这可能会与 Windows 版本形成重复安装。

为了保持单一安装，通常更推荐将运行在 Linux 中的应用附加到运行在 Windows 中的 Developer Tools 实例。

1. 按照[快速开始](/tools/developer-tools/installation)完成初始设置和 NuGet 包安装。
2. 对 WSL2 机器进行一次性配置：

    - （推荐）按照 [Mirrored mode networking](https://learn.microsoft.com/en-us/windows/wsl/networking#mirrored-mode-networking) 文档启用 WSL2 镜像网络模式。该模式会让 Windows 的 `localhost` 可直接在 WSL 实例中访问，无需额外配置或代码修改。

    - （备选）按照 WSL2 文档 [Accessing Windows networking apps from Linux (host IP)](https://learn.microsoft.com/en-us/windows/wsl/networking#accessing-windows-networking-apps-from-linux-host-ip) 获取 Windows 主机 IP 地址，然后在 `AttachDeveloperTools` 选项中使用它：

    ```csharp
    this.AttachDeveloperTools(o =>
    {
        o.Protocol = DeveloperToolsProtocol.CreateHttp(IPAddress.Parse("YOUR_LOCAL_NETWORK_HOST_IP"));
    });
    ```

3. 在 Windows 主机上运行 `Developer Tools` 实例，通常通过 `avdt` dotnet 工具命令行启动。
4. 运行你的 Linux 应用，并通过 <kbd>F12</kbd> 附加到 `Developer Tools`。

![Attaching to WSL2](/img/tools/dev-tools/attaching-wsl.png)
## 另请参阅

- [附加到远程工具](/tools/developer-tools/attaching-to-the-remote-tool)
- [Developer Tools 安装](/tools/developer-tools/installation)
