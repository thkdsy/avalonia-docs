---
id: embedded-linux
title: 嵌入式 Linux
description: 使用 DRM/KMS 在嵌入式 Linux 设备上发布、传输并运行 Avalonia 应用。
doc-type: how-to
---

将 Avalonia 应用部署到嵌入式 Linux 设备，与桌面部署有多方面不同。目标设备通常没有包管理器（大多数情况下）、没有可集成的桌面环境，而且应用往往作为唯一的图形进程运行。本页将介绍如何在嵌入式 Linux 目标设备上发布、传输并运行你的应用。

## 发布

请将应用发布为面向合适运行时标识符（RID）的自包含单文件可执行程序。对于嵌入式目标，强烈建议采用自包含部署，因为设备上通常不会预装 .NET 运行时。

请选择与你目标硬件匹配的 RID：

| 目标架构 | RID | 常见设备 |
|---|---|---|
| ARM 32 位 | `linux-arm` | Raspberry Pi（32 位系统）、较老的 ARM 单板机 |
| ARM 64 位 | `linux-arm64` | Raspberry Pi 4/5（64 位系统）、NVIDIA Jetson、BeagleBone AI、大多数现代 ARM 单板机 |
| x64 | `linux-x64` | Intel NUC、工业平板电脑、AMD 嵌入式主板 |

```bash
dotnet publish -c Release -r linux-arm64 --self-contained true \
  -p:PublishSingleFile=true \
  -p:PublishTrimmed=true \
  -p:PublishReadyToRun=true \
  -p:IncludeNativeLibrariesForSelfExtract=true
```

请将 `linux-arm64` 替换为适合你设备的 RID。

### 发布选项说明

| 选项 | 作用 |
|---|---|
| `--self-contained true` | 打包 .NET 运行时，因此目标设备无需单独安装。 |
| `-p:PublishSingleFile=true` | 生成单个可执行文件，而不是一整个程序集目录。 |
| `-p:PublishTrimmed=true` | 移除未使用代码，显著减小输出体积。 |
| `-p:PublishReadyToRun=true` | 预编译程序集为本机代码，以加快启动速度。 |
| `-p:IncludeNativeLibrariesForSelfExtract=true` | 将原生库（SkiaSharp、HarfBuzz）嵌入单文件中。 |

:::tip
裁剪可能会移除应用通过反射使用的代码。如果你在运行时遇到 `MissingMethodException` 或类似错误，请配置[裁剪器根程序集](https://learn.microsoft.com/en-us/dotnet/core/deploying/trimming/trimming-options)以保留受影响的类型。
:::

## 传输到设备

将发布输出复制到目标设备。常见方式如下：

**SCP（通过 SSH）：**
```bash
scp -r ./publish/ user@device-hostname:/home/user/myapp/
```

**rsync（增量同步，适合重复部署）：**
```bash
rsync -avz --progress ./publish/ user@device-hostname:/home/user/myapp/
```

**USB 驱动器：**
将发布目录复制到 U 盘，在目标设备上挂载后再复制文件。

## 运行应用

为可执行文件设置权限，并带上 `--drm` 参数运行：

```bash
chmod +x /home/user/myapp/MyApp
sudo ./home/user/myapp/MyApp --drm
```

`--drm` 参数会告诉应用使用 DRM/KMS 输出，而不是尝试连接到 X11 或 Wayland。通常需要使用 `sudo` 运行，因为访问 DRM 设备一般需要 root 权限。

:::tip
如果你希望避免以 root 身份运行，可以将当前用户加入 `video` 和 `input` 组：
```bash
sudo usermod -aG video,input $USER
```
退出并重新登录后，组变更才会生效。之后你就可以不使用 `sudo` 运行应用。
:::

## 开机自动启动

对于信息亭或专用设备场景，可以将应用配置为设备启动时自动运行。

### 使用 systemd 服务

在 `/etc/systemd/system/myapp.service` 创建一个服务文件：

```ini
[Unit]
Description=My Avalonia Application
After=multi-user.target

[Service]
Type=simple
User=appuser
Group=appuser
SupplementaryGroups=video input
ExecStart=/opt/myapp/MyApp --drm
Restart=on-failure
RestartSec=5
Environment=DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=1

[Install]
WantedBy=multi-user.target
```

请将 `appuser` 替换为设备上的实际用户账号，并根据你的部署位置调整 `ExecStart` 路径。`/opt/myapp/` 是嵌入式系统中常见的应用二进制目录约定，但你也可以使用任意其他路径。

启用并启动该服务：

```bash
sudo systemctl enable myapp.service
sudo systemctl start myapp.service
```

### 查看日志

```bash
journalctl -u myapp.service -f
```

## 缩减镜像体积

嵌入式系统通常存储空间有限。以下策略有助于减小已部署应用的体积：

- **裁剪**（上文已启用）会移除未使用的 .NET 程序集和方法。
- **Native AOT** 编译会生成更小的纯原生二进制文件，不再承担 .NET 运行时开销。详情请参阅 [Native AOT 部署](/docs/deployment/native-aot)。
- **固定区域性模式**（`DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=1`）可移除对 ICU 库的依赖，大约节省 30 MB。只有当你的应用不需要与区域设置相关的格式化或排序时才建议使用。

## 目标设备所需库

即使采用自包含部署，目标设备上仍必须存在某些原生 Linux 库。在基于 Debian 的系统（Raspberry Pi OS、Armbian、Ubuntu）上：

```bash
sudo apt-get install libgbm1 libgl1-mesa-dri libegl1-mesa libinput10
```

其他发行版中也会有等效的软件包，只是名称可能略有不同。

| 软件包 | 提供内容 | Avalonia 为什么需要它 |
|---|---|---|
| `libgbm1` | Generic Buffer Management（GBM）分配器，用于创建可被 GPU 访问、并可由 DRM 输出到显示器的缓冲区。 | Avalonia 在 DRM 模式下通过 GBM 分配渲染表面。没有它就无法创建帧缓冲。 |
| `libgl1-mesa-dri` | Mesa 的 DRI（Direct Rendering Infrastructure）驱动。这些是面向具体 GPU 的模块（例如 Raspberry Pi 的 `vc4`、Mali GPU 的 `panfrost`），用于将 OpenGL 调用转换为硬件命令。 | 提供真正的 GPU 加速能力。即使设备没有独立 GPU，这里也包含软件光栅器（`llvmpipe`）。 |
| `libegl1-mesa` | Mesa 对 EGL 的实现（EGL 最初意为 “Embedded-System Graphics Library”，现在是由 Khronos 维护的独立名称）。EGL 是位于渲染 API（如 OpenGL ES）与原生显示系统之间的平台无关 API，负责创建渲染上下文、将其绑定到绘图表面，以及管理缓冲区和同步对象等资源。在桌面 Linux + X11 上，EGL 会与 X 服务器通信；而在嵌入式 DRM 环境中，EGL 则直接与 GBM 表面交互。 | Avalonia 使用 EGL 创建 OpenGL ES 渲染上下文，并将其绑定到由 DRM 帧缓冲支持的 GBM 表面上。这一步把 Avalonia 的绘图命令真正连接到了显示设备上的像素输出。 |
| `libinput10` | libinput 库。它通过内核的 evdev 接口，为键盘、鼠标、触控板和触摸屏提供统一的输入读取 API。 | Avalonia 在脱离桌面环境运行时，会通过 libinput 读取所有用户输入。没有它，触摸、鼠标和键盘输入都无法工作。 |

## 另请参阅

- [嵌入式 Linux 平台集成](/docs/platform-specific-guides/embedded-linux)：了解帧缓冲与 DRM 概念。
- [在 Raspberry Pi 上运行](/docs/platform-specific-guides/embedded-linux/raspberry-pi)：面向具体硬件的实战说明。
- [桌面 Linux 部署](/docs/deployment/linux)：了解 `.deb` 打包。
- [Native AOT 部署](/docs/deployment/native-aot)
