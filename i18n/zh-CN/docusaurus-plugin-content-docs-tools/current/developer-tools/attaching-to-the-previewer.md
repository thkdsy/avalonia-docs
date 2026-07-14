---
id: attaching-to-the-previewer
title: 将 DevTools 附加到预览器
sidebar_label: 附加到预览器
description: 了解如何将 Avalonia Developer Tools 附加到 XAML 预览器进程，以进行可视树检查和诊断。
doc-type: how-to
tags:
  - avalonia plus
  - avalonia pro
  - avalonia enterprise
---

:::caution
此功能为实验性功能，未来版本中可能会发生变化。
:::

[AvaloniaVS](https://marketplace.visualstudio.com/items?itemName=AvaloniaTeam.AvaloniaVS) 和 [AvaloniaRider](https://plugins.jetbrains.com/plugin/14839-avaloniarider) 扩展会在完整的应用进程中运行预览器窗口，但并不包含真实的窗口子系统。这会限制你可用的诊断能力，使可视树分析和实际控件位置检查变得更加困难。

由于 Developer Tools 可以在进程外运行，你可以将其附加到预览器进程，从而获得完整的诊断能力，包括可视树检查、属性编辑和布局分析。

![Example of DevTools app attached to the previewer process](/img/tools/dev-tools/attaching-to-previewer.png)

## 配置

预览扩展不支持键盘输入，因此目前 `AutoConnectFromDesignMode` 是你唯一可用的连接方式。将以下代码添加到应用启动代码中：

```csharp title="App.axaml.cs"
this.AttachDeveloperTools(o =>
{
    o.AutoConnectFromDesignMode = true;
});
```

默认情况下，当 `IsDesignMode` 为 `true` 时，`DeveloperToolsOptions.Runner` 会被禁用。这样可以避免每次你在 IDE 中打开 XAML 文件时都启动不必要的进程。

由于 runner 被禁用，你需要单独打开 Developer Tools 应用（与浏览器和移动端目标使用的方式相同）。

## 故障排查

### 快捷键无效

如上所述，预览器扩展不会监听键盘输入。你无法在预览器内部使用键盘快捷键触发 Developer Tools。请改为直接使用 Developer Tools 应用窗口中的操作按钮或快捷键。

### Developer Tools 打开过多窗口

Developer Tools 会为每个已连接进程打开一个工具窗口。如果你在 IDE 中打开了多个 XAML 预览器标签页，就会为每个标签页分别打开一个工具窗口。为了减少干扰，请关闭那些当前不需要检查的预览器标签页。

## 另请参阅

- [附加应用程序](/tools/developer-tools/attaching-applications)
- [附加到远程工具](/tools/developer-tools/attaching-to-the-remote-tool)
- [Developer Tools 选项](/tools/developer-tools/options)
- [Developer Tools 快捷键](/tools/developer-tools/shortcuts)
