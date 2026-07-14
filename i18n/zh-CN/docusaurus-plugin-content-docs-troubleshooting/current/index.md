---
id: index
title: 故障排除
sidebar_label: 首页
---

import DocsCard from '@site/src/components/global/DocsCard';
import DocsCards from '@site/src/components/global/DocsCards';

<head>
  <title>Avalonia 文档：故障排除</title>
  <meta
    name="description"
    content="查找有关诊断和解决 Avalonia 应用中常见问题的指导，包括安装、性能、平台特定行为和 UI 开发。"
  />
  <style>{`
    :root {
      --doc-item-container-width: 60rem;
    }
  `}</style>
</head>

在构建 Avalonia 应用时，查找有关诊断和解决常见问题的帮助。无论你遇到的是安装问题、性能瓶颈，还是样式与主题方面的挑战，这些页面都将为你提供针对性的指导，帮助你的应用回到正轨。

## 常规

<DocsCards>
  <DocsCard header="安装" href="/troubleshooting/installation">
    <p>设置 .NET 和 Avalonia 时遇到的常见问题。</p>
  </DocsCard>
  <DocsCard header="应用性能问题" href="/troubleshooting/app-performance-issues">
    <p>提升 Avalonia 应用运行时性能的步骤。</p>
  </DocsCard>
</DocsCards>

## 控件

<DocsCards>
  <DocsCard header="MediaPlayer" href="/troubleshooting/controls/mediaplayer">
    <p>Avalonia Pro MediaPlayer 控件的常见问题。</p>
  </DocsCard>
  <DocsCard header="MessageBox" href="/troubleshooting/controls/messagebox">
    <p>显示消息对话框的选项，包括第三方替代方案。</p>
  </DocsCard>
</DocsCards>

## 平台特定问题

<DocsCards>
  <DocsCard header="macOS" href="/troubleshooting/platform-specific-issues/macos">
    <p>macOS 特有的问题，包括应用菜单和系统集成。</p>
  </DocsCard>
  <DocsCard header="WebAssembly" href="/troubleshooting/platform-specific-issues/webassembly">
    <p>在浏览器中运行 Avalonia 应用时的常见问题，包括缺少本机库。</p>
  </DocsCard>
  <DocsCard header="Windows" href="/troubleshooting/platform-specific-issues/windows">
    <p>Windows 特有的问题，包括打包、签名和 SmartScreen 警告。</p>
  </DocsCard>
</DocsCards>

## 工具

<DocsCards>
  <DocsCard header="开发者工具" href="/troubleshooting/tools/developer-tools">
    <p>使用 Avalonia DevTools 检查和调试应用时的常见问题。</p>
  </DocsCard>
</DocsCards>

## UI 开发

<DocsCards>
  <DocsCard header="样式" href="/troubleshooting/ui-development/styles">
    <p>样式选择器的常见问题，包括静默失败和未匹配的目标。</p>
  </DocsCard>
  <DocsCard header="主题" href="/troubleshooting/ui-development/themes">
    <p>控件主题的常见问题，包括主题查找和意外的副作用。</p>
  </DocsCard>
</DocsCards>