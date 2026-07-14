---
id: build-mcp
title: Build MCP
sidebar_label: Build MCP
doc-type: how-to
description: "配置 Build MCP 服务器，让你的 AI 编程助手可以搜索 Avalonia 指南、教程、API 参考，并获得分步迁移指导。"
keywords:
  - mcp
  - model context protocol
  - ai assistant
  - documentation
  - copilot
  - claude
  - cursor
  - windsurf
  - gemini
  - ai tools
  - wpf migration
  - xpf
tags:
  - mcp
  - ai
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

## 什么是 Build MCP？

Build MCP 服务器为你的 AI 编程助手提供对 Avalonia 文档和专家开发指导的直接访问。你的助手不必依赖可能过时或不完整的训练数据，而是可以实时搜索指南、教程和 API 参考，加载 Avalonia 专用编码规则，并在创建新项目或根据截图复刻 UI 等常见工作流中使用引导式提示。

Build MCP **可免费使用**，无需许可证密钥，也无需本地安装。它以远程服务器形式运行，因此在任何兼容 MCP 的编辑器或 CLI 工具中，配置都只需几秒钟。

该服务器还提供迁移工具，可引导你的助手完成升级到最新 Avalonia Developer Tools 包，以及将 WPF 应用迁移到 Avalonia 的过程——既可以是完整的原生迁移，也可以是借助 Avalonia XPF 实现的即插即用式跨平台部署。

有关 MCP 的一般介绍，请参阅 [AI 工具](/tools/ai-tools/)。

## 可用工具

Build MCP 服务器向你的 AI 助手公开八个工具：

### 文档与规则

| 工具 | 说明 |
|------|-------------|
| `search_avalonia_docs` | 搜索完整的 Avalonia 文档，包括 API 参考、教程、指南和迁移文档。像 “styling”“binding”“mvvm” 这类常见主题会自动路由到优化过的查询，以获得更好的结果。 |
| `lookup_avalonia_api` | 在 API 参考中查询特定的 Avalonia 类、属性、方法或事件。适用于 `TextBlock`、`Window.Show`、`StyledProperty` 这类定向查询。 |
| `get_avalonia_expert_rules` | 返回一组完整的 Avalonia 开发规则，涵盖 AXAML 语法、属性系统、样式、数据绑定、MVVM 模式、自定义控件、布局、主题、资源、线程以及应避免的常见错误。建议在开发会话开始时调用它，以便助手生成正确、地道的 Avalonia 代码。 |

### 迁移

| 工具 | 说明 |
|------|-------------|
| `migrate_diagnostics` | 提供分步指导，用于配置或迁移到当前的 Avalonia Developer Tools 包。涵盖移除已弃用的 `Avalonia.Diagnostics` 包、安装 `AvaloniaUI.DiagnosticsSupport`、更新 `Program.cs` 和 `App.axaml.cs`，以及替换旧 API 调用。 |
| `analyze_wpf_project` | 用于将 WPF 应用迁移到 Avalonia 的入口工具。它会扫描项目的目标框架、WPF 引用、第三方控件套件（Telerik、DevExpress、Syncfusion、Infragistics、Actipro、SciChart、Xceed、ComponentOne）、MVVM 框架以及 P/Invoke 使用情况，然后推荐使用 Avalonia XPF 或原生 Avalonia 迁移，并根据结果移交给 `migrate_to_xpf` 或 `migrate_to_avalonia`。 |
| `migrate_to_xpf` | 提供使用 XPF 将 WPF 应用迁移到 Avalonia 的分步指导（保留现有 WPF 代码、XAML 和第三方控件的即插即用跨平台方案）。涵盖 NuGet 源配置、SDK 切换、许可证密钥设置、版本冲突处理和故障排查。 |
| `migrate_to_avalonia` | 完整原生 Avalonia 迁移的分阶段操作手册。返回指导原则（先迁移不重构、保持可运行基线、按垂直切片推进）、第 0 阶段校验、项目设置、文件迁移、构建/测试/迭代指导以及迁移后步骤。它会按需通过 `lookup_wpf_to_avalonia_mapping` 拉取聚焦的映射表，而不是一开始就加载整份参考。 |
| `lookup_wpf_to_avalonia_mapping` | 返回单个主题的聚焦式 WPF→Avalonia 对照表，让助手在迁移时只加载当前需要的内容。主题包括：`namespaces`、`controls`、`custom-controls`、`properties`、`styling`、`bindings`、`templates`、`events`、`resources`、`layout`、`threading`、`windows`、`animations`、`mvvm`、`navigation`、`gotchas`。 |

## 可用提示

除了工具之外，Build MCP 服务器还提供用于特定工作流的提示，以配置你的助手。MCP 提示是一组预先编写好的指令，用于设置助手在某项任务中的上下文与行为。

:::note
不同客户端对提示的支持程度不同。Claude Desktop、Claude Code 和 Cursor 支持 MCP 提示，其他编辑器可能不会在 UI 中暴露它们。如果你的编辑器不支持提示，也可以直接要求助手调用 `get_avalonia_expert_rules` 工具，以达到相同效果。
:::

| 提示 | 说明 |
|--------|-------------|
| `init` | 为现有项目初始化一个 Avalonia 专家会话。它会加载开发规则、设置简洁响应行为，并配置助手在处理每个技术问题时使用文档工具。 |
| `new` | 引导你创建新的 Avalonia 应用。涵盖模板选择（桌面端使用 `avalonia.mvvm`，跨平台使用 `avalonia.xplat`）、通过 CommunityToolkit.Mvvm 创建项目、配置编译绑定，以及安装开发者工具。支持可选的 `app_name` 参数。 |
| `recreate-ui` | 建立一个根据截图或图像复刻 UI 的迭代式设计工作流。助手会编写 AXAML，使用 [DevTools MCP](/tools/developer-tools/mcp) 的 `attach-to-file` 工具进行预览，截取屏幕截图与目标对比，并持续改进直到结果匹配。支持可选 `theme` 参数（`light` 或 `dark`）。DevTools MCP 集成需要 [Avalonia 许可证](https://avaloniaui.net/pricing)。 |

## 配置 MCP 服务器

Build MCP 使用远程 URL 端点。你的编辑器通过 HTTP 连接到该服务器，因此无需进行本地安装。

请在下方选择你的编辑器或 CLI 工具：

<Tabs groupId="editor">
<TabItem value="vscode" label="VS Code">

**方案 A：命令面板**

1. Open the command palette (`Ctrl+Shift+P` / `Cmd+Shift+P`).
2. Run **MCP: Add Server**.
3. Select **HTTP** as the server type.
4. Enter `https://docs-mcp.avaloniaui.net/mcp` as the URL.
5. Set the server name to `avalonia-docs`.
6. Choose whether to install the server for this workspace or globally.

**方案 B：手动配置**

Add the following to `.vscode/mcp.json` in your workspace root:

```json title=".vscode/mcp.json"
{
    "servers": {
        "avalonia-docs": {
            "type": "http",
            "url": "https://docs-mcp.avaloniaui.net/mcp"
        }
    }
}
```

</TabItem>
<TabItem value="visual-studio" label="Visual Studio">

Visual Studio 2022（17.x 及更高版本）支持通过 `mcp.json` 配置文件使用 MCP 服务器。

请将以下内容添加到解决方案目录中的 `.vscode/mcp.json`：

```json title=".vscode/mcp.json"
{
    "servers": {
        "avalonia-docs": {
            "type": "http",
            "url": "https://docs-mcp.avaloniaui.net/mcp"
        }
    }
}
```

:::tip
Visual Studio 与 VS Code 读取同一个 `.vscode/mcp.json` 路径。如果你已经为 VS Code 配置过，它会自动在 Visual Studio 中生效。
:::

</TabItem>
<TabItem value="rider" label="Rider">

JetBrains Rider 通过 AI Assistant 插件和 GitHub Copilot 插件支持 MCP 服务器。

**方案 A：设置界面**

1. Open **Settings** > **Tools** > **AI Assistant** > **MCP Servers**.
2. Click **Add** and select **Streamable HTTP** as the transport type.
3. Enter `https://docs-mcp.avaloniaui.net/mcp` as the URL.
4. Set the server name to `avalonia-docs`.

**方案 B：手动配置**

Create or edit `.idea/mcp.json` in your project directory:

```json title=".idea/mcp.json"
{
    "servers": {
        "avalonia-docs": {
            "type": "http",
            "url": "https://docs-mcp.avaloniaui.net/mcp"
        }
    }
}
```

</TabItem>
<TabItem value="cursor" label="Cursor">

Add the following to `.cursor/mcp.json` in your project directory, or to `~/.cursor/mcp.json` for global configuration:

```json title=".cursor/mcp.json"
{
    "mcpServers": {
        "avalonia-docs": {
            "url": "https://docs-mcp.avaloniaui.net/mcp"
        }
    }
}
```

</TabItem>
<TabItem value="windsurf" label="Windsurf">

Add the following to `~/.codeium/windsurf/mcp_config.json`:

```json title="~/.codeium/windsurf/mcp_config.json"
{
    "mcpServers": {
        "avalonia-docs": {
            "serverUrl": "https://docs-mcp.avaloniaui.net/mcp"
        }
    }
}
```

</TabItem>
<TabItem value="claude-code" label="Claude Code">

请在终端中运行以下命令：

```bash
claude mcp add --transport http avalonia-docs https://docs-mcp.avaloniaui.net/mcp
```

验证是否已添加：

```bash
claude mcp list
```

</TabItem>
<TabItem value="claude-desktop" label="Claude Desktop">

1. Go to **Customize** → **Connectors**.
2. Click the **+** button at the top of the window, then **Add custom connector**.
3. In the "Name" field, input "avalonia-docs".
4. In the "Remote MCP server URL" field, input `https://docs-mcp.avaloniaui.net/mcp`.
5. Click **Add**.
6. You may need to restart Claude Desktop.

</TabItem>
<TabItem value="gemini-cli" label="Gemini CLI">

Add the following to `~/.gemini/settings.json` (or the project-level `.gemini/settings.json`):

```json title="~/.gemini/settings.json"
{
    "mcpServers": {
        "avalonia-docs": {
            "httpUrl": "https://docs-mcp.avaloniaui.net/mcp"
        }
    }
}
```

</TabItem>
</Tabs>

## 验证连接

完成 MCP 服务器配置后，请按以下方式验证它是否正常工作：

1. **检查服务器是否已列出。** 打开编辑器中的 MCP 面板或状态指示器，确认 `avalonia-docs` 已作为已连接服务器出现。在 VS Code 中，可通过命令面板运行 **MCP: List Servers**。
2. **用提示词测试。** 询问你的 AI 助手：

```text
"Search the Avalonia docs for how to set up data binding."
```

如果助手返回了带来源链接的文档结果，说明配置已完成。

## 故障排查

### 编辑器中未显示 MCP 服务器

- **重启编辑器。** 添加或修改 MCP 配置文件后，大多数编辑器都需要重启才能检测到新服务器。
- **检查配置文件路径。** 每种编辑器都有特定的配置路径。请参阅上方针对你所用编辑器的配置说明。
- **校验 JSON 格式。** 配置文件中的语法错误（如缺少逗号、多余尾逗号、括号不匹配）会导致服务器静默加载失败。

### 服务器已显示，但工具不可用

- **确认编辑器支持 HTTP 传输。** 某些较旧版本的编辑器只支持基于 STDIO 的 MCP 服务器。请将编辑器更新到最新版本。
- **检查网络连通性。** 服务器托管在 `docs-mcp.avaloniaui.net`。请确认你的网络可以访问该域名。

### 结果看起来过时

Build MCP 服务器索引的是已发布的 Avalonia 文档。如果你发现内容过时，可能是文档站点本身尚未更新。可直接访问 [docs.avaloniaui.net](https://docs.avaloniaui.net) 进行确认。

## 使用示例

请用自然语言描述你想完成的事情，AI 助手会自动调用 MCP 工具：

**搜索文档：**

```text
"Search the Avalonia docs for how to use TreeView with data binding."
```

**查询 API 类型：**

```text
"Look up the Avalonia TextBlock control in the API reference."
```

**在会话开始时加载专家规则：**

```text
"Load the Avalonia expert rules so you can help me build my app correctly."
```

**创建新项目（使用 `new` 提示）：**

```text
"Create a new Avalonia desktop app called WeatherTracker."
```

**根据截图复刻 UI（使用 `recreate-ui` 提示）：**

```text
"Recreate this UI in Avalonia. Use the light theme."
```

这个提示在结合 [DevTools MCP](/tools/developer-tools/mcp) 使用时效果最好，因为后者提供了可用于实时 XAML 预览的 `attach-to-file` 工具。助手会编写 AXAML、进行预览、截取屏幕截图，并持续迭代，直到结果与你的目标设计一致。DevTools MCP 需要 [Avalonia 许可证](https://avaloniaui.net/pricing)。

**迁移 WPF 应用：**

```text
"Analyze my WPF project and recommend the best migration path to Avalonia."
```

助手会调用 `analyze_wpf_project`，扫描你的项目中的目标框架、WPF 引用、第三方控件套件（Telerik、DevExpress、Syncfusion、Infragistics、Actipro、SciChart、Xceed、ComponentOne）、MVVM 框架以及平台特定代码。根据扫描结果，它会推荐以下两种路径之一，并移交给对应工具：

- **Avalonia XPF** —— 保留现有 WPF 代码、XAML 和第三方控件的即插即用跨平台方案。助手会调用 `migrate_to_xpf`，引导你完成 NuGet 配置、SDK 切换和许可证设置。
- **原生 Avalonia** —— 完整迁移到现代 Avalonia 控件和主题。助手会调用 `migrate_to_avalonia` 获取分阶段迁移手册，并在迁移每个文件时，通过 `lookup_wpf_to_avalonia_mapping` 拉取按主题聚焦的映射表（控件、属性、样式、绑定、事件、模板、常见陷阱等）。这样可以保持上下文聚焦——助手只加载当前正在处理部分所需的映射内容。

**配置 Avalonia Developer Tools：**

```text
"Help me set up the latest Avalonia Developer Tools in my project."
```

助手会调用 `migrate_diagnostics` 工具，引导你安装 `AvaloniaUI.DiagnosticsSupport`，并在发现已弃用的 `Avalonia.Diagnostics` 包时将其移除。

## 另请参阅

- [AI 工具概览](/tools/ai-tools/)
- [DevTools MCP](/tools/developer-tools/mcp)
- [Parcel MCP](/tools/parcel/mcp)
