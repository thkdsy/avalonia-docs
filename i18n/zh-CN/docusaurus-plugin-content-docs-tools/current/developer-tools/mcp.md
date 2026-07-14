---
id: mcp
title: DevTools MCP
sidebar_label: DevTools MCP
doc-type: how-to
description: "配置 DevTools MCP 服务器，让 AI 助手可以检查、调试并修改正在运行的 Avalonia 应用程序。"
keywords:
  - mcp
  - model context protocol
  - ai assistant
  - devtools
  - copilot
  - claude
  - cursor
  - ai tools
tags:
  - mcp
  - ai
  - avalonia plus
  - avalonia pro
  - avalonia enterprise
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

## 什么是 DevTools MCP？

DevTools MCP 服务器让 AI 助手可以连接到正在运行的 Avalonia 应用，并直接与之交互。你的助手可以检查可视树，按类型或名称搜索元素，读取和修改属性，截取屏幕截图，并发送输入事件。它还可以附加到 XAML 预览器，因此在不离开编辑器的情况下迭代布局时非常有用。

有关 MCP 的一般介绍，请参阅 [AI Tools](/tools/ai-tools/)。

## 前提条件

在配置 MCP 服务器之前，请确保你具备以下条件：

1. 已安装 **DevTools .NET 工具**。请按照 [入门指南](/tools/developer-tools/installation) 操作。
2. 拥有**有效的 Avalonia Plus 许可证密钥**。你可以从 [Avalonia 门户](https://portal.avaloniaui.net/) 获取。

### 设置许可证密钥

MCP 服务器会从 `AVALONIA_TOOLS_LICENSE_KEY` 环境变量中读取你的许可证。你可以在 [Avalonia 客户门户](https://portal.avaloniaui.net/) 中找到许可证密钥。MCP 是付费功能，不包含在 Community 许可证中。

:::note
从 Avalonia 12.0.0 开始使用 `AVALONIA_TOOLS_LICENSE_KEY` 变量。如果你使用的是 Avalonia 11.x.x 或更早版本，请改用 `ACCELERATE_LICENSE_KEY`。
:::

请将该密钥写入 shell 配置文件，以便在不同会话之间持久生效：

<Tabs>
<TabItem value="macos-linux" label="macOS / Linux">

将这一行添加到你的 shell 配置文件中（`~/.zshrc`、`~/.bashrc` 或等效文件）：

```bash
export AVALONIA_TOOLS_LICENSE_KEY="your-license-key"
```

然后重新加载配置文件，或打开新的终端：

```bash
source ~/.zshrc
```

</TabItem>
<TabItem value="windows-powershell" label="Windows (PowerShell)">

为你的用户帐户设置持久化环境变量：

```powershell
[System.Environment]::SetEnvironmentVariable('AVALONIA_TOOLS_LICENSE_KEY', 'your-license-key', 'User')
```

重启所有已打开的终端和编辑器，以使更改生效。

</TabItem>
<TabItem value="windows-cmd" label="Windows (Command Prompt)">

```cmd
setx AVALONIA_TOOLS_LICENSE_KEY "your-license-key"
```

重启所有已打开的终端和编辑器，以使更改生效。

</TabItem>
</Tabs>

:::caution[从图形界面快捷方式启动的编辑器]
如果你是通过桌面快捷方式或应用菜单启动编辑器，而不是从终端启动，它可能不会继承 shell 配置文件中的环境变量。如果 MCP 服务器报告缺少许可证密钥，你可以通过在 MCP 配置中添加 `env` 块来直接设置：

```json
{
    "env": {
        "AVALONIA_TOOLS_LICENSE_KEY": "your-license-key"
    }
}
```

关于应将该配置块放在哪个位置，请参阅下方针对不同编辑器的说明。
:::

:::note
DevTools MCP 仅在 Avalonia Plus 或更高许可证下可用。
:::
## 准备你的应用程序
## Prepare your application
MCP 服务器通过 `AvaloniaUI.DiagnosticsSupport` 包与你的 Avalonia 应用通信。没有这个包以及必需的启动调用，MCP 服务器将无法发现或附加到你正在运行的应用。

:::caution[MCP 连接所必需]
这一步是 `attach-to-app` 正常工作的必要条件。如果跳过它，MCP 服务器会反复连接失败，而且没有明确错误信息。`attach-to-file` 工具（XAML 预览器）不要求应用正在运行，但仍要求安装该包。
:::

**1. 将 diagnostics support 包添加到项目中：**

```bash
dotnet add package AvaloniaUI.DiagnosticsSupport
```

**2. 在应用启动时启用 developer tools。**

请选择以下任一方式：

<Tabs>
<TabItem value="appbuilder" label="App builder (Program.cs)">

```csharp
public static AppBuilder BuildAvaloniaApp()
    => AppBuilder.Configure<App>()
        .UsePlatformDetect()
        .WithDeveloperTools();
```

</TabItem>
<TabItem value="application" label="Application class (App.axaml.cs)">

```csharp
public override void Initialize()
{
    AvaloniaXamlLoader.Load(this);

#if DEBUG
    this.AttachDeveloperTools();
#endif
}
```

</TabItem>
</Tabs>

有关完整安装流程，包括平台特定要求和激活步骤，请参阅 [Installing the Avalonia Plus developer tools](/tools/developer-tools/installation)。

## 配置 MCP 服务器

DevTools 提供了一个以本地进程形式运行的 MCP 服务器。其底层命令是 `avdt mcp`，但你不需要手动运行它；完成配置后，编辑器会自动启动它。

:::note
从 Avalonia 12.0.0 开始使用 `AVALONIA_TOOLS_LICENSE_KEY` 变量。如果你使用的是 Avalonia 11.x.x 或更早版本，请改用 `ACCELERATE_LICENSE_KEY`。
:::

请在下方选择你的编辑器：

<Tabs groupId="editor">
<TabItem value="vscode" label="VS Code">

**方案 A：一键安装**

[Install DevTools MCP for VS Code](https://vscode.dev/redirect/mcp/install?name=avalonia_devtools&config=%7b%22type%22%3a%22stdio%22%2c%22command%22%3a%22avdt%22%2c%22args%22%3a%5b%22mcp%22%5d%7d)

**方案 B：命令面板**

1. 打开命令面板（`Ctrl+Shift+P` / `Cmd+Shift+P`）。
2. 运行 **MCP: Add Server**。
3. 选择 **stdio** 作为服务器类型。
4. 输入 `avdt mcp` 作为命令。
5. 将服务器名称设为 `avalonia_devtools`。
6. 选择仅为当前工作区安装，或全局安装该服务器。

**方案 C：手动配置**

将以下内容添加到工作区根目录下的 `.vscode/mcp.json`：

```json title=".vscode/mcp.json"
{
    "servers": {
        "avalonia_devtools": {
            "type": "stdio",
            "command": "avdt",
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
        "avalonia_devtools": {
            "type": "stdio",
            "command": "avdt",
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
3. Set the command to `avdt` with argument `mcp`.
4. Set the server name to `avalonia_devtools`.

**方案 B：手动配置**

在项目目录中创建或编辑 `.idea/mcp.json`：

```json title=".idea/mcp.json"
{
    "servers": {
        "avalonia_devtools": {
            "type": "stdio",
            "command": "avdt",
            "args": ["mcp"]
        }
    }
}
```

</TabItem>
<TabItem value="cursor" label="Cursor">

**方案 A：一键安装**

[Install DevTools MCP for Cursor](https://cursor.com/en/install-mcp?name=avalonia_devtools&config=eyJ0eXBlIjoic3RkaW8iLCJjb21tYW5kIjoiYXZkdCIsImFyZ3MiOlsibWNwIl19)

**方案 B：手动配置**

将以下内容添加到项目目录中的 `.cursor/mcp.json`，或添加到 `~/.cursor/mcp.json` 以进行全局配置：

```json title=".cursor/mcp.json"
{
    "mcpServers": {
        "avalonia_devtools": {
            "command": "avdt",
            "args": ["mcp"]
        }
    }
}
```

</TabItem>
<TabItem value="claude-code" label="Claude Code">

请在终端中运行以下命令：

```bash
claude mcp add --scope user avalonia_devtools -- avdt mcp
```

验证是否已添加：

```bash
claude mcp list
```

</TabItem>
<TabItem value="claude-desktop" label="Claude Desktop">

1. 打开 **Settings** > **Developer**，然后点击 **Edit Config**。
2. 将 DevTools MCP 服务器添加到 `claude_desktop_config.json` 中：

```json
{
    "mcpServers": {
        "avalonia_devtools": {
            "command": "avdt",
            "args": ["mcp"],
            "env": {
                "AVALONIA_TOOLS_LICENSE_KEY": "your-license-key"
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

1. **检查服务器是否正在运行。** 打开编辑器中的 MCP 面板或状态指示器，确认 `avalonia_devtools` 已显示为已连接服务器。在 VS Code 中，可通过命令面板运行 **MCP: List Servers**。
2. **启动你的 Avalonia 应用程序**（如果使用预览器，也可以直接打开一个 XAML 文件）。
3. **使用提示词进行测试。** 向你的 AI 助手提问：

```text
"Connect to my running Avalonia app and show me the visual tree."
```

如果助手返回了树结构，说明配置已完成。

## 故障排查

### 找不到 “avdt” 命令

`avdt` 命令必须存在于系统 PATH 中。如果你将它作为全局 .NET 工具安装，请确保 `~/.dotnet/tools`（macOS/Linux）或 `%USERPROFILE%\.dotnet\tools`（Windows）已加入 PATH。

你可以通过运行以下命令来确认该工具是否已安装：

```bash
dotnet tool list -g
```

### 未检测到许可证密钥

如果 MCP 服务器已启动，但报告许可证密钥缺失或无效：

- **确认变量已设置。** 在启动编辑器所用的同一个终端中运行 `echo $AVALONIA_TOOLS_LICENSE_KEY`（macOS/Linux）或 `echo %AVALONIA_TOOLS_LICENSE_KEY%`（Windows）。
- **如果你的编辑器是从图形界面快捷方式启动的**，它可能不会继承 shell 环境变量。请按上文 [设置许可证密钥](#setting-your-license-key) 一节所示，在 MCP 配置中添加 `env` 块。

### 编辑器中未显示 MCP 服务器

- **重启编辑器。** 添加或修改 MCP 配置文件后，大多数编辑器都需要重启才能检测到新的 MCP 服务器。
- **检查配置文件路径。** 每个编辑器都要求配置文件位于特定路径。请参阅上文对应编辑器的配置说明。
- **校验 JSON。** 配置文件中的语法错误（缺少逗号、多余尾逗号、括号不匹配）会导致服务器静默加载失败。

### 服务器已连接，但找不到应用程序

- **确认你的应用已安装 diagnostics 包。** 你的项目必须添加 `AvaloniaUI.DiagnosticsSupport` NuGet 包，并且需要在 app builder 上调用 `.WithDeveloperTools()`，或在 `Application` 类中调用 `this.AttachDeveloperTools()`。参见上文 [准备你的应用程序](#prepare-your-application)。
- **确保 Avalonia 应用程序正在运行**，然后再让助手进行连接。
- 如果同时运行了多个 Avalonia 应用，助手会先列出它们，并询问你要附加到哪一个。
- 对于 XAML 预览，请使用 `attach-to-file`，而不是 `attach-to-app`。它会连接到 XAML 预览器，因此不要求应用处于运行状态。

### 无法附加到正在运行的应用程序

这是首次配置 MCP 服务器时最常见的问题。`attach-to-app` 工具要求以下条件全部满足：

1. 项目中已安装 `AvaloniaUI.DiagnosticsSupport` 包。
2. 应用启动时已调用 `.WithDeveloperTools()` 或 `.AttachDeveloperTools()`。
3. 应用程序正在运行，并且已经完全启动（已越过启动画面或初始化阶段）。

如果其中任意一项缺失，MCP 服务器都将无法找到你的应用程序。在应用中按 F12 对 MCP 连接没有作用；该快捷键打开的是独立的 DevTools 窗口，而不是 MCP 连接。

### 更新 DevTools

如果工具行为异常，请确认你运行的是最新版本：

```bash
dotnet tool update -g avdt
```

## 可用工具

### 连接

| 工具 | 说明 |
|------|-------------|
| `attach-to-app` | 连接到正在运行的 Avalonia 应用。如果同时运行了多个应用，会先列出供你选择。 |
| `attach-to-file` | 连接到指定文件的 XAML 预览器。用于预览 XAML 布局时，推荐优先于 `attach-to-app`。 |
| `detach` | 断开与当前应用或预览器会话的连接。 |

### 检查

| 工具 | 说明 |
|------|-------------|
| `tree` | 返回某个节点的子元素。传入 `null` 的 `nodeId` 可获取根元素。 |
| `ancestors` | 返回从某个节点向上的父级链，直到根节点。 |
| `search` | 按类型名称或 `x:Name` 查找元素。 |
| `screenshot` | 截取指定 UI 元素的 PNG 屏幕截图。 |

### 属性与样式

| 工具 | 说明 |
|------|-------------|
| `props` | 返回某个节点的全部属性值。 |
| `set-prop` | 为某个节点设置属性值。使用 `null` 或 `unset` 可以清除该值。 |
| `styles` | 返回应用于某个节点的样式及其 setter。 |
| `pseudo-class` | 在某个节点上激活伪类（例如 `:pointerover`）。省略伪类名称时，会列出可用选项。 |

### 资源与资产

| 工具 | 说明 |
|------|-------------|
| `resources` | 返回应用程序中定义的资源。也可以可选地限定到某个特定节点范围。 |
| `assets` | 列出嵌入资源（图片、字体等），并返回可供 `open-asset` 使用的 URL。 |
| `open-asset` | 根据 URL 下载嵌入资源（URL 由 `assets` 工具返回）。 |

### 交互

| 工具 | 说明 |
|------|-------------|
| `input` | 向某个 UI 元素发送输入事件（点击、按键等）。 |
| `action` | 对某个 UI 元素执行更高层级的操作。 |

## 使用示例

请用自然语言描述你想完成的事情，AI 助手会自动调用 MCP 工具：

**检查 UI：**

```text
"Connect to my running app and show me the visual tree structure."
```

**查找元素：**

```text
"Find all Button elements in my application."
```

**调试样式：**

```text
"What styles are applied to the MainWindow?"
```

**截取屏幕截图：**

```text
"Take a screenshot of the login panel."
```

**在运行时修改属性：**

```text
"Set the Background of the sidebar panel to #F0F0F0."
```

**借助屏幕截图迭代 UI 设计：**

下面的提示展示了一个完整的设计迭代工作流。AI 助手会编写 XAML，使用 `attach-to-file` 进行预览，截取屏幕截图，并不断优化，直到结果与目标设计匹配：

```text
"Create an Avalonia application and recreate the attached UI. You can write XAML
and prefer MVVM-friendly code. Use the Avalonia MCP server to periodically confirm
if designs match. Design for the light theme. If the design doesn't match, you must
continue to iterate on it until it does. Use the Avalonia MVVM template and install
AvaloniaUI.DiagnosticsSupport and add .WithDeveloperTools() to the app builder to
enable MCP. Only use the attach-to-file tool. Don't call detach. You don't need to
rebuild the project on change."
```

这个提示之所以效果很好，是因为它：

- 明确要求助手使用 `attach-to-file`（它会连接到 XAML 预览器，无需重新构建）。
- 直接在提示中包含应用埋点说明（`DiagnosticsSupport` + `.WithDeveloperTools()`），因此助手从一开始就能正确配置项目。
- 建立了一个反馈闭环，让助手持续迭代，直到设计匹配为止。

## 另请参阅

- [AI Tools 概览](/tools/ai-tools/)
- [DevTools 安装](/tools/developer-tools/installation)
- [Parcel MCP](/tools/parcel/mcp)
