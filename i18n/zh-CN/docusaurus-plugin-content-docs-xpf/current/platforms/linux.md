---
id: linux
title: Linux
---

## 支持的发行版

以下 Linux 发行版经过了全面测试并受支持：

* **Debian**：9 及更新版本
* **Ubuntu**：16.04 及更新版本
* **Fedora**：30 及更新版本

### 其他发行版

除了上述发行版之外，Avalonia XPF 还可以在许多其他 Linux 发行版上运行。如果你使用的是未官方支持的发行版：

* Avalonia 支持团队可以帮助确保与你所选发行版的兼容性
* 发行版特定问题将按个案处理
* 可能需要额外的配置或测试

:::note
如果你计划部署到未列出的发行版上，请在开发过程早期联系支持团队。
:::


## 安装 .NET

许多发行版在其软件包仓库中提供 .NET 的版本，但**不应**使用这些版本，因为它们不包含所需的 `Microsoft.NET.Sdk.WindowsDesktop` SDK。你必须从 Microsoft 软件包源安装 .NET。

### Ubuntu

```bash
# 注册 Microsoft 软件包仓库
wget https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
rm packages-microsoft-prod.deb

# 安装 .NET SDK
sudo apt update
sudo apt install dotnet-sdk-8.0
```

### Debian

```bash
wget https://packages.microsoft.com/config/debian/$(cat /etc/debian_version | cut -d. -f1)/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
rm packages-microsoft-prod.deb

sudo apt update
sudo apt install dotnet-sdk-8.0
```

### Fedora

```bash
sudo dnf install dotnet-sdk-8.0
```

Fedora 在其默认仓库中包含了 Microsoft 的 .NET 软件包，并且这些软件包与 XPF 兼容。

### 其他发行版

对于其他发行版，请将 [Microsoft 软件包源](https://packages.microsoft.com/) 添加到你的软件包管理器中，并从那里安装 .NET SDK。

### 修复损坏的 .NET 安装

如果你之前是从发行版的软件包仓库（而不是从 Microsoft）安装的 .NET，则必须在从 Microsoft 软件包源安装之前将其完全卸载。混用软件包来源会导致冲突以及缺失 SDK 组件。

```bash
# 1. 删除发行版的 .NET 软件包
sudo apt remove 'dotnet*' 'aspnet*' 'netstandard*'   # Debian/Ubuntu
# 或
sudo dnf remove 'dotnet*' 'aspnet*' 'netstandard*'   # Fedora

# 2. 删除任何 .NET 安装目录
sudo rm -rf /usr/share/dotnet
sudo rm -rf /usr/lib/dotnet

# 3. 删除发行版的 .NET 软件包源，以防止它被重新安装
# 在 Ubuntu 上，检查并删除：
sudo rm /etc/apt/sources.list.d/*dotnet* 2>/dev/null
sudo rm /etc/apt/preferences.d/*dotnet* 2>/dev/null

# 4. 从 Microsoft 软件包源安装 .NET（见上面的说明）
```

:::danger
混合安装（部分软件包来自你的发行版，部分来自 Microsoft）会导致难以诊断的构建失败。如果 `dotnet --list-sdks` 没有显示 `Microsoft.NET.Sdk.WindowsDesktop`，则说明你的安装不正确。
:::

## 其他依赖项

运行 XPF 需要以下原生库：`libICE`、`libSM`、`fontconfig` 和 `libgdiplus`。

### Debian / Ubuntu

```bash
sudo apt install libice6 libsm6 libfontconfig1 libgdiplus
```

### Fedora

```bash
sudo dnf install libICE libSM fontconfig libgdiplus
```

### RHEL / CentOS / Rocky Linux / AlmaLinux

```bash
sudo dnf install libICE libSM fontconfig
```

`libgdiplus` 在默认的 RHEL 仓库中不可用。请从 EPEL 安装它：

```bash
sudo dnf install epel-release
sudo dnf install libgdiplus
```

### 其他发行版

使用你的发行版的软件包管理器安装等效的软件包。库名称在不同发行版之间可能有所不同（例如，Debian 上的 `libice6` 对应于 Fedora/RHEL 上的 `libICE`）。

## 为 Linux 发布

始终从命令行发布 XPF 应用程序，而不是从 Visual Studio 中发布。Visual Studio 的发布可能会生成不完整的输出，缺少原生库（例如 `libSkiaSharp.so`）。

```bash
dotnet publish -r linux-x64 -c Release
```

对于自包含部署：

```bash
dotnet publish -r linux-x64 -c Release --self-contained
```

:::caution
通过 Visual Studio 发布可能会省略关键的原生依赖项。如果你遇到 `libSkiaSharp` 的 `DllNotFoundException` 或类似错误，请切换到使用 CLI 发布。
:::

### 使用 ReadyToRun 时的原生库解析

使用 `PublishReadyToRun` 时，.NET 运行时可能会改变原生库的解析方式。如果你的应用程序在运行时找不到 `.so` 文件：

- 确保原生库与可执行文件位于同一目录
- 对于移动过位置的原生库，设置 `LD_LIBRARY_PATH` 以包含库目录
- 考虑使用自包含发布，它会将所有依赖项放在一起

## 在 Linux 上调试

### 从 Windows（Visual Studio）

要从 Windows 上的 Visual Studio 调试运行在 Linux 上的 XPF 应用程序：

1. 从命令行构建 linux-x64：
   ```bash
   dotnet publish -r linux-x64 -c Debug
   ```
2. 将输出复制到你的 Linux 机器或 WSL2 实例
3. 在 Linux 上运行应用程序
4. 在 Visual Studio 中，使用 **Debug > Attach to Process**，选择 WSL2 或 SSH 连接类型，并附加到正在运行的进程

### 从 VS Code

使用带有 C# DevKit 扩展的 VS Code。为通过 SSH 或 WSL2 的远程调试配置 `launch.json`。

### 从 JetBrains Rider

Rider 原生支持通过 SSH 进行远程调试，并通过 Gateway 功能支持 WSL2。

:::tip
在使用 XPF SDK 时，`net8.0-windows` 目标框架可在非 Windows 平台上工作。为 Linux 构建时，你不需要更改目标框架。
:::

## 托盘图标

XPF 通过 StatusNotifierItem/AppIndicator 协议支持 Linux 上的系统托盘图标。

**GNOME** 默认不包含托盘图标支持。安装 [AppIndicator GNOME Extension](https://extensions.gnome.org/extension/615/appindicator-support/) 以启用它。

**KDE Plasma** 原生支持托盘图标，无需额外配置。

## 较旧发行版上的文件对话框

在较旧的 Linux 发行版（如 RHEL 8）上，GNOME 版本可能过旧，无法支持基于 DBus 的文件对话框协议。这可能导致 `OpenFolderDialog.InitialDirectory` 和类似属性被忽略。

要解决此问题，请在你的 [自定义初始化](/xpf/configuration/customizing-initialization) 中禁用 DBus 文件选择器：

```csharp
AppBuilder.Configure<AvaloniaUI.Xpf.Helpers.DefaultXpfAvaloniaApplication>()
    .UsePlatformDetect()
    .With(new X11PlatformOptions { UseDBusFilePicker = false })
    .WithAvaloniaXpf()
    .SetupWithLifetime(new ClassicDesktopStyleApplicationLifetime
    {
        ShutdownMode = ShutdownMode.OnExplicitShutdown
    });
```

这会回退到 GTK 文件对话框，它支持旧系统上的 `InitialDirectory`。

## Win32 API shim 与原生 API 冲突

如果你的应用程序调用原生 Linux API（例如通过 `DllImport` 调用 X11 函数），同时又启用了 Win32 API shim，那么 shim 层可能会拦截这些原生调用并导致 `EntryPointNotFoundException`。

要解决此问题，请将原生 Linux API 调用移动到单独的程序集，并将其排除在 Win32 shim 之外：

```csharp
AvaloniaUI.Xpf.WinApiShim.WinApiShimSetup.AutoEnable(asm =>
{
    // 跳过包含原生 Linux API 调用的程序集
    if (asm.GetName().Name == "MyLinuxNativeInterop")
        return true; // true = 跳过此程序集
    return false;
});
```

或者，使用 `WinApiShimSetup.AddLibrary` 仅为特定程序集启用 shim，而不是使用 `AutoEnable`。

## 从 `systemd` 启动

在 X11 上将 XPF 应用程序作为 systemd 服务启动时，可能会出现竞态条件：应用程序在窗口管理器完全初始化之前就启动了。这可能导致 `WindowStyle="None"` 被忽略，从而显示标题栏。

请在你的 systemd 服务文件中添加启动延迟：

```ini
[Service]
ExecStartPre=/bin/sleep 5
ExecStart=/path/to/your/application
```

## Linux 上的 WebView

XPF 通过 `NativeWebDialog`（来自 `Avalonia.Xpf.Controls.WebView` NuGet 包）在 Linux 上提供网页内容嵌入。可嵌入的 `NativeWebView` 控件不受 Linux 支持。

`NativeWebDialog` 需要 webkit2gtk 4.1 版本：

### Debian / Ubuntu

```bash
sudo apt install libwebkit2gtk-4.1-dev
```

### Fedora

```bash
sudo dnf install webkit2gtk4.1-devel
```

### RHEL / CentOS

```bash
sudo dnf install webkit2gtk4.1-devel
```

如果你的 RHEL 版本中没有 `webkit2gtk4.1-devel`，请检查 EPEL 或更新的 AppStream 模块是否提供了它。

:::note
需要 webkit2gtk 4.1 版本。较旧的版本（4.0）不支持 `NativeWebDialog` 所需的所有功能。
:::

有关所有浏览器嵌入选项的比较，请参阅 [Web 内容嵌入](/xpf/interop/web-content)。

## 显示服务器注意事项

### X11

X11 是大多数 Linux 发行版上的默认显示服务器。XPF 在 X11 上运行良好，但请注意以下事项：

- **窗口管理器时序**：当在桌面会话早期启动 XPF 应用程序时（例如，从 systemd 服务启动），窗口管理器可能尚未完全初始化。这可能导致 `WindowStyle="None"` 被忽略。有关解决方法，请参阅 [从 systemd 启动](#launching-from-systemd)。
- **窗口消息**：Win32 窗口消息（例如 `WM_ACTIVATEAPP`）会由 shim 层在受支持的第三方控件所需的范围内进行模拟。并非所有消息都会生成。如果你的应用程序依赖特定窗口消息进行跨窗口通信，请考虑改用 .NET IPC 机制。
- **多显示器特性**：窗口定位和 DPI 行为可能因窗口管理器而异。请尽早在目标窗口管理器上进行测试。

### Wayland

Wayland 是较新的显示协议，最近版本的 Fedora 和 Ubuntu 默认使用它。XPF 通过 XWayland（X11 兼容层）支持 Wayland。

- **键盘隔离**：只有当前获得焦点的窗口会接收键盘输入。没有机制可将键盘输入定向到未获得焦点的窗口。如果你的应用程序需要在窗口之间隔离键盘输入（例如用于具有多个输入设备的 kiosk 场景），这是 Wayland 协议的限制。
- **窗口定位**：Wayland 不允许应用程序设置绝对窗口位置。`Window.Left` 和 `Window.Top` 可能会被合成器忽略。

## 已知限制

- **UI 测试自动化**：Avalonia 目前不支持 Linux 上的 AT-SPI2 辅助功能协议。依赖辅助功能 API 的自动化 UI 测试工具（如 pywinauto 或 Appium）功能有限。
- **透明窗口点击穿透**：与 macOS 类似，XPF 在 Linux 上不支持点击穿过窗口的透明区域。对透明区域的鼠标点击会被窗口捕获，而不会传递给下方的窗口。对于叠加场景，请将内容嵌入单个窗口，而不是叠加透明窗口。
- **Wayland 键盘隔离**：在 Wayland 合成器上，只有当前获得焦点的窗口会接收键盘输入。没有机制可将键盘输入定向到未获得焦点的窗口。
- **远程桌面透明度**：某些远程桌面工具（如 MobaXterm）不支持 alpha 通道透明度，这可能导致透明窗口显示为白色背景。这是远程桌面工具的限制，不是 XPF 的限制。请通过测试原生 Linux 应用程序（例如启用透明度的 Konsole）来验证透明度行为。
- **NativeControlHost**：`NativeControlHost`（用于嵌入原生 Linux 控件）在 Linux 上的支持有限。如果你需要嵌入原生内容，请考虑改用 Avalonia 的 composition API。