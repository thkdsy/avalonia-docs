---
id: top-level
title: TopLevel 顶层对象
description: 通过 TopLevel 基类访问窗口、剪贴板、存储及其他平台服务。
doc-type: reference
---

[`TopLevel`](/api/avalonia/controls/toplevel) 充当视觉根节点，也是所有顶层控件的基类，例如 [`Window`](/api/avalonia/controls/window)。它负责调度布局、样式和渲染，同时还会跟踪客户端区域大小。大多数服务都通过 `TopLevel` 来访问。

## 获取 TopLevel

下面是两种常见的获取 `TopLevel` 实例的方法。

### 使用 TopLevel.GetTopLevel

你可以使用 `TopLevel` 类的静态方法 `GetTopLevel`，来获取包含当前控件的顶层控件。

```csharp
var topLevel = TopLevel.GetTopLevel(control);
// 在这里，你可以通过 topLevel 实例访问 Clipboard、StorageProvider 等各种服务。
```

如果你当前位于用户控件或更底层的组件中，并且需要访问 TopLevel 提供的服务，那么这种方法会很有帮助。

:::note
如果 `TopLevel.GetTopLevel` 返回 null，通常说明该控件尚未附加到根节点上。为了确保控件已附加，你可以处理 `Control.Loaded` 和 `Control.Unloaded` 事件，并在这些事件中维护当前顶层对象的引用。
:::

### 使用 Window 类

由于 `Window` 类继承自 `TopLevel`，因此你可以直接通过 `Window` 实例来访问这些服务：

```csharp
var topLevel = window;
```

当你本身已经处于某个窗口上下文中时，例如在 `Window` 类的事件处理器中，或者在与窗口强相关的逻辑里，这种方式最为直接。

## 常用属性

### ActualTransparencyLevel

获取平台实际能够提供的 `WindowTransparencyLevel`。

```csharp
WindowTransparencyLevel ActualTransparencyLevel { get; }
```

### ClientSize

获取窗口客户端区域的大小。

```csharp
Size ClientSize { get; }
```

### Clipboard

获取平台的 [Clipboard](/docs/services/clipboard) 实现。

```csharp
IClipboard? Clipboard { get; }
```

### FocusManager

获取根对象对应的 [焦点管理器](/docs/services/focus-manager)。

```csharp
IFocusManager? FocusManager { get; }
```

### FrameSize

获取顶层对象的总尺寸，如果存在系统边框，也会包含在内。

```csharp
Size? FrameSize { get; }
```

### InsetsManager

获取平台的 [InsetsManager](/docs/services/insets-manager) 实现。

```csharp
IInsetsManager? InsetsManager { get; }
```

### PlatformSettings

表示一个用于访问顶层 [平台特定设置](/docs/services/platform-settings) 的契约接口。

```csharp
IPlatformSettings? PlatformSettings { get; }
```

### RendererDiagnostics

获取一个值，用于指示渲染器是否应绘制特定的诊断信息。

```csharp
RendererDiagnostics RendererDiagnostics { get; }
```

### RenderScaling

获取渲染时使用的缩放因子。

```csharp
double RenderScaling { get; }
```

### Screens

获取 [`Screens`](/api/avalonia/controls/screens) 实例，它提供有关已连接显示器的信息，包括分辨率、工作区、缩放和方向。

```csharp
Screens Screens { get; }
```

你可以使用 `Screens` 查询主显示器、枚举所有显示器，或查找包含某个窗口或坐标点的显示器。用法示例请参阅 [Working with screens](/docs/app-development/window-management#working-with-screens)。

### RequestedThemeVariant

获取或设置该控件（以及它的子元素）在资源解析时所使用的 UI 主题变体。你通过 `ThemeVariant` 指定的主题可以覆盖应用级别的 `ThemeVariant`。

```csharp
ThemeVariant? RequestedThemeVariant { get; set; }
```

### StorageProvider

用于文件选择器和书签功能的[文件系统存储](/docs/services/storage/storage-provider)服务。

```csharp
IStorageProvider StorageProvider { get; }
```

### TransparencyBackgroundFallback

获取或设置当平台不支持透明效果时，透明区域用于混合的 `IBrush`。默认值是纯白色画刷。

```csharp
IBrush TransparencyBackgroundFallback { get; set; }
```

### TransparencyLevelHint

获取或设置 TopLevel 在可能情况下应使用的 `WindowTransparencyLevel`。它可以接受多个值，并按回退顺序依次尝试。例如，当设置为 `"Mica, Blur"` 时，如果平台支持 Mica 就优先使用，否则回退到 Blur。默认值为空数组或 `"None"`。

```csharp
IReadOnlyList<WindowTransparencyLevel> TransparencyLevelHint { get; set; }
```

## 常用事件

### BackRequested

当实体返回按钮被按下，或系统请求执行返回导航时触发。

```csharp
event EventHandler<RoutedEventArgs> BackRequested { add; remove; }
```

### Closed

当窗口关闭时触发。

```csharp
event EventHandler Closed;
```

### Opened

当窗口打开时触发。

```csharp
event EventHandler Opened;
```

### ScalingChanged

当 TopLevel 的缩放比例发生变化时触发。

```csharp
event EventHandler ScalingChanged;
```

## 常用方法

### GetTopLevel

获取承载指定 `Visual` 的 `TopLevel`。

#### 参数

`control`
要查询其 TopLevel 的视觉对象

```csharp
static TopLevel? GetTopLevel(Visual? visual)
```

### RequestAnimationFrame

将一个回调加入队列，使其在下一次动画时钟到来时执行。该回调会运行在 UI 线程上，并与 Avalonia 的渲染周期保持同步。每次调用只会安排执行一次；如果你要创建持续的动画循环，就需要在回调内部再次调用 `RequestAnimationFrame`。

```csharp
void RequestAnimationFrame(Action<TimeSpan> action)
```

#### 参数

`action`
一个回调，它会接收一个 `TimeSpan` 参数，表示自动画系统启动以来经过的时间。你可以用它来计算与帧率无关的动画进度。

#### 示例：持续动画循环

```csharp
var topLevel = TopLevel.GetTopLevel(this);

topLevel.RequestAnimationFrame(OnAnimationFrame);

private void OnAnimationFrame(TimeSpan elapsed)
{
    // Update your visual state based on elapsed time
    _angle = elapsed.TotalSeconds * 90; // 90 degrees per second
    InvalidateVisual();

    // Schedule the next frame to keep the loop running
    TopLevel.GetTopLevel(this)?.RequestAnimationFrame(OnAnimationFrame);
}
```

This is the Avalonia equivalent of WPF's `CompositionTarget.Rendering`. For render-thread callbacks that do not block the UI thread, see [CompositionCustomVisualHandler](/docs/graphics-animation/custom-rendering#compositioncustomvisualhandler).

### RequestPlatformInhibition

Requests a `PlatformInhibitionType` to be inhibited. The behavior remains inhibited until the return value is disposed. The available set of `PlatformInhibitionType`s depends on the platform. If a behavior is inhibited on a platform where this type is not supported the request will have no effect.

```csharp
async Task<IDisposable> RequestPlatformInhibition(PlatformInhibitionType type, string reason)
```

### TryGetPlatformHandle

Tries to get the platform handle for the TopLevel-derived control.

```csharp
IPlatformHandle? TryGetPlatformHandle()
```

## See also

- [Main window](/docs/fundamentals/main-window)
- [Application lifetimes](/docs/fundamentals/application-lifetimes)
- [Working with screens](/docs/app-development/window-management#working-with-screens): Query monitor resolution, bounds, and scaling.
- [Custom rendering](/docs/graphics-animation/custom-rendering): Custom drawing and render-thread callbacks.
- [Composition animations](/docs/graphics-animation/composition-animations): Render-thread property animations.
- [`TopLevel.cs` source code on GitHub](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/TopLevel.cs)
