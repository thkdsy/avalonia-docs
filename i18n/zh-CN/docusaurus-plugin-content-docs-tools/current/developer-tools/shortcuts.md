---
id: shortcuts
title: Developer Tools 快捷键
sidebar_label: 快捷键
description: Avalonia Developer Tools 的键盘快捷键参考，涵盖检查、搜索、导航、布局和工具切换命令。
doc-type: reference
---

本页列出了 Avalonia Developer Tools 中可用的所有键盘快捷键。若某个快捷键标记为 **Unassigned**，你可以在 [Developer Tools settings](/tools/developer-tools/settings) 中自行绑定。

## 检查

在调试布局问题、跟踪焦点顺序或测量元素间距时，可以使用这些快捷键。

| 显示名称 | 说明 | 使用场景 | Windows / Linux | macOS |
|---|---|---|---|---|
| Focus Tracking | 高亮应用程序中当前获得焦点的元素。在 Developer Tools 和目标应用中都可使用。 | 当你在调试 Tab 顺序或焦点相关问题时使用，这样可以准确看到哪个控件当前拥有焦点。 | <kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>K</kbd> | <kbd>⌘</kbd> <kbd>⇧</kbd> <kbd>K</kbd> |
| Inspect Element | 通过在应用程序中点击 UI 元素来选择并检查它们。在 Developer Tools 和目标应用中都可使用。 | 当你想快速跳转到元素树中的某个特定控件，而不是手动展开节点时使用。 | <kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>C</kbd> | <kbd>⌘</kbd> <kbd>⇧</kbd> <kbd>C</kbd> |
| Highlight Elements | 切换 UI 元素的实时高亮显示。在 Developer Tools 和目标应用中都可使用。 | 当你需要可视化元素边界和内边距，从而快速发现布局问题时使用。 | <kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>H</kbd> | <kbd>⌘</kbd> <kbd>⇧</kbd> <kbd>H</kbd> |
| Show Overlay Rulers | 显示或隐藏覆盖层中的测量标尺。在 Developer Tools 和目标应用中都可使用。 | 当你需要像素级测量元素之间的距离，或验证对齐是否准确时使用。 | <kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>R</kbd> | <kbd>⌘</kbd> <kbd>⇧</kbd> <kbd>R</kbd> |
| Show Overlay Info | 显示或隐藏覆盖层中的详细信息。在 Developer Tools 和目标应用中都可使用。 | 当你想直接在元素覆盖层上查看 `Width`、`Height` 和 `Margin` 等属性值，而不切换到属性面板时使用。 | <kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>D</kbd> | <kbd>⌘</kbd> <kbd>⇧</kbd> <kbd>D</kbd> |
| Toggle TopMost | 切换 Developer Tools 的置顶模式。在 Developer Tools 和目标应用中都可使用。 | 当你想让 Developer Tools 窗口始终位于应用之上，从而无需反复切换窗口时使用。 | <kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>T</kbd> | <kbd>⌘</kbd> <kbd>⇧</kbd> <kbd>T</kbd> |
| Set Breakpoint | 在属性或事件上设置断点。 | 当你需要在某个特定属性发生变化或某个特定事件触发时暂停执行，以便追踪异常状态变化时使用。 | <kbd>F9</kbd> | <kbd>F9</kbd> |

## 搜索与导航

这些快捷键可以帮助你在 Developer Tools 中查找元素、属性或资源。

| 显示名称 | 说明 | 使用场景 | Windows / Linux | macOS |
|---|---|---|---|---|
| Search Current List | 激活当前工具中的搜索功能。 | 当你需要把很长的元素树或属性列表筛选到目标项时使用。 | <kbd>Ctrl</kbd>+<kbd>F</kbd> | <kbd>⌘</kbd> <kbd>F</kbd> |
| Next Search Result | 跳转到当前视图中的下一个搜索结果。 | 输入搜索词后，用它向前循环浏览匹配结果。 | <kbd>F3</kbd> | <kbd>⌘</kbd> <kbd>G</kbd> |
| Previous Search Result | 跳转到当前视图中的上一个搜索结果。 | 用它向后循环浏览匹配结果。 | <kbd>Shift</kbd>+<kbd>F3</kbd> | <kbd>⌘</kbd> <kbd>⇧</kbd> <kbd>G</kbd> |
| Next Tool | 切换到下一个 developer tool 标签页。 | 当你想在多个工具之间切换，例如从 Elements 切到 Events，而不使用鼠标时使用。 | <kbd>Ctrl</kbd>+<kbd>]</kbd> | <kbd>⌘</kbd> <kbd>]</kbd> |
| Previous Tool | 切换到上一个 developer tool 标签页。 | 当你想回到刚刚使用过的工具时使用。 | <kbd>Ctrl</kbd>+<kbd>[</kbd> | <kbd>⌘</kbd> <kbd>[</kbd> |

## 布局与视图

这些快捷键用于控制 Developer Tools 窗口布局和面板可见性。

| 显示名称 | 说明 | 使用场景 | Windows / Linux | macOS |
|---|---|---|---|---|
| Refresh Current View | 刷新当前视图。 | 当你修改代码后，需要重新加载元素树或资源列表时使用。 | <kbd>F5</kbd> | <kbd>⌘</kbd> <kbd>⇧</kbd> <kbd>R</kbd> |
| Remove Item | 从当前列表或视图中移除选中项。 | 当你要移除一个不再需要的断点或日志项时使用。 | <kbd>Delete</kbd> | <kbd>Delete</kbd> |
| Clear Current List | 清空当前列表或视图中的所有项。 | 当你想重置 Events 或 Logs 工具，以便重新开始一次干净的采集会话时使用。 | <kbd>Ctrl</kbd>+<kbd>L</kbd> | <kbd>⌘</kbd> <kbd>L</kbd> |
| Show Navigation | 显示或隐藏导航面板。 | 当你不需要侧边栏，希望最大化主内容区域时使用。 | <kbd>Alt</kbd>+<kbd>1</kbd> | <kbd>⌥</kbd> <kbd>1</kbd> |
| Show Tools | 显示或隐藏工具面板。 | 当你想切换底部工具面板，以获得更多垂直空间时使用。 | <kbd>Alt</kbd>+<kbd>2</kbd> | <kbd>⌥</kbd> <kbd>2</kbd> |
| Reset Layout | 将 Developer Tools 窗口布局重置为默认状态。 | 当你重新排布过面板后，希望恢复原始布局时使用。 | Unassigned | Unassigned |

## 工具

这些快捷键可以直接打开特定的 Developer Tools 面板。

| 显示名称 | 说明 | 使用场景 | Windows / Linux | macOS |
|---|---|---|---|---|
| Open Elements | 打开 Elements 检查工具。 | 当你要查看并导航应用程序的可视树时使用。 | Unassigned | Unassigned |
| Open Assets | 打开 Assets 浏览器。 | 当你要浏览打包到应用中的图片、字体等嵌入资源时使用。 | Unassigned | Unassigned |
| Open Resources | 打开 Resources 浏览器。 | 当你要在运行时检查 `StaticResource` 和 `DynamicResource` 的值时使用。 | Unassigned | Unassigned |
| Open Settings | 打开 Developer Tools 设置。 | 当你要配置主题、按键绑定和连接选项时使用。 | Unassigned | <kbd>⌘</kbd> <kbd>.</kbd> |
| Open Logs | 打开应用日志查看器。 | 当你要查看绑定错误、布局警告和其他诊断消息时使用。 | Unassigned | Unassigned |
| Open Events | 打开事件监视工具。 | 当你要观察路由事件如何在可视树中隧道和冒泡时使用。 | Unassigned | Unassigned |
| Open Breakpoints | 打开断点管理工具。 | 当你要在一个地方查看、启用、禁用或移除所有属性和事件断点时使用。 | Unassigned | Unassigned |
| Open Metrics | 打开性能指标查看器。 | 当你在分析应用性能时，要监控帧率、渲染时间和其他性能计数器时使用。 | Unassigned | Unassigned |
| Open Protocol | 打开 Developer Tools 协议监视工具。 | 当你要检查 Developer Tools 与应用之间交换的底层消息时使用。 | Unassigned | Unassigned |
| Open Documentation | 打开 Developer Tools 文档。 | 当你想快速访问在线文档而不离开 Developer Tools 时使用。 | <kbd>Ctrl</kbd>+<kbd>F1</kbd> | <kbd>⌘</kbd> <kbd>?</kbd> |

## 另请参阅

- [Developer Tools 设置](/tools/developer-tools/settings)
- [Elements 工具](/tools/developer-tools/elements-tool)
- [Events 工具](/tools/developer-tools/events-tool)
- [Breakpoints 工具](/tools/developer-tools/breakpoints-tool)
- [Logs 工具](/tools/developer-tools/logs-tool)
- [Metrics 工具](/tools/developer-tools/metrics-tool)
- [Assets 工具](/tools/developer-tools/assets-tool)
- [Resources 工具](/tools/developer-tools/resources-tool)
