---
id: linux
title: 桌面 Linux
description: 了解 Avalonia 如何在桌面 Linux 上运行，包括 WSL 2 配置以及基于 AT-SPI2 的无障碍支持。
doc-type: guide
---

## Avalonia 如何在 Linux 上运行

Avalonia 在 Windows 上使用 Win32 API，在 macOS 上使用自己的原生 Objective-C++ 后端，而在 Linux 上则直接面向 X11。大多数支持 .NET SDK 且具备 X11 或 framebuffer 能力的 Linux 发行版都可以运行 Avalonia 应用。

:::note
Wayland 支持将在 Avalonia 12.0 中提供。
:::

## WSL 2 (Windows Subsystem for Linux)

[WSL 2](https://learn.microsoft.com/en-us/windows/wsl/) 是 Windows 的一项功能，它允许你在 Windows 中直接运行完整的 Linux 环境，而不需要传统虚拟机或双系统。这对于希望在 Windows 工作流中构建和测试 Linux 应用的开发者非常有用。

Avalonia 可以在 WSL 2 发行版中运行，但某些通常会预装在完整桌面发行版中的库，需要手动安装：

```bash
sudo apt install libice6 libsm6 libfontconfig1
```

## Accessibility

Avalonia 通过 **AT-SPI2**（Assistive Technology Service Provider Interface，辅助技术服务提供者接口）协议，在 Linux 上把无障碍树暴露给辅助技术。这使得像 Orca 这样的屏幕阅读器能够发现并与 Avalonia 控件交互，包括播报控件名称、读取文本内容以及跟踪焦点变化。

当 D-Bus 会话总线可用且无障碍服务正在运行时，AT-SPI2 支持会自动启用。你的应用无需额外配置。

### 使用 Orca 测试

[Orca](https://orca.gnome.org/) 是大多数基于 GNOME 的发行版中的默认屏幕阅读器。要验证应用的无障碍支持，请按以下步骤操作：

1. 如果尚未安装 Orca，请先安装：
   ```bash
   sudo apt install orca
   ```
2. 在桌面环境中启用无障碍功能。在 GNOME 中，打开 **Settings > Accessibility** 并启用 **Screen Reader** 开关，或者直接从终端启动 Orca：
   ```bash
   orca &
   ```
3. 运行你的 Avalonia 应用。控件获得焦点时，Orca 应该会进行播报。

### 使用 Accerciser 测试

[Accerciser](https://gitlab.gnome.org/GNOME/accerciser) 是一个交互式无障碍查看器，可显示 AT-SPI2 树。它非常适合用来验证控件是否正确暴露了角色、名称和状态：

```bash
sudo apt install accerciser
accerciser &
```

在与运行中的应用交互时，你可以在 Accerciser 中浏览树结构，以检查每个控件暴露了哪些信息。

关于如何让应用具备良好的无障碍支持，请参阅 [无障碍](/docs/app-development/accessibility)。

## 另请参阅

- [支持的平台](/docs/supported-platforms)：查看 Linux 发行版支持层级
- [部署到桌面 Linux](/docs/deployment/linux)
- [嵌入式 Linux](/docs/platform-specific-guides/embedded-linux)：了解 framebuffer 与 DRM 场景
- [无障碍](/docs/app-development/accessibility)：了解自动化属性与自定义自动化 peer
