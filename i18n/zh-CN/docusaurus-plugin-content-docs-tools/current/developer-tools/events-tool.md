---
id: events-tool
title: 事件工具
doc-type: reference
---

Events Tool 为 Avalonia 的路由事件系统提供实时监控与调试能力。Avalonia 中的路由事件采用较为复杂的事件处理机制，事件可以沿着可视树向上或向下传播。该工具可以帮助开发者跟踪事件传播过程、识别事件处理器，并调试应用程序中的事件相关问题。

如需了解更基础的信息，请参阅 Avalonia 文档中的 [Routed Events](https://docs.avaloniaui.net/docs/concepts/input/routed-events)。

![List of Raised Events](/img/tools/dev-tools/events-raised-events-list.png)

## 启用事件监听器

默认情况下，`Button.Click`、`KeyDown`、`KeyUp`、`TextInput`、`PointerReleased` 和 `PointerPressed` 事件处于启用状态。这些默认项可通过 `Default Routed Events` 设置控制；详见 [Developer Tools Settings](/tools/developer-tools/settings) 页面。

你可以通过 **Event Listeners** 弹出按钮来启用或禁用任意特定路由事件或事件组。

![Listeners Filter Flyout](/img/tools/dev-tools/events-listeners-filter.png)

这个列表会在选项卡首次打开时，从静态注册的路由事件中收集而来。
如果某个事件没有显示在该列表中，通常意味着它从未在应用程序中被引用过。

## 浏览事件处理器列表

在 Avalonia 中，路由事件有三种可能的路由策略：
- `Tunnel` 策略会从根节点（通常是窗口）路由到源元素（通常是被点击或获得焦点的元素）。
- `Bubble` 策略则相反，会从源元素一路路由回窗口。
- `Direct` 策略只会在事件直接从源元素上引发时发生，不会路由到其他元素。

`Bubble` 是 XAML 和 C# 事件处理器中默认使用的策略。`Tunnel` 策略通常也被称为 `Preview`，因为它允许在标准 `Bubble` 之前先处理事件。

单个引发的事件可能会经过多个元素处理器，但最终通常只有一个处理器会真正将该事件标记为 handled，从而终止路由。

在 `Developer Tools` 中，这三种策略都通过不同颜色进行区分。
已经处理了该事件的元素会以明显区别于其他元素的方式显示，从而指出事件路由停止的位置。

`Developer Tools` 仍然会显示后续元素处理器，因为它们可能会接收到已经被标记为 handled 的事件参数。

![Raised Event Handlers Chain](/img/tools/dev-tools/events-chain-list.png)

## 检查事件处理器控件

每个元素处理器都可以点击，点击后会将用户跳转到 elements tree 中对应的节点。

注意：如果点击某个元素时没有任何反应，通常说明该元素已经从 elements tree 中移除了。

![Inspect Handler](/img/tools/dev-tools/events-inspect-handler.gif)

## 另请参阅

- [元素工具](/tools/developer-tools/elements-tool)
- [断点工具](/tools/developer-tools/breakpoints-tool)
