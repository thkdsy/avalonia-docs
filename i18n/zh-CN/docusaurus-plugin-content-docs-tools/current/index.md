---
id: index
title: Avalonia 工具
sidebar_label: Avalonia 工具
doc-type: overview
---

import DocsCard from '@site/src/components/global/DocsCard';
import DocsCards from '@site/src/components/global/DocsCards';

<head>
  <title>Avalonia 工具</title>
  <meta
    name="description"
    content="适用于 Avalonia 的专业开发者工具。可视化调试、轻松打包、更高效地构建应用。"
  />
  <style>{`
    :root {
      --doc-item-container-width: 60rem;
    }
  `}</style>
</head>

Avalonia 是一个免费、开源的 UI 框架。你可以零成本构建并发布跨平台 .NET 应用程序。该框架基于 MIT 许可证，并由一个不断壮大的团队持续维护。

开发工具链则负责围绕框架本身的各类任务：诊断布局问题、为多个操作系统打包应用，以及在编写 XAML 时实时预览界面。

## Avalonia Plus

Avalonia Plus 是一套专为 Avalonia 开发打造的专业工具集。

<DocsCards>
<DocsCard header="Dev Tools" href="/tools/developer-tools/installation" img="/icons/feature-devtools-icon.png">
  <p>以可视化方式检查并诊断 Avalonia 应用。实时编辑属性、分析性能并调试布局，无需再靠猜测。</p>
</DocsCard>

<DocsCard header="Parcel" href="/tools/parcel/setup" img="/icons/feature-parcel-icon.png">
  <p>在单一工具中为 Windows、macOS 和 Linux 打包应用。代码签名、公证和安装程序生成都可一并处理。</p>
</DocsCard>

<DocsCard header="Avalonia for Visual Studio" href="/tools/visual-studio-extension" img="/icons/feature-vs-ext-icon.png">
  <p>专为 Avalonia 打造的 Visual Studio 扩展，提供 XAML 预览、代码补全和拖放式设计器。</p>
</DocsCard>
</DocsCards>

## Avalonia Pro

Avalonia Pro 包含 Avalonia Plus 中的全部专业工具，并额外提供高级 UI 控件，例如 [TreeDataGrid](/controls/data-display/structured-data/treedatagrid/)、[NativeWebView](/controls/web/nativewebview)、[VirtualKeyboard](/controls/input/text-input/virtualkeyboard) 等。这些组件覆盖从显示层级数据到在不捆绑 Chromium 的前提下嵌入原生 Web 内容等多种场景。

## 谁可以使用

对于非商业用途，[Community 许可证](https://avaloniaui.net/pricing) 可让你免费使用 Avalonia Plus 的工具和组件。没有试用期，也没有功能阉割。

对于更大的团队和组织，也提供付费订阅。详情请参阅我们的[定价页面](https://avaloniaui.net/pricing)。

这些订阅将用于支持开源框架的持续开发。

<br />
<div style={{ display: 'flex', justifyContent: 'center', gap: '10px' }}>
  <Button label="购买 Avalonia Enterprise" link="https://avaloniaui.net/pricing" variant="secondary" outline />
</div>

## 另请参阅

- [AI 工具](/tools/ai-tools/)
- [IDE 支持](/tools/ide/)
- [常见问题](/tools/faq)
