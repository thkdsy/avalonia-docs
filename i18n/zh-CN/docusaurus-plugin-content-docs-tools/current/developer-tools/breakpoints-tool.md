---
id: breakpoints-tool
title: Breakpoints 工具
doc-type: reference
---

应用程序 Breakpoints 工具允许你在不修改代码的情况下监视和调试 Avalonia 应用中的属性变化与事件。你可以在属性和事件上设置断点，以帮助诊断问题并理解应用行为。

断点在以下情况下会被视为 `Hit`（相应地会增加 `Hit Count` 值，或暂停执行）：
1. 属性发生变化（属性断点）或事件被触发（事件断点）。
2. 断点已启用。
3. 如果断点指定了目标，则该目标与属性/事件来源匹配。
4. 满足命中次数条件。

根据断点的创建方式，它可能会带有 `Target`。
例如，没有目标的事件断点会被视为全局断点，只要 _任意_ 元素触发该事件就会命中。

![List of breakpoints with options panel](/img/tools/dev-tools/breakpoints-list.png)

## 添加断点

### 添加属性断点

在 [Properties](/tools/developer-tools/elements-tool) 列表中，每个依赖属性都有一个 **Set Breakpoint** 上下文菜单项。

创建出的断点会绑定到设置它的那个元素上。

![Setting breakpoint on a property](/img/tools/dev-tools/breakpoint-set-on-propety.png)

### 添加事件断点

在 [Events](/tools/developer-tools/events-tool) 工具中，每个已触发事件都有设置断点的选项。

选择 **On a Source** 会将断点绑定到此前触发该事件的源元素上。或者，选择 **Globally** 会创建一个未绑定断点，它会在任意元素触发该事件时命中。

![Setting breakpoint on a raised event](/img/tools/dev-tools/breakpoint-set-on-raised-event.png)

你也可以将断点绑定到某个特定的路由链元素上，或者从 “Event Listeners” 浮出面板中设置。

![Setting breakpoint on a chain element](/img/tools/dev-tools/breakpoint-set-on-chain-element.png)

## 管理断点

默认情况下，任何断点在被触发时都只会增加 Hit Count。

还可以启用以下几个选项：

### 暂停执行

这与常见 IDE 中断点的工作方式类似：停止执行并将你导航到断点位置。

当你需要查看究竟是什么触发了属性变化或某个事件时，这个选项尤其有用。你可以在 IDE 中通过调用堆栈进行分析。

已连接的应用必须附加了第三方调试器，否则该断点会被忽略。

由于 `Developer Tools` 使用的是标准的 `Debugger.Break()` 方法，因此任何带调试器的常见 IDE 都可以工作，例如 Visual Studio、Rider 或 VSCode。
遗憾的是，目前没有干净的方式重写断点堆栈信息，因此 IDE 可能会显示来自 `Debugger.Break` 位置的内部代码。

### 日志消息

启用后，断点会向 [Logs](/tools/developer-tools/logs-tool) 工具写入一条日志消息。

![Log output from triggered breakpoints](/img/tools/dev-tools/breakpoints-logs-ouput.png)

### 命中后移除

顾名思义，断点在命中一次后就会被移除。它可以与其他选项组合使用。

## 另请参阅

- [Events 工具](/tools/developer-tools/events-tool)
- [Logs 工具](/tools/developer-tools/logs-tool)
