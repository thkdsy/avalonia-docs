---
id: index
title: Avalonia XPF
sidebar_label: Avalonia XPF
---

<head>
  <title>Avalonia 文档：XPF</title>
  <meta
    name="description"
    content="Avalonia XPF 使用 Avalonia 的跨平台引擎在 macOS、Linux、iOS、Android 和 WebAssembly 上运行您现有的 WPF 应用程序，只需极少的代码改动。"
  />
</head>

import TierBadge from '@site/src/components/global/TierBadge';

您花了多年构建 WPF 应用程序。它能正常工作。您的团队熟悉这套代码库。您的客户依赖它。现在有人在要求支持 macOS、Linux 或 Web。

一次完全重写并不是几个冲刺就能完成的事。至少需要数月，通常是数年。每一个屏幕、每一个边缘情况、每一个被团队遗忘的变通方案，都必须重新发现并重建。如果您的应用程序依赖 DevExpress、Telerik 或 Syncfusion 之类的第三方控件套件，成本会成倍增加：这些集成无法直接沿用，而替换它们意味着评估新的供应商、学习新的 API，并失去您已经发布的功能。与此同时，您的团队被分割在两个做同样事情的代码库之间。每个功能都要发布两次。每个修复都要发布两次。时间拖得越久，您在真正重要的产品工作上就会落后得越多。

Avalonia XPF 是一条不同的路径。它用 Avalonia 的跨平台引擎替换 WPF 下方的渲染层，同时保留您的代码所依赖的 API 和二进制兼容性。您的 XAML、视图模型，以及来自 Telerik、DevExpress、Infragistics、Actipro 和 Syncfusion 等供应商的第三方控件都能继续正常工作。您不是在重写应用程序，而是在别的地方运行它。

## 工作原理

XPF 用 Avalonia 的渲染引擎替换了 WPF 的底层渲染组件（MilCore）。该层之上的一切保持不变。您的应用程序看到的仍然是它一直以来使用的 WPF API，只不过这些 API 现在可以在 macOS、Linux 以及更多平台上工作。

这意味着：

- **对于大多数应用程序无需代码改动。** 如果它今天能针对 WPF 编译，那么它大概率也能针对 XPF 编译。
- **第三方控件继续正常工作。** XPF 保持二进制兼容性，因此主要供应商的控件无需修改即可运行。
- **您只需交付一个代码库。** 没有分支，没有并行重写，没有平台之间的漂移。

## 混合 XPF

XPF 并不是非此即彼的选择。借助 [Hybrid XPF](/xpf/interop/using-xpf-in-avalonia)，您可以在同一个应用程序中混合使用 Avalonia 控件和 WPF 控件。这意味着您可以先从 XPF 开始，以便今天就实现跨平台，然后按照自己的节奏逐步用原生 Avalonia 视图替换 WPF 视图。随着时间推移，XPF 会成为通往完整 Avalonia 迁移的垫脚石，没有一次性大改，也不会出现应用程序停止工作的时刻。

反过来也同样适用。如果您已经在使用 Avalonia 构建，Hybrid XPF 让您无需等待原生 Avalonia 移植，就能访问来自 Telerik、DevExpress、Infragistics、Actipro 和 Syncfusion 等供应商的 700 多个现有 WPF 控件。

## 平台支持

| 平台 | 内部 | 商业 | 企业 |
|---|---|---|---|
| [Windows](/docs/supported-platforms#windows) | <TierBadge tier={1} /> | <TierBadge tier={1} /> | <TierBadge tier={1} /> <TierBadge tier={2} /> <TierBadge tier={3} /> |
| [macOS](/docs/supported-platforms#macos) | <TierBadge tier={1} /> | <TierBadge tier={1} /> | <TierBadge tier={1} /> <TierBadge tier={2} /> <TierBadge tier={3} /> |
| [桌面 Linux](/docs/supported-platforms#desktop-linux) | <TierBadge tier={1} /> | <TierBadge tier={1} /> | <TierBadge tier={1} /> <TierBadge tier={2} /> <TierBadge tier={3} /> |
| [嵌入式 Linux](/docs/supported-platforms#embedded-linux) | | 付费附加项 | <TierBadge tier={1} /> <TierBadge tier={2} /> <TierBadge tier={3} /> |
| [iOS](/xpf/platforms/mobile-and-browser) | | | <TierBadge tier={1} /> <TierBadge tier={2} /> <TierBadge tier={3} /> |
| [Android](/xpf/platforms/mobile-and-browser) | | | <TierBadge tier={1} /> <TierBadge tier={2} /> <TierBadge tier={3} /> |
| [WebAssembly](/xpf/platforms/mobile-and-browser) | | | <TierBadge tier={1} /> |

所有层级都支持 Avalonia 的 [Tier 1 平台](/docs/supported-platforms)。企业许可证包括 Tier 2 支持，并可按个案安排 Tier 3 覆盖。

## 许可

XPF 是一款商业产品，分为三个层级：**内部**、**商业**和**企业**。所有许可证都是永久性的，这意味着无论许可证状态如何，您的应用程序都可以继续工作。

| | 内部 | 商业 | 企业 |
|---|---|---|---|
| macOS、桌面 Linux | 是 | 是 | 是 |
| 混合使用 Avalonia 控件 | | 是 | 是 |
| 跨平台 System.Drawing | | 是 | 是 |
| 嵌入式 Linux | | 付费附加项 | 是 |
| iOS、Android、WebAssembly | | | 是 |
| SLA | 10 个工作日 | 5 个工作日 | 3 个工作日 |

每个许可证都包含：

- 完整支持的 **30 天免费试用**
- 使用 XPF 构建的永久许可证
- 12 个月的更新和工程支持

企业试用版可通过联系销售获取。有关定价，请参阅 [Avalonia 网站](https://avaloniaui.net/xpf?utm_source=docs&utm_medium=referral&utm_content=xpf-index#pricing)。

## 开始使用

[入门指南](/xpf/getting-started) 会带您了解如何在新平台上运行您的 WPF 应用程序。大多数团队在几分钟内就能上手，而不是几个月。