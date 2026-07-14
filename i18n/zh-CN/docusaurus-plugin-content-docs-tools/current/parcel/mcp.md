---
id: mcp
title: Parcel MCP
sidebar_label: Parcel MCP
doc-type: how-to
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

## 什么是 Parcel MCP？

Parcel MCP 服务器让 AI 助手能够直接与 Parcel 的打包工具交互。你的助手可以从 .NET 项目创建打包配置，设置代码签名与公证，并为 Windows、macOS 和 Linux 构建安装程序，而这一切都可以通过自然语言对话完成。

有关 MCP 的一般介绍，请参阅 [AI Tools](/tools/ai-tools/)。

## 前提条件

在配置 MCP 服务器之前，请确保你具备以下条件：

1. 已安装 **Parcel .NET 工具**。请按照 [安装指南](/tools/parcel/setup) 操作。
2. 拥有**有效的 Avalonia Plus 许可证密钥**。你可以从 [Avalonia 门户](https://portal.avaloniaui.net/) 获取。

### 设置许可证密钥

MCP 服务器会从 `PARCEL_LICENSE_KEY` 环境变量中读取你的许可证。你可以在 [Avalonia Customer Portal](https://portal.avaloniaui.net/) 中找到许可证密钥。MCP 是付费功能，不包含在 Community 版本中。

请将该密钥写入 shell 配置文件，以便在不同会话之间持久生效：

<Tabs>
<TabItem value="macos-linux" label="macOS / Linux">

将这一行添加到你的 shell 配置文件中（`~/.zshrc`、`~/.bashrc` 或等效文件）：

```bash
export PARCEL_LICENSE_KEY="your-license-key"
```

然后重新加载配置文件，或打开新的终端：

```bash
source ~/.zshrc
```

</TabItem>
<TabItem value="windows-powershell" label="Windows (PowerShell)">

为你的用户帐户设置持久化环境变量：

```powershell
[System.Environment]::SetEnvironmentVariable('PARCEL_LICENSE_KEY', 'your-license-key', 'User')
```

重启所有已打开的终端和编辑器，以使更改生效。

</TabItem>
<TabItem value="windows-cmd" label="Windows (Command Prompt)">

```cmd
setx PARCEL_LICENSE_KEY "your-license-key"
```

重启所有已打开的终端和编辑器，以使更改生效。

</TabItem>
</Tabs>

:::caution[从图形界面快捷方式启动的编辑器]
如果你是通过桌面快捷方式或应用菜单启动编辑器，而不是从终端启动，它可能不会继承 shell 配置文件中的环境变量。如果 MCP 服务器报告缺少许可证密钥，你可以通过在 MCP 配置中添加 `env` 块来直接设置：

```json
{
    "env": {
        "PARCEL_LICENSE_KEY": "your-license-key"
    }
}
```

关于应将该配置块放在哪个位置，请参阅下方针对不同编辑器的说明。
:::

:::note
Parcel MCP 仅在完整的 [Avalonia Plus](https://avaloniaui.net/pricing) 许可证下可用。
:::
## 配置 MCP 服务器
## Setting up the MCP server
Parcel 提供了一个以本地进程形式运行的 MCP 服务器。其底层命令是 `parcel mcp`，但你不需要手动运行它；完成配置后，编辑器会自动启动它。
Parcel provides an MCP server that runs as a local process. The underlying command is `parcel mcp`, but you do not need to run it manually. Your editor starts it automatically once configured.
请在下方选择你的编辑器：

<Tabs groupId="editor">
<TabItem value="vscode" label="VS Code">

**方案 A：命令面板**

1. 打开命令面板（`Ctrl+Shift+P` / `Cmd+Shift+P`）。
2. 运行 **MCP: Add Server**。
3. 选择 **stdio** 作为服务器类型。
4. 输入 `parcel mcp` 作为命令。
5. 将服务器名称设为 `parcel`。
6. 选择仅为当前工作区安装，或全局安装该服务器。

**方案 B：手动配置**

将以下内容添加到工作区根目录下的 `.vscode/mcp.json`：

```json title=".vscode/mcp.json"
{
    "servers": {
        "parcel": {
            "type": "stdio",
            "command": "parcel",
            "args": ["mcp"]
        }
    }
}
```

</TabItem>
<TabItem value="visual-studio" label="Visual Studio">

Visual Studio 2022（17.x 及更高版本）支持通过 `mcp.json` 配置文件使用 MCP 服务器。

将以下内容添加到解决方案目录中的 `.vscode/mcp.json`：

```json title=".vscode/mcp.json"
{
    "servers": {
        "parcel": {
            "type": "stdio",
            "command": "parcel",
            "args": ["mcp"]
        }
    }
}
```

:::tip
Visual Studio 与 VS Code 使用相同的 `.vscode/mcp.json` 路径。如果你已经为 VS Code 完成配置，它会自动在 Visual Studio 中生效。
:::

</TabItem>
<TabItem value="rider" label="Rider">

JetBrains Rider 通过 AI Assistant 插件和 GitHub Copilot 插件支持 MCP 服务器。

**方案 A：设置界面**

1. Open **Settings** > **Tools** > **AI Assistant** > **MCP Servers**.
2. Click **Add** and select **stdio** as the transport type.
3. Set the command to `parcel` with argument `mcp`.
4. Set the server name to `parcel`.

**方案 B：手动配置**

在项目目录中创建或编辑 `.idea/mcp.json`：

```json title=".idea/mcp.json"
{
    "servers": {
        "parcel": {
            "type": "stdio",
            "command": "parcel",
            "args": ["mcp"]
        }
    }
}
```

</TabItem>
<TabItem value="cursor" label="Cursor">

将以下内容添加到项目目录中的 `.cursor/mcp.json`，或添加到 `~/.cursor/mcp.json` 以进行全局配置：

```json title=".cursor/mcp.json"
{
    "mcpServers": {
        "parcel": {
            "command": "parcel",
            "args": ["mcp"]
        }
    }
}
```

</TabItem>
<TabItem value="claude-code" label="Claude Code">

请在终端中运行以下命令：

```bash
claude mcp add --scope user parcel -- parcel mcp
```

验证是否已添加：

```bash
claude mcp list
```

</TabItem>
<TabItem value="claude-desktop" label="Claude Desktop">

1. 打开 **Settings** > **Developer**，然后点击 **Edit Config**。
2. 将 Parcel MCP 服务器添加到 `claude_desktop_config.json` 中：

```json
{
    "mcpServers": {
        "parcel": {
            "command": "parcel",
            "args": ["mcp"],
            "env": {
                "PARCEL_LICENSE_KEY": "your-license-key"
            }
        }
    }
}
```

3. 保存文件并重启 Claude Desktop。

:::note
Claude Desktop 不会继承 shell 配置文件中的环境变量，因此必须像上面展示的那样，直接在配置中设置许可证密钥。
:::
</TabItem>
</Tabs>

## 验证连接
完成 MCP 服务器配置后，请验证其是否正常工作：

1. **检查服务器是否正在运行。** 打开编辑器中的 MCP 面板或状态指示器，确认 `parcel` 已显示为已连接服务器。在 VS Code 中，可通过命令面板运行 **MCP: List Servers**。
2. **使用提示词进行测试。** 向你的 AI 助手提问：

```text
"List the available Parcel packaging tools."
```

如果助手返回了一组能力列表，说明配置已完成。

## 故障排查

### 找不到 “parcel” 命令

`parcel` 命令必须存在于系统 PATH 中。如果你将它作为全局 .NET 工具安装，请确保 `~/.dotnet/tools`（macOS/Linux）或 `%USERPROFILE%\.dotnet\tools`（Windows）已加入 PATH。

你可以通过运行以下命令来确认该工具是否已安装：

```bash
dotnet tool list -g
```

### 未检测到许可证密钥

如果 MCP 服务器已启动，但报告许可证密钥缺失或无效：

- **确认变量已设置。** 在启动编辑器所用的同一个终端中运行 `echo $PARCEL_LICENSE_KEY`（macOS/Linux）或 `echo %PARCEL_LICENSE_KEY%`（Windows）。
- **如果你的编辑器是从图形界面快捷方式启动的**，它可能不会继承 shell 环境变量。请按上文 [设置许可证密钥](#setting-your-license-key) 一节所示，在 MCP 配置中添加 `env` 块。

### 编辑器中未显示 MCP 服务器

- **重启编辑器。** 添加或修改 MCP 配置文件后，大多数编辑器都需要重启才能检测到新的 MCP 服务器。
- **检查配置文件路径。** 每个编辑器都要求配置文件位于特定路径。请参阅上文对应编辑器的配置说明。
- **校验 JSON。** 配置文件中的语法错误（缺少逗号、多余尾逗号、括号不匹配）会导致服务器静默加载失败。

### 更新 Parcel

如果工具行为异常，请确认你运行的是最新版本：

```bash
dotnet tool update -g parcel
```

## 能力

完成 MCP 服务器配置后，你的 AI 助手可以帮助你完成以下工作：

### 项目配置

- **创建 Parcel 配置**，基于现有 .NET 项目生成打包配置
- **配置应用属性**，例如包名、显示名称、图标和 bundle 标识符
- **设置构建目标**，覆盖多个平台和架构

### 代码签名配置

- **Windows Azure Trusted Signing** - 配置证书和签名参数
- **macOS Code Signing** - 设置 P12 证书和配置描述文件
- **macOS Notarization** - 配置 Apple ID 与应用专用密码

### 构建与打包

- **构建并打包** 多个平台（Windows、macOS、Linux）的应用程序
- **生成多种格式的安装包**（DMG、DEB、NSIS、ZIP 等）
- **执行跨平台打包**，并生成特定运行时输出

## 使用示例

请用自然语言描述你想完成的事情，AI 助手会自动调用 MCP 工具：

**项目设置：**

```text
"Create a packaging config for my Avalonia project and set up macOS signing."
```

**打包：**

```text
"Package my app for macOS as a DMG with code signing enabled."
```

**配置管理：**

```text
"Update my app's display name and icon, then rebuild the Windows installer."
```

<video controls width="90%">
  <source src="/video/parcel/parcel_mcp.mp4" />
</video>

## 另请参阅

- [AI Tools 概览](/tools/ai-tools/)
- [Parcel 安装](/tools/parcel/setup)
- [DevTools MCP](/tools/developer-tools/mcp)
