---
id: index
title: AI 工具
doc-type: overview
---

Avalonia 提供了 MCP（Model Context Protocol，模型上下文协议）服务器，可将 AI 编程助手连接到文档、运行中的应用程序以及打包工作流。你无需再复制报错信息，或用文字描述 UI；AI 助手可以直接搜索官方文档、检查可视树、截取屏幕截图、设置属性，甚至帮你完成应用打包。

## 什么是 MCP？

Model Context Protocol（MCP）是一项开放标准，它允许 AI 模型通过统一接口使用外部工具和服务。你不必再手动运行命令并把输出粘贴到对话中，MCP 让 AI 助手可以直接调用工具、双向传递结构化数据。这样就形成了更紧密的反馈闭环：助手能够看到你的应用、进行修改并验证结果，而无需你充当中间人。

## 支持的 AI 助手

每个 MCP 配置页面都包含以下编辑器和命令行工具的分步配置说明：

- VS Code with GitHub Copilot
- Visual Studio with Copilot
- JetBrains Rider (AI Assistant and Copilot plugins)
- Cursor
- Windsurf
- Claude Code
- Claude Desktop
- Gemini CLI

## Build MCP

Build MCP 服务器为你的 AI 编程助手提供对 Avalonia 文档和专家级开发规则的直接访问。助手可以实时搜索指南、教程和 API 参考，加载适用于惯用 Avalonia 开发的完整编码规则，并在创建新项目、根据截图复刻 UI、或将 WPF 应用迁移到 Avalonia 等常见工作流中使用引导式提示。

Build MCP **可免费使用**，无需许可证密钥，也无需本地安装。它以远程服务器形式运行，并通过 HTTP 连接，因此配置只需几秒钟。

[配置 Build MCP](/tools/ai-tools/build-mcp)

## DevTools MCP

DevTools MCP 服务器为你的 AI 助手提供对运行中 Avalonia 应用的直接访问。它可以连接到实时应用或 XAML 预览器，检查可视树，按类型或名称搜索元素，读取和修改属性，截取屏幕截图，并发送输入事件。

这对于调试布局问题尤其有用。你无需再口头描述问题，AI 助手可以直接看到它、检查相关属性，并在同一段对话中提出或应用修复方案。

[配置 DevTools MCP](/tools/developer-tools/mcp)

## Parcel MCP

Parcel MCP 服务器让你的 AI 助手能够处理应用打包工作。它可以根据你的 .NET 项目生成 Parcel 配置，设置代码签名与公证，并为 Windows、macOS 和 Linux 构建安装程序。

借助 Parcel MCP 服务器，你只需用自然语言描述需求，AI 助手就会处理配置与执行过程，包括 macOS 的签名与公证。

[配置 Parcel MCP](/tools/parcel/mcp)

## 另请参阅

- [Build MCP](/tools/ai-tools/build-mcp)
- [DevTools MCP](/tools/developer-tools/mcp)
- [Parcel MCP](/tools/parcel/mcp)
- [Avalonia 工具概览](/tools/)
