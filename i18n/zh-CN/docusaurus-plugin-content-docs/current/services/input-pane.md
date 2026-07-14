---
id: input-pane
title: 输入面板
description: "在 Avalonia 应用中监视平台输入面板（软件键盘）的状态、边界和动画。"
doc-type: guide
---

`InputPane` 允许开发者监听平台输入面板（例如软件键盘或屏幕键盘）的当前状态和边界。

你可以通过 `TopLevel` 或 `Window` 的实例访问 `InputPane`。有关如何访问 `TopLevel` 的更多细节，请参阅 [TopLevel](/docs/fundamentals/top-level) 页面。

```csharp
var inputPane = TopLevel.GetTopLevel(control).InputPane;
```

:::note
目前，Avalonia 不会根据输入面板状态自动调整根视图和滚动位置。更推荐开发者使用 `IInputPane` API，并在应用中自行进行适配。

自动调整能力计划在后续 11.* 版本中加入。
:::

## 属性

### State
当前输入面板的状态。
可能的值包括：
- `InputPaneState.Closed`
- `InputPaneState.Opened`

```csharp
InputPaneState State { get; }
```

### OccludedRect
当前输入面板所占据的边界区域。

```csharp
Rect OccludedRect { get; }
```

:::note
返回值使用相对于当前顶级窗口的客户区坐标。
如果输入面板是浮动式或分离式，且显示在视图上方，则会返回空矩形。
:::

## 事件

### StateChanged
当输入面板状态发生变化时触发。

```csharp
event EventHandler<InputPaneStateEventArgs>? StateChanged;
```

值得注意的是，事件参数中包含多个很有用的值：
- `InputPaneStateEventArgs.NewState` - 输入面板的新状态。
- `InputPaneStateEventArgs.StartRect` - 输入面板的起始边界。
- `InputPaneStateEventArgs.EndRect` - 输入面板的最终边界。
- `InputPaneStateEventArgs.AnimationDuration` - 输入面板状态变化动画的持续时间。
- `InputPaneStateEventArgs.Easing` - 输入面板状态变化动画使用的缓动函数。

借助 `AnimationDuration` 和 `Easing`，开发者可以在两种状态之间创建平滑过渡。

## 平台兼容性

| Feature        | Windows | macOS | Linux | Browser | Android |  iOS |
|---------------|-------|-------|-------|-------|-------|-------|
| `State` | ✓ | ✗ | ✗ | ✓* | ✓ | ✓ |
| `OccludedRect` | ✓ | ✗ | ✗ | ✓*  | ✓ | ✓ |
| `StateChanged` | ✓ | ✗ | ✗ | ✓* | ✓ | ✓ |
| `StateChanged.StartRect` | ✗ | ✗ | ✗ | ✓* | ✓ | ✓ |
| `StateChanged.AnimationDuration` | ✗ | ✗ | ✗ | ✗ | ✓ | ✓ |
| `StateChanged.Easing` | ✗ | ✗ | ✗ | ✗ | ✓ | ✓ |

\* - 只有移动端 Chromium 浏览器支持 `IInputPane` API。

## 另请参阅

- [Insets Manager](/docs/services/insets-manager)：系统栏可见性与安全区域管理。
- [TopLevel](/docs/fundamentals/top-level)：从控件访问平台服务。
