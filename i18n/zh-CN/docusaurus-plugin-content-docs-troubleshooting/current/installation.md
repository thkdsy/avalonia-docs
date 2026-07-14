---
id: installation
title: 安装故障排除
description: 关于安装 .NET SDK、Avalonia 模板和 NuGet 包源时常见问题的解决方案。
doc-type: troubleshooting
---

本页涵盖了在安装 .NET SDK 或 Avalonia 项目模板时可能遇到的最常见问题，以及逐步解决方案。

## .NET 不是可识别的程序

如果你的终端报告 `dotnet` 不是可识别的命令，则说明 .NET SDK 要么尚未安装，要么没有添加到系统 `PATH` 中。

### 第 1 步：检查 SDK 是否已安装

运行以下命令：

```bash
dotnet --list-sdks
```

如果 .NET SDK 已正确安装，则会返回类似以下内容：

```text
8.0.202 [C:\Program Files\dotnet\sdk]
```

如果你看到的是错误信息，请从 [.NET 官方网站](https://dotnet.microsoft.com/en-us/download/dotnet) 下载并安装 .NET SDK。

### 第 2 步：重启终端

安装 SDK 后，关闭并重新打开终端（或启动新的 shell 会话）。安装程序会更新系统 `PATH`，但现有的终端会话不会自动获取该更改。

### 第 3 步：验证 PATH（如果问题仍然存在）

如果重启终端后 `dotnet` 仍然不被识别，请确认 SDK 安装目录已包含在你的 `PATH` 环境变量中。

典型安装位置：

| 操作系统 | 默认路径                              |
|---------|---------------------------------------|
| Windows | `C:\Program Files\dotnet`             |
| macOS   | `/usr/local/share/dotnet`             |
| Linux   | `/usr/share/dotnet` or `$HOME/.dotnet`|

在 **Windows** 上，打开 **系统属性 > 环境变量**，检查上述路径是否出现在 `Path` 变量中。在 **macOS** 和 **Linux** 上，检查你的 shell 配置文件（例如 `~/.bashrc`、`~/.zshrc` 或 `~/.bash_profile`）中是否有导出 dotnet 目录的行。

:::tip
在 macOS 上，如果你通过官方安装程序安装了 .NET，但命令仍然找不到，请尝试运行：

```bash
export PATH="$PATH:/usr/local/share/dotnet"
```

将这行添加到你的 shell 配置文件中即可永久生效。
:::

### 第 4 步：检查是否存在多个 SDK 安装

如果你安装了多个 .NET SDK 版本或使用了多种安装方式（例如 macOS 上的 Homebrew 和官方安装程序），它们可能会冲突。运行：

```bash
which dotnet
```

确保返回的路径指向你期望的安装位置。如果不是，请调整你的 `PATH`，使正确的安装优先。

## 无法找到 `Avalonia.Templates` 包

如果 `dotnet new install Avalonia.Templates` 因“not found”错误而失败，那么你的 NuGet 包源配置中可能缺少公共 NuGet 源。

### 第 1 步：列出你的 NuGet 源

运行此命令：

```bash
dotnet nuget list source
```

检查输出中是否包含以下条目：

```text
nuget.org [Enabled]
https://api.nuget.org/v3/index.json
```

### 第 2 步：如果缺失则添加 NuGet 源

如果列表中没有出现 `nuget.org`，请添加它：

```bash
dotnet nuget add source https://api.nuget.org/v3/index.json -n nuget.org
```

然后重新尝试安装模板：

```bash
dotnet new install Avalonia.Templates
```

### 第 3 步：重新启用已禁用的源

如果 `nuget.org` 出现在列表中，但显示为 `[Disabled]`，请启用它：

```bash
dotnet nuget enable source nuget.org
```

### 第 4 步：检查网络和防火墙设置

如果源已列出并启用，但安装仍然失败，请检查以下内容：

- **公司代理或 VPN**：你的网络可能阻止访问 `api.nuget.org`。请联系你的网络管理员，或尝试从其他网络连接。
- **防火墙规则**：确保允许到 `api.nuget.org` 的出站 HTTPS 流量（端口 443）。
- **DNS 解析**：运行 `ping api.nuget.org` 验证域名是否正确解析。

### 第 5 步：清除 NuGet 缓存

损坏或过期的缓存数据有时会导致安装失败。清除缓存后再试一次：

```bash
dotnet nuget locals all --clear
dotnet new install Avalonia.Templates
```

## 模板安装成功但模板未出现

如果 `dotnet new install Avalonia.Templates` 显示成功，但 `dotnet new list` 没有显示任何 Avalonia 模板，你可能遇到了模板引擎缓存问题。

尝试重新安装：

```bash
dotnet new uninstall Avalonia.Templates
dotnet new install Avalonia.Templates
```

如果你使用的是带有 Avalonia 扩展的 Visual Studio，请注意该扩展会捆绑其自己的模板副本。`dotnet new install` 这一步仅对命令行或非 Visual Studio 的 IDE 工作流是必需的。

## 安装期间的权限错误

在 macOS 和 Linux 上，安装模板或 .NET SDK 时可能会看到权限错误。

- 避免在 `dotnet new install` 中使用 `sudo`。.NET 模板引擎会将模板存储在你的用户配置目录中，不需要提升权限。
- 如果你之前使用 `sudo` 运行过安装，模板缓存可能归 root 所有。使用以下命令重置所有权：

```bash
sudo chown -R $(whoami) ~/.templateengine
```

## 另请参阅

- [安装 Avalonia](/docs/get-started/install-avalonia)
- [设置你的 IDE](/docs/get-started/set-up-your-ide)
- [应用性能问题](/troubleshooting/app-performance-issues)