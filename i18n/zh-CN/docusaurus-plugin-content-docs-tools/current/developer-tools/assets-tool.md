---
id: assets-tool
title: 资源文件工具
description: 使用 Developer Tools 的 Assets 面板浏览、搜索、预览并复制正在运行的应用程序中所有嵌入式 Avalonia 资源的 URI。
doc-type: reference
---

Assets 工具会显示运行中进程内嵌入的所有 Avalonia 资源列表。

这其中也包括来自依赖项的嵌入资源，例如第三方主题或图标库，因为它们同样会作为 Avalonia 资源提供。

![Assets Page](/img/tools/dev-tools/assets-page.png)

## 浏览资源文件列表

Assets 工具会以网格视图展示所有嵌入资源。每一行会显示资源名称及其所属程序集来源。你可以使用顶部的搜索框按名称或路径过滤资源。

资源 URI 使用 `avares://` 方案。例如，`avares://MyApp/Assets/logo.png` 表示 `MyApp` 程序集中的 `Assets` 文件夹下名为 `logo.png` 的文件。

## 资源文件上下文菜单

右键单击任意资源即可打开上下文菜单。在这里你可以复制该资源的绝对 URI，或将资源导出到文件系统中。

复制得到的 URI 可以直接在 XAML 中使用。例如，在复制某个图像资源 URI 后：

```xml
<Image Source="avares://MyApp/Assets/logo.png" />
```

![Asset context menu](/img/tools/dev-tools/assets-context-menu.png)

## 资源文件预览

网格列表只显示每个资源的有限信息，以避免不必要地将它们读取到内存中。

要预览某个资源，可以双击它，或从上下文菜单中选择 **Preview**。该工具会从应用进程中下载资源并将其显示出来。支持预览的格式包括：

- **位图图像**（PNG、JPEG、BMP 以及其他位图格式）
- **字体**（TrueType 和 OpenType）
- **文本文件**（XAML、XML、JSON 和纯文本）

对于图像资源，预览还会显示其位图格式和解码后的像素尺寸。

:::note
任何大于 100mb 的资源都无法预览，并且目前这一限制不可配置。
:::

![Image Asset preview example](/img/tools/dev-tools/assets-image.png)

![Font Asset preview example](/img/tools/dev-tools/assets-font.png)

## 另请参阅

- [资源文件基础](/docs/fundamentals/including-assets)
- [资源工具](/tools/developer-tools/resources-tool)
- [元素工具](/tools/developer-tools/elements-tool)
