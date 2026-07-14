---
id: command-line-reference
title: Parcel 命令行参考
sidebar_label: 命令行参考
doc-type: reference
---

Parcel 提供了一个命令行工具，用于为 Windows、macOS 和 Linux 打包 Avalonia 应用。它内置应用签名和打包能力，可简化可安装二进制分发流程。

## 前提条件

在使用 Parcel 之前，请确保你具备以下条件：

1. 已安装 **Parcel .NET 工具** —— 请参阅 [安装指南](/tools/parcel/setup)。
2. 拥有**有效许可证密钥** —— 设置 `PARCEL_LICENSE_KEY` 环境变量，或使用 `--license-key` 选项并传入从门户获取的有效许可证密钥。

:::note
Parcel CLI 仅在拥有 [Avalonia Plus](https://avaloniaui.net/pricing) 许可证时可用。
:::

## 概览

```bash
parcel [command] [options]
```

## 全局选项

| 选项 | 说明 |
|--------|-------------|
| `-?, -h, --help` | 显示帮助和用法信息 |
| `--version` | 显示版本信息 |
| `--license-key` | 运行 Parcel 所需的许可证密钥。未设置时，会回退到 `PARCEL_LICENSE_KEY` 环境变量或现有应用会话 |
| `--verbosity` | 设置日志详细级别（quiet、minimal、normal、detailed、diagnostic） |

## 命令

### pack

根据预定义设置和输入参数构建并打包项目。

```bash
parcel pack <project> [options]
```

**参数：**

- `<project>` - 用于加载配置的 Parcel 项目文件

**选项：**

| 选项 | 说明 | 默认值 |
|--------|-------------|---------|
| `-o, --output` | 输出目录 | `<project-dir>\bin\packages` |
| `-r, --runtimes` | 要打包到的运行时标识符（可多次指定） | 当前平台运行时 |
| `-p, --packages` | 输出包格式：`deb`、`dmg`、`nsis`、`zip`（可多次指定） | 当前平台包格式 |
| `--no-build` | 跳过重新编译输入项目 | false |

**示例：**

```bash
# 为当前平台打包
parcel pack MyApp.parcel.xml

# 为多个平台和格式打包
parcel pack MyApp.parcel.xml -r osx-x64 -r linux-x64 -p dmg -p deb
```

### step

运行打包流程中的某个特定步骤。适用于调试或自定义打包工作流。

```bash
parcel step [command] <input> <output> [options]
```

**可用步骤命令：**

| 命令 | 说明 | 输入 | 输出 |
|---------|-------------|-------|--------|
| `publish` | 为目标平台发布 .NET 项目 | -unused- | 已发布应用目录 |
| `merge-mac` | 将多架构合并为通用 macOS 应用 | 含架构子目录的目录（osx-x64、osx-arm64） | 通用应用目录 |
| `bundle-mac` | 将 macOS 应用打包为单个 bundle | 应用目录 | 应用 bundle（.app） |
| `sign-mac` | 使用凭据为 macOS bundle 签名 | 应用 bundle 或平铺目录 | 已签名 bundle |
| `notary-mac` | 将应用提交给 Apple 进行公证 | 已压缩的 app bundle 或 DMG | 已公证文件（DMG 情况下会附加 stapled） |
| `sign-win` | 为 Windows 可执行文件签名 | 应用可执行文件或安装器 | 已签名可执行文件或安装器 |
| `create-zip` | 创建用于分发的 zip 压缩包 | 目录或文件 | zip 压缩包（.zip） |
| `create-dmg` | 为 macOS 创建 DMG 磁盘镜像 | 应用 bundle（.app） | 未签名 DMG 镜像文件 |
| `create-deb` | 为 Linux 创建 Debian 包 | 应用目录 | Debian 包（.deb） |
| `create-nsis` | 创建 Windows NSIS 安装器 | 应用目录 | 未签名 NSIS 安装器（.exe） |

**示例：**

这些命令虽然没有强制顺序，也可以独立执行，但通常会按不同平台采用如下标准流程。

其中任意步骤都可以替换为你自己的脚本，因此它比标准的 `parcel pack` 命令具有更高灵活性。


<Tabs>
<TabItem value="win" label="Windows" default>

```bash
# 也可以改用 `parcel step publish ./publish -r win-x64 -p project.parcel`
dotnet publish -r win-x64 -o ./publish

# 签名，参数从 .parcel 配置文件中读取
parcel step sign-win ./publish ./signed -p project.parcel

# 安装器
parcel step create-nsis ./signed ./installer.exe -p project.parcel

# 或 ZIP 压缩包
parcel step create-zip ./signed ./archive.zip -p project.parcel
```

</TabItem>
<TabItem value="mac" label="macOS">

```bash
mkdir ./publish

# 若要生成通用包，需要同时发布两个架构
dotnet publish -r osx-x64 -o ./publish/osx-x64
dotnet publish -r osx-arm64 -o ./publish/osx-arm64

# 将两个架构合并为通用版本
parcel step merge-mac ./publish ./merged -p project.parcel

# 创建应用 bundle
parcel step bundle-mac ./merged ./bundle.app -p project.parcel

# 签名，参数从 .parcel 配置文件中读取
parcel step sign-mac ./bundle.app ./signed.app -p project.parcel

# 公证
parcel step notary-mac ./signed.app ./notarized.app -p project.parcel

# DMG 包
parcel step create-dmg ./notarized.app ./package.dmg -p project.parcel

# 或 ZIP 压缩包
parcel step create-zip ./notarized.app ./archive.zip -p project.parcel
```

:::note

如果你希望同时在 Intel 和 Apple Silicon 处理器上获得原生性能、避免仿真层，那么通用包就是必需的。

缺点是通用包的可执行二进制体积最多可能增大到原来的两倍。

如果你不需要这一点，可以完全跳过 `merge-mac` 步骤。

:::

</TabItem>
<TabItem value="lin" label="Linux">


```bash
# 也可以改用 `parcel step publish ./ ./publish -r linux-x64 -p project.parcel`
dotnet publish -r linux-x64 -o ./publish 

# 安装器
parcel step create-deb ./publish ./installer.deb -p project.parcel

# 或 ZIP 压缩包
parcel step create-zip ./publish ./archive.zip -p project.parcel
```

</TabItem>
</Tabs>

**通用选项：**

- `-p, --project` - 用于加载配置的 Parcel 项目文件
- `-w, --overwrite` - 覆盖现有输出文件
- `-r, --runtime` - 运行时标识符（用于 publish 命令）

### install-tools

下载或更新打包配置所需的工具依赖项。

```bash
parcel install-tools [options]
```

**选项：**

| 选项 | 说明 |
|--------|-------------|
| `-r, --runtimes` | 运行时标识符（可多次指定） |
| `-p, --packages` | 包格式：`deb`、`dmg`、`nsis`、`zip`（可多次指定） |

**示例：**

```bash
# 为指定平台和包格式安装依赖
parcel install-tools -r win-x64 -r osx-x64 -p nsis -p dmg
```

这个命令会预先下载 Parcel 运行所需的 **NSIS** 和 **DMG** 工具依赖。

### mcp

运行一个 Model Context Protocol 服务器，使 Parcel 命令可以从 LLM AI 会话中执行。

```bash
parcel mcp
```

更多用法请参阅 [Model Context Protocol](/tools/parcel/mcp)。

## 环境变量

- `PARCEL_LICENSE_KEY` - 当未提供 `--license-key` 选项时使用的默认许可证密钥

## 备注

- 所有打包选项、签名凭据和视觉自定义都定义在 Parcel 项目文件（.parcel）中
- 使用 `--no-build` 时，请确保发布相关设置（trimming、AOT、single-file）与你的构建结果和 Parcel 配置保持一致

## 另请参阅

- [Parcel 安装](/tools/parcel/setup)
- [为 macOS 打包应用](/tools/parcel/packaging-for-macos)
- [为 Windows 打包应用](/tools/parcel/packaging-for-windows)
- [为 Linux 打包应用](/tools/parcel/packaging-for-linux)
