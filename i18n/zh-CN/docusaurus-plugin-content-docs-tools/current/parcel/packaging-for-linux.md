---
id: packaging-for-linux
title: 为 Linux 打包应用
description: 使用 Parcel 为 Linux 打包 Avalonia 应用，支持 DEB、RPM、ZIP 和 AppImage 格式，并涵盖依赖管理与桌面集成。
sidebar_label: Linux
doc-type: reference
tags:
  - avalonia plus
  - avalonia pro
  - avalonia enterprise
---

Parcel 可将 Linux 应用打包为多种分发格式，以适配不同的 Linux 包管理器和使用场景。

## 支持的包格式

- **DEB**：Debian/Ubuntu 包（`.deb`）—— 可通过 `apt` 安装
- **RPM**：Red Hat/Fedora 包（`.rpm`）—— 可通过 `dnf`/`yum` 安装
- **ZIP**：用于手动安装的压缩包
- **AppImage**：便携式单文件应用 *（尚未提供）*

Parcel 会自动生成 `.desktop` 文件，以便正确集成到应用启动器中。

## 依赖项

Parcel 会在 DEB/RPM 包中包含以下依赖项，以确保跨 Linux 发行版的兼容性：

### 运行时依赖
- `libc6` - GNU C 库
- `libgcc1` 或 `libgcc-s1` - GCC 运行时库
- `libgssapi-krb5-2` - Kerberos 身份验证
- `libstdc++6` - GNU 标准 C++ 库
- `zlib1g` - 压缩库
- `libssl1.0.0 | libssl1.0.2 | libssl1.1 | libssl3` - SSL/TLS 库（支持多个版本）
- `libicu` - Unicode 和国际化支持（支持多个版本）

### Avalonia 特定依赖
- `libx11-6` - X11 客户端库
- `libice6` - Inter-Client Exchange 库
- `libsm6` - X11 会话管理库
- `libfontconfig1` - 字体配置库

## Bundle 配置

Parcel 提供配置选项，用于自定义 Linux 应用包，以实现正确的桌面集成和品牌展示。

### 通用属性

**Application Name**:

显示在应用启动器和桌面菜单中的应用名称。该名称会用于 `.desktop` 条目文件中。

**Package Name**:

作为输出文件名以及 `/usr/share/` 应用条目使用的包标识符。不能包含空格。

### DEB/RPM 特有属性

用于 Debian 和 RPM 包的附加配置属性。

**Application Icon**:

应用图标文件路径。Parcel 会自动：
- 在合适分辨率下生成 hicolor 图标主题条目
- 在 `.desktop` 文件中链接该图标

**支持格式**：PNG、SVG

**Company/Maintainer**:

包维护者或公司名称。它会出现在包元数据中，并由包管理器显示。

:::note
如果未指定，则默认使用 Package Name 的值。
:::

**Desktop Category**:

用于桌面环境菜单和启动器中的应用分类。它决定了应用在应用菜单层级中的位置。

## 安装与卸载

### DEB 包（Debian/Ubuntu）

**安装**：
```bash
sudo apt install ./my-app.deb
```

**卸载**：
```bash
sudo apt remove my-app
```

### RPM 包（Fedora/RHEL）

**安装**：
```bash
sudo dnf install ./my-app.rpm
# or
sudo rpm -i ./my-app.rpm
```

**卸载**：
```bash
sudo dnf remove my-app
# or
sudo rpm -e my-app
```

### ZIP 压缩包

**解压并运行**：
```bash
unzip my-app.zip
cd my-app
./my-awesome-app
```

## 另请参阅

- [Parcel 安装](/tools/parcel/setup)
- [Parcel 命令行参考](/tools/parcel/command-line-reference)
