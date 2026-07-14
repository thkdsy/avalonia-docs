---
id: linux
title: Linux 部署
description: 如何在 Linux 上发布和部署 XPF 应用程序，包括所需的原生库和运行时依赖项。
---

## 发布

始终从命令行发布 XPF 应用程序。Visual Studio 的发布可能会生成不完整的输出，缺少诸如 `libSkiaSharp.so` 之类的原生库。

```bash
dotnet publish -r linux-x64 -c Release
```

对于自包含部署：

```bash
dotnet publish -r linux-x64 -c Release --self-contained
```

对于 ARM64 设备：

```bash
dotnet publish -r linux-arm64 -c Release --self-contained
```

## 运行时依赖项

确保目标系统已安装以下软件包。

### Debian / Ubuntu

```bash
sudo apt install libice6 libsm6 libfontconfig1 libgdiplus
```

### Fedora

```bash
sudo dnf install libICE libSM fontconfig libgdiplus
```

### RHEL / CentOS / Rocky Linux

```bash
sudo dnf install epel-release
sudo dnf install libICE libSM fontconfig libgdiplus
```

对于 WebView 支持，还需安装 `libwebkit2gtk-4.1-dev`（Debian/Ubuntu）或 `webkit2gtk4.1-devel`（Fedora/RHEL）。

详情请参见 [Linux: 其他依赖项](/xpf/platforms/linux#other-dependencies)。

## ReadyToRun

ReadyToRun（R2R）编译会将程序集预编译为本机代码，显著减少启动时间。这在嵌入式 Linux 设备上尤其有益。

```xml
<PropertyGroup>
    <PublishReadyToRun>true</PublishReadyToRun>
</PropertyGroup>
```

```bash
dotnet publish -r linux-x64 -c Release --self-contained
```

:::note
ReadyToRun 可能会改变本机 `.so` 库的解析方式。详情请参见 [Linux: 本机库解析](/xpf/platforms/linux#native-library-resolution-with-readytorun)。
:::

## 依赖框架与自包含

**依赖框架**（默认）：要求目标机器上已安装 .NET。生成的部署包更小。

**自包含**：包含 .NET 运行时。包更大，但除系统库外没有外部依赖。建议用于分发给最终用户。

```bash
# Framework-dependent
dotnet publish -r linux-x64 -c Release

# Self-contained
dotnet publish -r linux-x64 -c Release --self-contained
```

## 打包格式

### AppImage

AppImage 会将你的应用程序打包为单个可执行文件。可以使用 [appimage-builder](https://appimage-builder.readthedocs.io/) 等工具，或将发布输出手动打包为 AppImage。

### Debian 包（.deb）

对于基于 Debian 的发行版，请创建 `.deb` 包。可使用 `dpkg-deb` 或类似 [dotnet-packaging](https://github.com/quamotion/dotnet-packaging) 的工具：

```bash
dotnet tool install --global dotnet-deb
dotnet deb -r linux-x64 -c Release
```

### RPM 包

对于基于 Fedora 和 RHEL 的发行版：

```bash
dotnet tool install --global dotnet-rpm
dotnet rpm -r linux-x64 -c Release
```

### Flatpak 和 snap

XPF 应用程序可以作为 Flatpak 或 Snap 包进行分发。请参阅各打包系统的文档，了解如何打包 .NET 应用程序。

## CI/CD

在 CI/CD 流水线中构建 XPF 应用程序时：

1. 添加包含许可证密钥的 `NuGet.config`（密钥值使用 CI secrets）
2. 在构建环境中安装所需依赖项
3. 从命令行发布

GitHub Actions 示例步骤：

```yaml
- name: Publish for Linux
  run: dotnet publish -r linux-x64 -c Release --self-contained
  env:
    XpfLicenseKey: ${{ secrets.XPF_LICENSE_KEY }}
```

有关使用环境变量与许可证密钥，请参见 [集中管理多个 XPF 项目](/xpf/configuration/centralizing-multiple-xpf-projects#license-keys)。

## 调试远程 Linux 目标

要从 Windows 开发机器调试在 Linux 上运行的 XPF 应用程序，请参见 [Linux: 调试](/xpf/platforms/linux#debugging-on-linux)。