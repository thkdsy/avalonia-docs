---
id: insets-manager
title: Insets 管理器
description: "在 Avalonia 中管理移动端和浏览器平台上的系统栏可见性、安全区域内边距和沉浸式显示。"
doc-type: guide
---

`InsetsManager` 允许你与平台系统栏交互，并处理移动端窗口安全区域的变化。

你可以通过 `TopLevel` 或 `Window` 的实例访问 `InsetsManager`。有关如何访问 `TopLevel` 的更多细节，请参阅 [TopLevel](/docs/fundamentals/top-level) 页面。

```csharp
var insetsManager = TopLevel.GetTopLevel(control).InsetsManager;
```

:::note
此服务当前实现于移动端和浏览器后端。若要自定义桌面端窗口装饰，请使用 `Window.ExtendClientAreaToDecorationsHint` 配合 `WindowDrawnDecorations`。详见 [Window Management](/docs/app-development/window-management#custom-title-bar)。
:::

:::note
从 Avalonia 11.1 开始，Avalonia 应用会根据 inset 值自动调整根视图。你可以在根视图上设置附加属性 `TopLevel.AutoSafeAreaPadding="False"` 来禁用该行为。
:::

## 属性

### IsSystemBarVisible
获取或设置一个值，表示系统栏是否可见。如果平台不支持显示或隐藏系统栏，则返回 `null`。

```csharp
bool? IsSystemBarVisible { get; set; }
```

### DisplayEdgeToEdge
获取或设置一个值，表示窗口是否应以沉浸式方式绘制到可见系统栏之后。

```csharp
bool DisplayEdgeToEdge { get; set; }
```

### SafeAreaPadding
获取当前安全区域内边距。安全区域表示窗口中未被系统栏遮挡的部分。

```csharp
Thickness SafeAreaPadding { get; }
```

### SystemBarColor
获取或设置平台系统栏的颜色。如果平台不支持设置系统栏颜色，则返回 `null`。

```csharp
Color? SystemBarColor { get; set; }
```

## 事件

### SafeAreaChanged
当当前窗口的安全区域发生变化时触发。这可能发生在系统栏显示或隐藏时，或者窗口尺寸、方向发生变化时。

```csharp
event EventHandler<SafeAreaChangedArgs>? SafeAreaChanged;
```

#### SafeAreaChangedArgs

`SafeAreaChangedArgs` 是一个为 `SafeAreaChanged` 事件提供数据的类。

#### SafeAreaPadding
获取新的安全区域内边距。

```csharp
public Thickness SafeAreaPadding { get; }
```

## SystemBarTheme

`SystemBarTheme` 是一个枚举，用于表示系统栏的浅色和深色主题。

### Light
系统栏使用浅色背景和深色前景。

### Dark
系统栏使用深色背景和浅色前景。


## 平台兼容性

| Feature        | Windows | macOS | Linux | Browser | Android |  iOS |
|---------------|-------|-------|-------|-------|-------|-------|
| `IsSystemBarVisible` | ✗ | ✗ | ✗ | ✓* | ✓ | ✓ |
| `DisplayEdgeToEdge` | ✗ | ✗ | ✗ | ✗  | ✓ | ✓ |
| `SafeAreaPadding` | ✗ | ✗ | ✗ | ✓* | ✓ | ✓ |
| `SystemBarColor` | ✗ | ✗ | ✗ | ✗ | ✓ | ✗ |
| `SafeAreaChanged` | ✗ | ✗ | ✗ | ✓* | ✓ | ✓ |

\* - 只有移动端 Chromium 浏览器支持 `IInsetsManager` API。

## 另请参阅

- [Input Pane](/docs/services/input-pane)：软件键盘状态与边界信息。
- [TopLevel](/docs/fundamentals/top-level)：从控件访问平台服务。
