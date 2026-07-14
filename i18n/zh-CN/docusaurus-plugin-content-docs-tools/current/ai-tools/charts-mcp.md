---
id: charts-mcp
title: Charts MCP
sidebar_label: Charts MCP
doc-type: how-to
description: "配置 Charts MCP 服务器，让 AI 助手可以生成 Avalonia 图表预览和代码。"
keywords:
  - mcp
  - model context protocol
  - ai assistant
  - charts
  - avalonia charts
  - copilot
  - claude
  - cursor
  - windsurf
  - gemini
  - ai tools
tags:
  - mcp
  - ai
  - avalonia charts
  - avalonia pro
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

图表模型上下文协议（MCP）服务器让 AI 助手能够根据自然语言请求创建和渲染 Avalonia 图表。你的助手可以选择图表类型、提供 JSON 数据、自定义常用选项（如大小、主题、标签和调色板），然后接收带有 C# 和 XAML 代码片段的 PNG 渲染预览。

Charts MCP 以本地 stdio MCP 服务器方式运行。它从 `Avalonia.Controls.Charts` 创建控件，并通过 Avalonia Headless 和 Skia 渲染。它可用于图表生成、图表探索和代码搭建。

:::note
Charts MCP 不会附加到正在运行的 Avalonia 应用程序，也不会检查可视化树。如需实时应用程序检查，请使用 [DevTools MCP](/tools/developer-tools/mcp)。
:::

有关 MCP 的一般介绍，请参阅 [AI 工具](/tools/ai-tools/)。

## 前提条件

在设置 MCP 服务器之前，请确保你具备以下条件：

- **已安装 .NET SDK。** 该服务器以 .NET 工具形式分发。
- **拥有包含 Charts 的有效 Avalonia Pro 许可证。** Charts 包含在 [Avalonia Pro](https://avaloniaui.net/pricing) 中。Community、Plus 和旧版 Accelerate 订阅不包含 Charts。
- **包含 Charts MCP 包的 NuGet 源。** `dotnet tool install` 命令仅在 `Avalonia.Controls.Charts.Mcp` 及其匹配的运行时包（如 `Avalonia.Controls.Charts.Mcp.osx-arm64`）可从你配置的某个 NuGet 源获取时才能正常工作。
- **支持 MCP 的编辑器或助手。** VS Code、Visual Studio、Rider、Cursor、Windsurf、Claude Code、Claude Desktop 和 Gemini CLI 均可配置 MCP 服务器。

## 安装服务器

1. 配置包含 `Avalonia.Controls.Charts.Mcp` 及其匹配运行时包的 NuGet 包源。
2. 通过运行 `dotnet tool install` 将 Charts MCP 服务器作为全局 .NET 工具安装。

```bash
dotnet tool install --global Avalonia.Controls.Charts.Mcp
```

3. 如果 `dotnet` 报告找不到包，请确认所需包源已配置并启用。运行以下命令检查你的 NuGet 源。

```bash
dotnet nuget list source
```

4. 通过运行以下命令访问 Charts MCP 服务器。大多数编辑器在配置完成后会自动执行此操作。

```bash
mcp-server-charts
```

:::tip
如果安装后命令行无法找到 `mcp-server-charts`，请检查 .NET 全局工具目录是否在你的 `PATH` 中。在 macOS 和 Linux 上，该目录通常是 `~/.dotnet/tools`。在 Windows 上，通常是 `%USERPROFILE%\.dotnet\tools`。
:::

图表渲染需要在 MCP 服务器进程中拥有有效的 Avalonia Pro（或更高版本）许可证。请在环境中设置 `AVALONIA_LICENSE_KEY`，或者如果编辑器不继承 shell 变量，请在 MCP 客户端配置中添加 `"env": { "AVALONIA_LICENSE_KEY": "your-license-key" }`。请勿提交许可证密钥。

## 从源码运行

如果你是从源码检出版本而不是已安装的工具运行：

1. 构建 MCP 项目。

```bash
dotnet build /absolute/path/to/Avalonia.Controls.Charts/src/Avalonia.Controls.Charts.Mcp/Avalonia.Controls.Charts.Mcp.csproj
```

2. 将你的 MCP 客户端配置为运行：

```bash
dotnet run --project /absolute/path/to/Avalonia.Controls.Charts/src/Avalonia.Controls.Charts.Mcp/Avalonia.Controls.Charts.Mcp.csproj
```

:::note
从源码运行适用于贡献者和本地验证。图表生成需要有效的 Avalonia Pro（或更高版本）许可证。如果图表渲染因图表初始化或许可证错误而失败，请验证构建或包中的 Charts 许可证设置。
:::

## 在编辑器中配置 MCP 服务器

以下示例使用全局 .NET 工具命令。如果你是从源码运行，请将 `"command": "mcp-server-charts"` 替换为：

```json
"command": "dotnet",
"args": [
    "run",
    "--project",
    "/absolute/path/to/Avalonia.Controls.Charts/src/Avalonia.Controls.Charts.Mcp/Avalonia.Controls.Charts.Mcp.csproj"
]
```

<Tabs groupId="editor">
<TabItem value="vscode" label="VS Code">

#### 方案 A：命令面板

1. 打开命令面板（`Ctrl+Shift+P` / `Cmd+Shift+P`）。
2. 运行 **MCP: Add Server**。
3. 选择 **stdio** 作为服务器类型。
4. 输入 `mcp-server-charts` 作为命令。
5. 将服务器名称设置为 `avalonia_charts`。
6. 选择是为当前工作区还是全局安装该服务器。

#### 方案 B：手动配置

将以下内容添加到工作区根目录的 `.vscode/mcp.json` 中：

```json title=".vscode/mcp.json"
{
    "servers": {
        "avalonia_charts": {
            "type": "stdio",
            "command": "mcp-server-charts"
        }
    }
}
```

</TabItem>
<TabItem value="visual-studio" label="Visual Studio">

Visual Studio 2022（17.x 及更高版本）支持通过 `mcp.json` 配置文件使用 MCP 服务器。

在解决方案目录中创建或编辑 `.mcp.json`：

```json title=".mcp.json"
{
    "servers": {
        "avalonia_charts": {
            "type": "stdio",
            "command": "mcp-server-charts"
        }
    }
}
```

:::tip
Visual Studio 也会读取 `.vscode/mcp.json`。如果你已经在同一解决方案中为 VS Code 配置了 Charts MCP，Visual Studio 可以自动发现该配置。
:::

</TabItem>
<TabItem value="rider" label="Rider">

JetBrains Rider 通过 AI Assistant 插件支持 MCP 服务器。

1. 打开 **Settings** → **Tools** → **AI Assistant** → **Model Context Protocol (MCP)**。
2. 点击 **Add**。
3. 选择 **STDIO**。
4. 粘贴以下 JSON 配置：

```json
{
    "mcpServers": {
        "avalonia_charts": {
            "command": "mcp-server-charts"
        }
    }
}
```

5. 点击 **OK**，然后点击 **Apply**。

</TabItem>
<TabItem value="cursor" label="Cursor">

将以下内容添加到项目目录中的 `.cursor/mcp.json`，或添加到 `~/.cursor/mcp.json` 以实现全局配置：

```json title=".cursor/mcp.json"
{
    "mcpServers": {
        "avalonia_charts": {
            "command": "mcp-server-charts"
        }
    }
}
```

</TabItem>
<TabItem value="windsurf" label="Windsurf">

将以下内容添加到 `~/.codeium/windsurf/mcp_config.json`：

```json title="~/.codeium/windsurf/mcp_config.json"
{
    "mcpServers": {
        "avalonia_charts": {
            "command": "mcp-server-charts"
        }
    }
}
```

</TabItem>
<TabItem value="claude-code" label="Claude Code">

在终端中运行以下命令：

```bash
claude mcp add --transport stdio --scope user avalonia_charts -- mcp-server-charts
```

验证是否已添加：

```bash
claude mcp list
```

</TabItem>
<TabItem value="claude-desktop" label="Claude Desktop">

1. 打开 **Settings** → **Developer**，然后点击 **Edit Config**。
2. 将 Charts MCP 服务器添加到 `claude_desktop_config.json`：

```json
{
    "mcpServers": {
        "avalonia_charts": {
            "command": "mcp-server-charts"
        }
    }
}
```

3. 保存文件。
4. 重启 Claude Desktop。

:::note
Claude Desktop 不会继承 shell 配置文件中的环境变量。如果服务器需要环境变量，请在此配置中添加 `env` 块。
:::

</TabItem>
<TabItem value="gemini-cli" label="Gemini CLI">

将以下内容添加到 `~/.gemini/settings.json` 或项目级别的 `.gemini/settings.json`：

```json title="~/.gemini/settings.json"
{
    "mcpServers": {
        "avalonia_charts": {
            "command": "mcp-server-charts"
        }
    }
}
```

</TabItem>
</Tabs>

## 验证连接

配置 MCP 服务器后，请验证其是否正常工作：

1. **检查服务器是否正在运行。** 打开编辑器的 MCP 面板或状态指示器。（在 VS Code 中，可通过命令面板运行 **MCP: List Servers**。）确认 `avalonia_charts` 已作为已连接服务器出现。
2. **用提示词测试。** 询问你的 AI 助手：

```text
列出可用的 Avalonia 图表工具，并向我展示条形图的数据格式。
```

你还可以测试图表渲染：

```text
创建一个深色主题的 Avalonia 条形图，标题为"季度收入"，Q1=125, Q2=148, Q3=171, Q4=193。使用蓝色和绿色调色板。
```

## 使用示例

用自然语言描述你想要的图表。AI 助手会自动调用 MCP 工具。

### 生成图表预览

```text
创建一个折线图，比较从一月到六月的月度注册人数。使用默认浅色主题，并包含生成的 Avalonia 代码。
```

### 选择图表类型

```text
读取 Avalonia Charts 分类资源，并推荐一种用于展示分阶段销售转化的图表。
```

### 使用特定工具

```text
使用 avalonia_sankey_chart 展示产品从获客到激活再到留存的流程。以 900x520 尺寸渲染。
```

### 创建应用代码

```text
创建一个 Avalonia 组合图，包含柱状系列（收入）和折线系列（利润率）。返回生成的 C# 和 XAML，以便我适配到 MVVM。
```

### 调试数据格式

```text
显示 avalonia_calendar_heatmap 所需的数据格式，然后渲染一个小示例。
```

## 可用工具

Charts MCP 服务器提供 86 个图表生成工具和 1 个目录工具。

### 目录

| 工具 | 说明 |
|------|-------------|
| `avalonia_list_charts` | 列出所有可用图表及其说明和示例数据格式。 |

### 笛卡尔坐标

| 工具 | 说明 |
|------|-------------|
| `avalonia_line_chart` | 趋势和时间序列数据。 |
| `avalonia_area_chart` | 带填充区域的定量数据。 |
| `avalonia_bar_chart` | 跨类别比较值。 |
| `avalonia_scatter_chart` | 两个变量之间的关系。 |
| `avalonia_combo_chart` | 组合系列，例如柱状图和折线图。 |

### 圆形

| 工具 | 说明 |
|------|-------------|
| `avalonia_pie_chart` | 整体的各部分占比。 |
| `avalonia_donut_chart` | 带中心孔的占比图。 |
| `avalonia_progress_donut` | 单值环形进度。 |
| `avalonia_semi_donut` | 半圆形占比。 |
| `avalonia_nightingale_rose` | 极坐标面积图，也称为南丁格尔玫瑰图。 |
| `avalonia_radial_bar_chart` | 呈圆形排列的柱状条目。 |
| `avalonia_sunburst_chart` | 分层占比。 |

### 仪表

| 工具 | 说明 |
|------|-------------|
| `avalonia_gauge_chart` | 标准表盘仪表。 |
| `avalonia_bullet_chart` | 与目标及范围进行比较的性能。 |
| `avalonia_circular_gauge` | 圆形指针指示器。 |
| `avalonia_linear_gauge` | 水平或垂直刻度指示器。 |
| `avalonia_gradient_ring_chart` | 以彩色圆环表示的进度或数值。 |
| `avalonia_liquid_fill_gauge` | 以液位表示的数值。 |

### 极坐标

| 工具 | 说明 |
|------|-------------|
| `avalonia_radar_chart` | 多变量比较。 |
| `avalonia_polar_chart` | 角度和径向线数据。 |
| `avalonia_wind_rose_chart` | 风向频率分布。 |
| `avalonia_smith_chart` | 复阻抗和工程数据。 |
| `avalonia_polar_area_chart` | 等角扇形图，半径可变。 |

### 统计

| 工具 | 说明 |
|------|-------------|
| `avalonia_violin_plot` | 分布密度和范围。 |
| `avalonia_boxplot_chart` | 四分位数、中位数和离群值。 |
| `avalonia_ridgeline_chart` | 多重分布比较。 |
| `avalonia_beeswarm_plot` | 单个点分布。 |
| `avalonia_contour_plot` | 二维密度等高线。 |
| `avalonia_density_plot` | 核密度估计。 |
| `avalonia_strip_plot` | 沿单一分类轴的点。 |

### 比较与排名

| 工具 | 说明 |
|------|-------------|
| `avalonia_tornado_chart` | 按类别进行配对比较。 |
| `avalonia_bump_chart` | 排名随时间的变化。 |
| `avalonia_slope_chart` | 前后状态变化。 |
| `avalonia_dumbbell_chart` | 范围或差距比较。 |
| `avalonia_pareto_chart` | 频率与累积百分比。 |
| `avalonia_mirror_bar_chart` | 背对背水平条形图。 |
| `avalonia_diverging_bar_chart` | 正负值对比。 |
| `avalonia_pyramid_chart` | 分层细分。 |
| `avalonia_population_pyramid` | 年龄和人口结构。 |
| `avalonia_funnel_chart` | 顺序流程阶段。 |

### 流程与网络

| 工具 | 说明 |
|------|-------------|
| `avalonia_sankey_chart` | 多个类别之间的流量。 |
| `avalonia_arc_diagram` | 有序节点之间的连接。 |
| `avalonia_chord_diagram` | 圆形排列中的相互关联。 |
| `avalonia_alluvial_chart` | 随步骤或时间的结构变化。 |
| `avalonia_flow_chart` | 通用流程示意图。 |
| `avalonia_network_chart` | 节点链接关系。 |
| `avalonia_force_directed_graph` | 基于物理的节点图布局。 |
| `avalonia_organization_chart` | 传统层级树。 |

### 比例与层次

| 工具 | 说明 |
|------|-------------|
| `avalonia_treemap_chart` | 用于层次结构的嵌套矩形。 |
| `avalonia_icicle_chart` | 邻接图层次结构。 |
| `avalonia_circle_packing_chart` | 嵌套在圆圈中的层次结构。 |
| `avalonia_dendrogram_chart` | 层次聚类树。 |
| `avalonia_flame_graph` | 堆栈或性能分析层次可视化。 |
| `avalonia_indented_tree_chart` | 缩进层次树。 |
| `avalonia_radial_tree_chart` | 径向排列的层次结构。 |
| `avalonia_waffle_chart` | 百分比网格。 |
| `avalonia_mekko_chart` | 可变宽度堆叠条形图。 |
| `avalonia_parliament_chart` | 席位分布半圆形图。 |
| `avalonia_venn_chart` | 集合关系与交集。 |

### 网格与矩阵

| 工具 | 说明 |
|------|-------------|
| `avalonia_heatmap_chart` | 按颜色网格表示的数值强度。 |
| `avalonia_matrix_chart` | 相关或关系矩阵。 |
| `avalonia_carpet_plot` | 两个自变量与一个因变量的关系。 |
| `avalonia_hexbin_chart` | X/Y 点的六边形箱体密度。 |
| `avalonia_mosaic_chart` | 比例分组和子分组。 |
| `avalonia_parallel_coordinates_chart` | 跨平行轴的多维记录。 |
| `avalonia_ternary_chart` | 三部分组成图。 |
| `avalonia_calendar_heatmap` | 按日期的活动分布。 |

### 时间与调度

| 工具 | 说明 |
|------|-------------|
| `avalonia_gantt_chart` | 项目管理和任务调度。 |
| `avalonia_swimlane_chart` | 按泳道分组的工作流任务。 |
| `avalonia_event_timeline` | 按时间顺序的事件。 |
| `avalonia_spiral_timeline` | 以螺旋形式表示的时间。 |

### 金融

| 工具 | 说明 |
|------|-------------|
| `avalonia_candlestick_chart` | 金融价格变动。 |
| `avalonia_ohlc_chart` | 开盘-最高-最低-收盘（OHLC）柱状图。 |
| `avalonia_heikin_ashi_chart` | 平滑的 OHLC 趋势 K 线。 |
| `avalonia_kagi_chart` | 价格反转走势图。 |
| `avalonia_renko_chart` | 固定尺寸价格变动砖形图。 |
| `avalonia_point_and_figure_chart` | X/O 价格变动列。 |

### 地图

| 工具 | 说明 |
|------|-------------|
| `avalonia_choropleth_map` | 按数值着色地理区域。 |
| `avalonia_shape_map` | 自定义多层几何形状。 |

### 仪表板、数据、文本和气泡图

| 工具 | 说明 |
|------|-------------|
| `avalonia_sparkline_chart` | 紧凑趋势线。 |
| `avalonia_kpi_card` | 带状态或变化的指标展示。 |
| `avalonia_table_chart` | 带条件样式的表格数据。 |
| `avalonia_word_cloud` | 词频云。 |
| `avalonia_bubble_chart` | 使用 X、Y 和大小值的气泡图。 |
| `avalonia_packed_bubble_chart` | 非层次气泡打包。 |
| `avalonia_bubble_cloud` | 力导向气泡云布局。 |

## 可用资源

服务器还会暴露 MCP 资源，你的助手在调用图表工具前可以读取这些资源。它们可以帮助助手选择图表类型、检查输入格式或选择调色板。

| 资源 URI | 说明 |
|--------------|-------------|
| `charts://catalog` | 按类别分组的完整图表目录。 |
| `charts://palettes` | 默认和建议的调色板值。 |
| `charts://data-formats` | 数据格式参考，源自目录示例。 |

## 输入与输出

大多数图表工具接受标题、JSON `data` 字符串、可选的宽度和高度值，以及可选的主题。某些工具暴露图表特定的参数，如 `geoJson`、`nodes` 或 `innerRadius`。

JSON 属性名不区分大小写。例如，`Label`、`label` 和 `LABEL` 映射到同一属性。

```json
[
    { "Label": "Q1", "Value": 125 },
    { "Label": "Q2", "Value": 148 },
    { "Label": "Q3", "Value": 171 },
    { "Label": "Q4", "Value": 193 }
]
```

调色板以逗号分隔的颜色代码传递：

```text
#2196F3, #4CAF50, #FF9800
```

`theme` 接受 `Light` 或 `Dark`。成功的图表生成响应包含渲染的 `image/png` 预览、生成的 C# 代码，以及生成的 XAML（如果支持）。

## 隐私与安全

Charts MCP 在本地运行。图表数据在 MCP 服务器进程中被解析和渲染，服务器不会调用外部 LLM 提供商，也不会读取提供商 API 密钥。

MCP 客户端会接收渲染的图表图像和生成的代码。生成的代码片段可能包含图表请求中的标签、值、颜色和其他字段。

请勿在 JSON、标题、标签或描述中包含机密信息、凭据或敏感个人数据，除非已获批准的 MCP 客户端有权接收这些数据。

## 故障排查

### `mcp-server-charts` 命令未找到

`mcp-server-charts` 命令必须位于系统的 `PATH` 中。如果你已将其安装为全局 .NET 工具，请检查 `PATH` 是否对编辑器可用。在 macOS 和 Linux 上，该路径通常是 `~/.dotnet/tools`，在 Windows 上，通常是 `%USERPROFILE%\.dotnet\tools`。

你可以运行以下命令确认工具是否已安装：

```bash
dotnet tool list -g
```

### 编辑器中未显示 MCP 服务器

- **重启编辑器。** 添加或修改 MCP 配置文件后，大多数编辑器都需要重启才能检测到新的 MCP 服务器。
- **检查配置文件位置。** 每种编辑器都有特定的配置路径。请参阅上方针对你所用编辑器的配置说明。
- **校验 JSON 格式。** 配置文件中的语法错误可能导致服务器无法加载。
- **使用绝对命令路径。** 这有助于编辑器解析 `mcp-server-charts`。

### 服务器已启动但图表渲染失败

- **确认 Avalonia Pro Charts 许可证。** 图表渲染需要服务器运行位置的有效许可证密钥。你可以在 [Avalonia 门户](https://portal.avaloniaui.net/)检查你的订阅和许可证密钥。
- **检查服务器日志。** 日志写入 MCP 服务器输出目录下的 `logs/AvaloniaChartsMcpServer_*.log`。这些日志可能指示启动或渲染失败。
- **先尝试较小的图表。** 调用简单的 `avalonia_bar_chart` 可以将设置问题与数据格式问题区分开来。
- **临时提高日志级别。** 尝试将 `AVALONIA_CHARTS_MCP_LOG_LEVEL` 设置为更高级别，故障排查后降低或移除。

### 格式错误

- 检查调色板和颜色值。十六进制颜色（如 `#2196F3`）最安全。
- 避免在逗号分隔的调色板字符串中出现空条目。

## 另请参阅

- [AI 工具概览](/tools/ai-tools/)
- [Avalonia Charts](/controls/data-display/charts/)
- [DevTools MCP](/tools/developer-tools/mcp)
- [Build MCP](/tools/ai-tools/build-mcp)
- [Parcel MCP](/tools/parcel/mcp)
