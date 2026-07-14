---
id: legacy-developer-tools
title: 使用旧版开发者工具
doc-type: reference
---

import DevToolsOverviewScreenshot from '/img/guides/development-optimization/devtools-overview.png';
import DevToolsPropertiesScreenshot from '/img/guides/development-optimization/devtools-properties.png';
import DevToolsLayoutScreenshot from '/img/guides/development-optimization/devtools-layout.png';
import DevToolsStylesScreenshot from '/img/guides/development-optimization/devtools-styles.png';
import DevToolsOverriddenStylesScreenshot from '/img/guides/development-optimization/devtools-overridden-styles.png';
import DevToolsSetterContextMenuScreenshot from '/img/guides/development-optimization/devtools-setter-contextmenu.png';
import DevToolsEventsScreenshot from '/img/guides/development-optimization/devtools-events.png';
import DevToolsChangePropertyScreenshot from '/img/guides/development-optimization/devtools-change-property.gif';
import DevToolsChangeLayoutScreenshot from '/img/guides/development-optimization/devtools-change-layout.gif';

:::note

本文档介绍的是旧版开发者工具。新版 [AvaloniaUI Developer Tools](/tools/developer-tools/installation) 已可用，提供了增强的功能和更完善的调试能力。旧版开发者工具仍会继续获得稳定性更新支持。

:::

## 附加 DevTools

Avalonia 内置了一个 DevTools 窗口，可以通过在 `Window` 构造函数中调用附加的 `AttachDevTools()` 方法来启用。默认模板会在程序以 `DEBUG` 模式编译时自动启用它：

```csharp
public partial class MainWindow : Window
{
    public MainWindow()
    {
        InitializeComponent();
    }
}

// 在 Avalonia.NameGenerator 自动生成的 MainWindow.g.cs 中：
partial class MainWindow
{
    // ...
    public void InitializeComponent(bool loadXaml = true, bool attachDevTools = true)
    {
        // ...
#if DEBUG
        if (attachDevTools)
        {
            this.AttachDevTools();
        }
#endif
        // ...
    }
}
```

要打开 DevTools，请按 <kbd>F12</kbd>，或向 `this.AttachDevTools()` 方法传入不同的 `Gesture`。

:::info
要使用 DevTools，你必须添加 `Avalonia.Diagnostics` NuGet 包。

```bash
dotnet add package Avalonia.Diagnostics
```

不过默认情况下它已经安装好了。
:::

<Image light={DevToolsOverviewScreenshot} alt="DevTools overview window" position="center" maxWidth={400} cornerRadius="true" />

在 .NET Core 2.1 下运行时有一个已知问题：按下 <kbd>F12</kbd> 会导致程序退出。遇到这种情况时，可以切换到 .NET Core 2.0 或 3.0+，或者将打开手势改为其他组合，例如 <kbd>Ctrl</kbd>+<kbd>F12</kbd>。

## 逻辑树与可视树

`Logical Tree` 和 `Visual Tree` 选项卡会显示窗口中的逻辑树和可视树控件。选择某个控件后，会在右侧面板中显示该控件的属性，并且可以直接编辑。

### 属性

可用于快速查看和编辑控件属性。你还可以按名称或使用正则表达式搜索属性。

| 列 | 说明 |
| -------- | ----------------------------- |
| Property | 属性名称 |
| Value    | 属性当前值 |
| Type     | 当前值的类型 |
| Priority | 值的优先级 |

<Image light={DevToolsPropertiesScreenshot} alt="DevTools properties panel" position="center" maxWidth={400} cornerRadius="true" />

### 布局

可用于检查和编辑常见布局属性（`Margin`、`Border`、`Padding`）。\
同时也会显示控件大小和尺寸约束。

:::info
如果 `Width` 或 `Height` 带有下划线，表示当前存在生效的约束。将鼠标悬停在该值上可以查看包含相关信息的工具提示。
:::

<Image light={DevToolsLayoutScreenshot} alt="DevTools layout panel" position="center" maxWidth={400} cornerRadius="true" />

### 样式

虽然[属性](#属性)面板显示的是当前生效的属性值，但样式面板会显示所有值以及这些值的来源。

此外，你还可以查看所有可能匹配该控件的样式（通过切换 `Show inactive` 选项）。

可以通过点击 **Snapshot** 按钮，或在鼠标悬停于目标窗口时按下 <kbd>Alt</kbd>+<kbd>S</kbd>，为当前样式创建快照。创建快照后，样式面板不会再随着控件状态变化而更新。这在排查 `:pointerover` 或 `:pressed` 选择器相关问题时尤其有用。

:::info
如果 setter 的值绑定到了某个资源，它会以一个圆点加资源键名的形式显示出来。
:::


<Image light={DevToolsStylesScreenshot} alt="DevTools styles panel" position="center" maxWidth={400} cornerRadius="true" />

:::info
如果某个值显示为删除线，表示它已被优先级更高的样式值覆盖。
:::

<Image light={DevToolsOverriddenStylesScreenshot} alt="DevTools styles panel showing overridden values with strikethrough" position="center" maxWidth={400} cornerRadius="true" />

Setter 带有上下文菜单，可用于快速将名称和值复制到剪贴板。


<Image light={DevToolsSetterContextMenuScreenshot} alt="DevTools setter context menu" position="center" maxWidth={400} cornerRadius="true" />

## 事件

Events 选项卡可用于跟踪[事件](/docs/input-interaction/routed-events)的传播过程。在左侧面板中选择要跟踪的事件后，该类型的所有事件都会显示在中上方区域。选择其中一个事件即可查看事件路由。

:::info
事件名称或控件类型下方的点状下划线表示可以快速导航。

* 双击事件类型会选中并滚动到对应事件类型
* 双击控件类型（和/或名称）会跳转到可视树选项卡并选中该控件。
:::

<Image light={DevToolsEventsScreenshot} alt="DevTools events tab" position="center" maxWidth={400} cornerRadius="true" />

## 快捷键

| 按键组合 | 功能 |
| ---------------- | ------------------------------|
| <kbd>Alt</kbd>+<kbd>S</kbd> | 启用样式快照 |
| <kbd>Alt</kbd>+<kbd>D</kbd> | 禁用样式快照 |
| <kbd>Ctrl</kbd>+<kbd>Shift</kbd> | 检查指针所在控件 |
| <kbd>Ctrl</kbd>+<kbd>Alt</kbd>+<kbd>F</kbd> | 切换 Popup 冻结 |
| <kbd>F8</kbd> | 为逻辑树或可视树中的选中项截图 |

## 示例

### 修改属性值

<Image light={DevToolsChangePropertyScreenshot} alt="Animation showing a property value being changed in DevTools" position="center" maxWidth={400} cornerRadius="true" />

### 修改布局属性

<Image light={DevToolsChangeLayoutScreenshot} alt="Animation showing layout properties being changed in DevTools" position="center" maxWidth={400} cornerRadius="true" />

## 另请参阅

- [Developer tools 安装](/tools/developer-tools/installation)
- [Avalonia Tools 概览](/tools/)
