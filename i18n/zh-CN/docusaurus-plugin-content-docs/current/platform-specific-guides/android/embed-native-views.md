---
id: embed-native-views
title: 嵌入 Android 原生视图
description: 了解如何使用 NativeControlHost 和 AndroidViewControlHandle 在 Avalonia 应用中嵌入 WebView、Button 等 Android 原生视图。
doc-type: guide
---

Avalonia 允许你通过继承 [`NativeControlHost`](/api/avalonia/controls/nativecontrolhost)，把 Android 原生视图嵌入到 Avalonia 视觉树中。你需要把每个 Android `View` 包装成 [`AndroidViewControlHandle`](/api/avalonia/android/androidviewcontrolhandle)，并从 `CreateNativeControlCore` 返回它。当你需要使用 Avalonia 没有对应实现的平台特定控件（例如 `WebView`、`MapView` 或媒体播放器）时，这会非常有用。

## 工作原理

`NativeControlHost` 会在 Avalonia 布局中预留一块区域，并把该区域的渲染交给原生平台。在 Android 上，你需要：

1. 在 `NativeControlHost` 子类中重写 `CreateNativeControlCore`。
2. 使用 `parent` 句柄提供的父级上下文创建所需的 Android 原生 `View`。
3. 将该 `View` 包装为 `AndroidViewControlHandle` 并返回。

Avalonia 会负责定位并裁剪原生视图，使其与宿主控件的边界保持一致。

## 获取父级上下文

传给 `CreateNativeControlCore` 的 `parent` 参数是一个 `IPlatformHandle`。在 Android 上，你可以把它转换为 `AndroidViewControlHandle`，从而获取 `View.Context`。如果转换失败，则退回到全局应用上下文：

```csharp
var parentContext = (parent as AndroidViewControlHandle)?.View.Context
    ?? global::Android.App.Application.Context;
```

## 示例：嵌入 WebView 和 Button

下面的示例演示了一个实现 `INativeDemoControl` 接口的类，它会根据参数创建两种不同的 Android 原生控件之一。

:::tip
这个示例基于 Avalonia 仓库中的 [ControlCatalog.Android 示例](https://github.com/AvaloniaUI/Avalonia/blob/master/samples/ControlCatalog.Android/EmbedSample.Android.cs)。
:::

首先，定义共享代码用于请求原生控件的接口：

```csharp
public interface INativeDemoControl
{
    IPlatformHandle CreateControl(
        bool isSecond,
        IPlatformHandle parent,
        Func<IPlatformHandle> createDefault);
}
```

然后在 Android 项目中实现它：

```csharp
public class EmbedSampleAndroid : INativeDemoControl
{
    public IPlatformHandle CreateControl(
        bool isSecond,
        IPlatformHandle parent,
        Func<IPlatformHandle> createDefault)
    {
        var parentContext = (parent as AndroidViewControlHandle)?.View.Context
            ?? global::Android.App.Application.Context;

        if (isSecond)
        {
            var webView = new global::Android.Webkit.WebView(parentContext);
            webView.LoadUrl("https://www.android.com/");
            return new AndroidViewControlHandle(webView);
        }

        var button = new global::Android.Widget.Button(parentContext)
        {
            Text = "Hello world"
        };

        var clickCount = 0;
        button.Click += (sender, args) =>
        {
            clickCount++;
            button.Text = $"Click count {clickCount}";
        };

        return new AndroidViewControlHandle(button);
    }
}
```

当 `isSecond` 为 `true` 时，该方法会创建一个 Android `WebView`、加载一个 URL，并将其包装为 `AndroidViewControlHandle` 后返回。当 `isSecond` 为 `false` 时，它会创建一个带点击计数器的原生 `Button` 并返回。

## 限制

原生视图会位于 Avalonia 渲染表面的上层。请注意以下限制：

- **不支持透明**：原生视图不能使用透明背景来显示其后的 Avalonia 内容。
- **不支持变换**：Avalonia 的渲染变换（旋转、缩放）不会影响原生视图。
- **Z 轴顺序受限**：原生视图始终渲染在 Avalonia 内容之上。你不能把 Avalonia 控件叠加到原生视图之上。
- **裁剪有限**：原生视图会被裁剪到宿主边界内，但不支持复杂裁剪几何体。

## 另请参阅

- [原生平台互操作](/docs/app-development/native-interop)
- [使用 Avalonia 开发 Android 应用](/docs/platform-specific-guides/android)
- [部署到 Android](/docs/deployment/android)
