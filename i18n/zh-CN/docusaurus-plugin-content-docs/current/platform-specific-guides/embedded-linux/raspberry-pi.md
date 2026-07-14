---
id: raspberry-pi
title: 在 Raspberry Pi 上运行 Avalonia
---

import RaspbianLiteDrmKmsCubeScreenshot from '/img/guides/platform-specific-guides/raspberry-pi/raspbian-lite-drm-kmscube.gif';
import RaspbianLiteDrmDesktopScreenshot from '/img/guides/platform-specific-guides/raspberry-pi/raspbian-lite-drm-desktop.jpg';
import RaspbianLiteRaspberryScreenshot from '/img/guides/platform-specific-guides/raspberry-pi/raspbian-lite-drm-run-on-raspberry.jpg';

## 所需硬件

将 Raspbian Stretch（2018-11-13）刷入一张 8GB SD 卡中。`balenaEtcher` 是一个很适合完成这项工作的工具。

插入 SD 卡并启动 `Raspberry Pi`。

你也可以参考这篇 [Raspbian 和 .NET Core 配置指南](https://blogs.msdn.microsoft.com/david/2017/07/20/setting_up_raspian_and_dotnet_core_2_0_on_a_raspberry_pi/)。下面对后续步骤做了简要汇总。

## 安装所需软件包

* 安装 `curl`、`libunwind8`、`gettext` 和 `apt-transport-https`。其中 `curl` 与 `apt-transport-https` 往往已经是最新版本。

```bash
sudo apt-get install curl libunwind8 gettext apt-transport-https
```

* 下载 tar 包。

```bash
curl -sSL -o dotnet.tar.gz https://dotnetcli.blob.core.windows.net/dotnet/Runtime/release/2.0.0/dotnet-runtime-latest-linux-arm.tar.gz
```

* 将 tar 包解压到 `/opt/dotnet`。

```bash
sudo mkdir -p /opt/dotnet && sudo tar zxf dotnet.tar.gz -C /opt/dotnet
```

* 创建 `dotnet` 可执行文件的链接。

```bash
sudo ln -s /opt/dotnet/dotnet /usr/local/bin
```

另一种方式：你可以先切换到超级用户（执行 `sudo su`）

```bash
apt-get -y install curl libunwind8 gettext apt-transport-https
curl -sSL -o dotnet.tar.gz https://dotnetcli.blob.core.windows.net/dotnet/Runtime/release/2.0.0/dotnet-runtime-latest-linux-arm.tar.gz
mkdir -p /opt/dotnet && sudo tar zxf dotnet.tar.gz -C /opt/dotnet
ln -s /opt/dotnet/dotnet /usr/local/bin
```

:::note
注意脚本的换行符。它应使用 `LF` 而不是 `CR LF`。请将脚本保存为 `.sh` 文件，并在 `Raspberry Pi` 上通过 bash `filename.sh` 运行。
:::

## 发布应用

* 要在 `Raspberry Pi` 上运行 `Avalonia` 应用，你需要使用以下 NuGet 包：

[SkiaSharp.NativeAssets.Linux](https://www.nuget.org/packages/SkiaSharp.NativeAssets.Linux/)

它包含 `libSkiaSharp.so`。

* 现在使用以下命令发布应用：

```bash
dotnet publish -r linux-arm -f netcoreapp2.0
```

* 将 publish 目录复制到 `Raspberry Pi`，然后使用 `dotnet publish/ApplicationName.dll` 运行。

## 在 Raspbian Lite 上运行

本教程将展示如何通过 [DRM](https://en.wikipedia.org/wiki/Direct\_Rendering\_Manager) 在安装了 Raspbian Lite 的 Raspberry Pi 上运行 Avalonia 应用。

### 第 1 步：配置 Raspberry Pi

第一步是先把 Raspberry Pi 配置好。

#### 下载 Raspbian Lite 操作系统镜像

你可以从 Raspberry Pi 官方网站下载 Raspbian Lite 操作系统镜像。\
[Raspberry Pi 操作系统镜像下载页](https://www.raspberrypi.com/software/operating-systems/)

#### 为刷机准备 Raspberry Pi

不同型号安装 Raspberry Lite 的方式会稍有不同。

[**Raspberry Pi 4 b**](https://www.raspberrypi.com/products/raspberry-pi-4-model-b/)\
对于 Pi 4 b，你需要一张用于安装操作系统的 SD 卡。\
将 SD 卡插入电脑。\
然后你可以直接跳到步骤 1.2。

[**Raspberry CM4**](https://www.raspberrypi.com/products/compute-module-4/?variant=raspberry-pi-cm4001000)\
由于 CM4 面向嵌入式场景设计，你还需要一块 IO 板。可以使用官方的 [Compute Module 4 IO board](https://www.raspberrypi.com/products/compute-module-4-io-board/)，也可以使用其他板卡，例如 [SourceKit PiTray mini](https://sourcekit.cc/#/?id=sourcekit%C2%AE-pitray-mini)。

要为挂载准备 EMMC 存储，请按照这些[步骤](https://www.raspberrypi.com/documentation/computers/compute-module.html#flashing-the-compute-module-emmc)操作。

#### 刷写操作系统

* [下载](https://etcher.io/) Etcher 镜像写入工具并安装。
* 打开 Etcher，选择你在步骤 1.1 中下载的 `.zip` 文件。
* 选择要写入镜像的存储设备（SD 卡或 CM4 的 EMMC）。
* 检查你的选择，然后点击 “Flash!” 开始写入。刷写完成后，请在 Raspberry 的 boot 分区中创建一个名为 **ssh** 的空文件（无扩展名，例如使用 `touch ssh`）。这样 Raspberry Pi 启动后就会启用 SSH 守护进程，你也就可以通过网络登录。
* _**仅限 CM4**：将以下内容加入 `/boot/config.txt`，以启用 USB 2.0 端口_

```conf
dtoverlay=dwc2,dr_mode=host
```

* 启动 Raspberry 并登录。\
  **Raspberry Pi 4 b**：将 SD 卡插入 Raspberry，然后接通电源。\
  **CM 4**：在 CM4 IO Board 上先拔掉电源，移除 J2 跳线，再重新接通电源。

#### 安装缺失的库

以下是通过 DRM 在 Raspbian Lite 上运行 Avalonia 应用所需的一些库：

```bash
sudo apt update
sudo apt upgrade
sudo reboot
sudo apt-get install libgbm1 libgl1-mesa-dri libegl1-mesa libinput10
```

#### 验证 DRM（可选）

你可以使用一个简单但很有用的工具 [kmscube](https://gitlab.freedesktop.org/mesa/kmscube) 来验证安装结果。

```bash
sudo apt-get install kmscube
sudo kmscube
```

现在你应该能在 Raspberry Pi 屏幕上看到旋转的立方体：\
<Image light={RaspbianLiteDrmKmsCubeScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

### 第 2 步：准备 Avalonia 应用

#### 创建新的 Avalonia 应用（Core 或 MVVM App）
在本教程中，我们将它命名为 _AvaloniaRaspbianLiteDrm_。

#### 添加包 [Avalonia.LinuxFrameBuffer](https://www.nuget.org/packages/Avalonia.LinuxFramebuffer)

```bash
dotnet add package Avalonia.LinuxFramebuffer
```

#### 2.3 创建 MainView
通过 FrameBuffer 运行时并没有窗口概念，因此你需要一个单独的视图（UserControl）作为顶层控件。它相当于普通桌面程序中的窗口。

`MainView` 将作为我们开发 UI 的基础界面：

```xml
<UserControl xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
             xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
             mc:Ignorable="d"
             d:DesignWidth="800"
             d:DesignHeight="450"
             x:Class="AvaloniaRaspbianLiteDrm.MainView">
    <StackPanel HorizontalAlignment="Center"
                VerticalAlignment="Center"
                Margin="30"
                Spacing="30">
        <TextBlock FontSize="25">
            Welcome to Avalonia! The best XAML framework ever ♥
        </TextBlock>
        <Slider />
    </StackPanel>
</UserControl>
```

接着创建一个名为 `MainSingleView` 的新 UserControl，并在其中承载 `MainView`：

```xml
<UserControl xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
             xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
             xmlns:avaloniaRaspbianLiteDrm="clr-namespace:AvaloniaRaspbianLiteDrm"
             mc:Ignorable="d"
             d:DesignWidth="800"
             d:DesignHeight="450"
             x:Class="AvaloniaRaspbianLiteDrm.MainSingleView">
    <avaloniaRaspbianLiteDrm:MainView />
</UserControl>
```

同时修改 `MainWindow.axaml`，让它也承载 `MainView`：

```xml
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:avaloniaRaspbianLiteDrm="clr-namespace:AvaloniaRaspbianLiteDrm"
        mc:Ignorable="d"
        d:DesignWidth="800"
        d:DesignHeight="450"
        x:Class="AvaloniaRaspbianLiteDrm.MainWindow"
        Title="AvaloniaRaspbianLiteDrm">
    <avaloniaRaspbianLiteDrm:MainView />
</Window>
```

`MainView` 同时被放在 `MainSingleView` 和 `MainWindow` 中。这样在开发阶段就能更方便地同时支持桌面运行和 Raspberry Pi 运行。

#### 准备 Program.cs
接下来，修改 `Program.cs` 以启用 DRM。\
请把 Main 方法改成下面这样：

```csharp
public static int Main(string[] args)
{
    var builder = BuildAvaloniaApp();
    if (args.Contains("--drm"))
    {
        SilenceConsole();
        // 默认情况下，Avalonia 会自动检测输出显卡。
        // 但你也可以手动指定，例如 "/dev/dri/card1"。
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

`SilenceConsole()` 会接管控制台输入并隐藏它。否则控制台光标会在屏幕上闪烁。

**2.4 准备 App.axaml.cs**\
接下来，为 DRM 场景下的 `ISingleViewApplicationLifetime` 设置 `MainView`。

请修改 `App.axaml.cs` 中的 `OnFrameworkInitializationCompleted()`：

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

#### 在桌面上运行和测试
现在你就可以像平常一样在桌面上运行/调试应用。\
启动应用后，你应该会看到类似这样的界面：\
<Image light={RaspbianLiteDrmDesktopScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

### 第 3 步：部署并在 Raspberry 上运行

#### 发布应用

```bash
dotnet publish -c Release -o publish -r linux-arm -p:PublishReadyToRun=true -p:PublishSingleFile=true -p:PublishTrimmed=true --self-contained true -p:IncludeNativeLibrariesForSelfExtract=true
```

#### 将应用复制到 Raspberry
把项目 `/publish` 目录中的文件复制到 Raspberry。\
你可以使用 `scp <source> <destination>`，也可以使用 [CyberDuck](https://cyberduck.io) 这类工具，或者通过 U 盘完成复制。

#### 在 Raspberry 上运行应用
首先需要把文件权限改成可执行。

```bash
sudo chmod +x /path/to/app/AvaloniaRaspbianLiteDrm
```

现在你可以使用以下命令运行应用：

```bash
sudo ./path/to/app/AvaloniaRaspbianLiteDrm --drm
```

现在你应该能在 Raspberry Pi 上看到应用已经运行起来：

<Image light={RaspbianLiteRaspberryScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

如果你安装了触摸屏，可以试着拖动一下滑块控件。

## 另请参阅

- [嵌入式 Linux 概览](/docs/platform-specific-guides/embedded-linux)
- [虚拟键盘](/docs/platform-specific-guides/embedded-linux/virtual-keyboard)
