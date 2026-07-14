---
id: settings
title: Developer Tools 设置
sidebar_label: 设置
doc-type: reference
---

可以从 **Tray Icon** 菜单（Windows 和 Linux）或 **macOS 全局菜单** 进入设置页面。
另外，当某个应用已经连接后，也可以通过**左侧导航栏**打开设置页面。

| 分类 | 设置项 | 说明 | 默认值 |
|----------|---------|-------------|---------------|
| **Appearance** |
| | Theme Variant | 控制应用程序的颜色主题 | Dark |
| | Exit On Last Window Close | 决定在最后一个窗口关闭时应用程序是否退出 | true |
| | Skip Welcome Window | 启动时跳过欢迎页 | false |
| | Enable Protocol Monitor | 显示诊断通信协议监视窗口 | false |
| **Elements Tree** |
| | Aggregate Templates | 将模板中的可视子元素合并为单个树节点，以获得更简洁的可视化效果，默认折叠 | true |
| | InlinePseudoclasses | 默认情况下，只有可见元素的伪类会在树中右对齐显示，其余会隐藏到覆盖按钮中。此设置允许无论是否可见，都将所有伪类直接内联显示。 | false |
| | Contextual Properties | 仅显示与当前上下文/所选元素状态相关的属性。例如，`Grid.Row` 属性只会在 `Grid` 的直接子元素上显示 | true |
| | Include CLR Properties | 除 Avalonia 特定属性外，同时显示 .NET CLR 属性，并排除重复项 | false |
| **Overlay** |
| | Show ToolTip Info | 在悬停元素上显示包含控件信息的工具提示 | true |
| | Visualize Margin & Padding | 高亮悬停元素的 margin、padding 和 border 区域 | true |
| | Show Rulers | 显示测量标尺，以便精确定位元素 | true |
| | Show Extension Lines | 显示悬停元素与标尺之间的辅助线 | true |
| **Events** |
| | Default Routed Events | Events 工具默认追踪的事件列表 | `Button.ClickEvent`, `InputElement.KeyDownEvent`, `InputElement.KeyUpEvent`, `InputElement.TextInputEvent`, `InputElement.PointerReleasedEvent`, `InputElement.PointerPressedEvent` |
| **Metrics** |
| | Observable meters polling interval (ms) | 轮询 observable meters 以获取新值的频率。较低的值会带来更频繁的更新，但可能影响已连接应用的性能。 | 1000ms |
| | Measurements frame interval (ms) | 采集并更新指标可视化数据的频率 | 250ms |
| | Aggregate frame measurements | 启用后，会合并同一帧中的多个测量值：对基于时间的指标取平均值，对计数器和 gauge 使用最新值。禁用后，会保留所有原始测量值。 | true |
| | Measurements history duration (s) | 定义保留并显示历史测量值的时长 | 60s |
| **Protocol** |
| | HTTP port | 定义监听应用连接所使用的 HTTP 端口。修改后需要重启。该值需要与目标应用中的 `DeveloperToolsOptions.Protocol` 设置保持一致。 | 29414 |

## 另请参阅

- [Developer tools 选项](/tools/developer-tools/options)
- [Developer tools 快捷键](/tools/developer-tools/shortcuts)
