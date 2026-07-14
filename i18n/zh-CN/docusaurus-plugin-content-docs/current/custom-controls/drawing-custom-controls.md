---
id: drawing-custom-controls
title: 绘制自定义控件
description: 通过重写 Render 方法并使用 DrawingContext 操作来渲染自定义视觉内容。
doc-type: how-to
---

import DrawWithPropertyScreenshot from '/img/guides/ui-development/custom-controls/draw-property.png';

本页将展示如何绘制一个自定义控件，并使用一个简单属性的值来定义背景颜色。代码如下：

```xml title='MainWindow.xaml'
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:cc="using:AvaloniaCCExample.CustomControls"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        mc:Ignorable="d" d:DesignWidth="800" d:DesignHeight="450"
        x:Class="AvaloniaCCExample.MainWindow"
        Title="Avalonia Custom Control">
  <cc:MyCustomControl Height="200" Width="300" Background="Red"/>
</Window>

```

```csharp title='MyCustomControl.cs'
using Avalonia.Controls;

namespace AvaloniaCCExample.CustomControls
{
    public class MyCustomControl : Control
    {
        public IBrush? Background { get; set; }

        public sealed override void Render(DrawingContext context)
        {
            if (Background != null)
            {
                var renderSize = Bounds.Size;
                context.FillRectangle(Background, new Rect(renderSize));
            }
            
            base.Render(context);
        }
    }
}
```

该示例在自定义控件上定义了一个简单的画刷属性，用于表示背景颜色。随后它通过重写 `Render` 方法来绘制控件。

绘制代码使用传入 `Render` 方法的 Avalonia 图形上下文，绘制了一个填充背景色的矩形，并让这个矩形与控件本身大小一致（大小由 `Bounds.Size` 提供）。

<Image light={DrawWithPropertyScreenshot} alt="Custom-drawn control rendered with a bound property value" position="center" maxWidth={400} cornerRadius="true"/>

注意，这个控件现在既可以在运行时显示（如上图），也能在预览窗格中显示。

下一页将介绍如何实现这个背景属性，使它能够被 Avalonia 样式系统修改。

:::tip
你可以在 [Avalonia.Samples](
https://github.com/AvaloniaUI/Avalonia.Samples/tree/main/src/Avalonia.Samples/CustomControls/SnowflakesControlSample)
 中找到一个更高级的教程。
:::

## DrawingContext 方法

`DrawingContext` 提供了多种方法，可用于在自定义控件中渲染内容：

| 方法 | 说明 |
|---|---|
| `DrawRectangle` | 绘制一个可选填充和描边的矩形 |
| `DrawEllipse` | 绘制一个椭圆 |
| `DrawLine` | 在两个点之间绘制一条线 |
| `DrawGeometry` | 绘制任意几何路径 |
| `DrawText` | 绘制格式化文本 |
| `DrawImage` | 绘制位图图像 |
| `FillRectangle` | 填充一个矩形（简写方式） |

## 绘制图形

下面的示例演示了如何在单个 `Render` 重写中绘制多个图形：

```csharp
public override void Render(DrawingContext context)
{
    var pen = new Pen(Brushes.Black, 2);

    // 绘制一个填充矩形
    context.DrawRectangle(Brushes.LightBlue, pen, new Rect(10, 10, 100, 60));

    // 绘制一个椭圆
    context.DrawEllipse(Brushes.Orange, pen, new Point(200, 40), 50, 30);

    // 绘制一条线
    context.DrawLine(new Pen(Brushes.Red, 3), new Point(10, 100), new Point(290, 100));
}
```

## 绘制文本

你可以将 `DrawText` 与 `FormattedText` 对象结合使用来绘制格式化文本：

```csharp
public override void Render(DrawingContext context)
{
    var text = new FormattedText(
        "Hello, Avalonia!",
        CultureInfo.CurrentCulture,
        FlowDirection.LeftToRight,
        new Typeface("Arial"),
        24,
        Brushes.Black);

    context.DrawText(text, new Point(10, 10));
}
```

## 使用裁剪与变换

`DrawingContext` 支持裁剪区域和变换。`PushClip` 与 `PushTransform` 都会返回可释放对象，因此将它们包裹在 `using` 代码块中，可以确保状态被自动恢复：

```csharp
public override void Render(DrawingContext context)
{
    // 使用 PushClip 保存并恢复状态
    using (context.PushClip(new Rect(0, 0, 100, 100)))
    {
        context.FillRectangle(Brushes.Blue, new Rect(0, 0, 200, 200));
        // 只有 100x100 范围内的部分可见
    }

    // 应用一个变换
    using (context.PushTransform(Matrix.CreateRotation(Math.PI / 4)))
    {
        context.FillRectangle(Brushes.Green, new Rect(150, 50, 40, 40));
    }
}
```

## 使视觉失效并重绘

若希望在属性变化时触发重新渲染，可以在静态构造函数中使用 `AffectsRender` 注册该属性，或者手动调用 `InvalidateVisual()`：

```csharp
static MyCustomControl()
{
    AffectsRender<MyCustomControl>(BackgroundProperty);
}
```

在控件实例上调用 `InvalidateVisual()` 会将其标记为需要重绘。随后 Avalonia 会在下一次布局过程中再次调用 `Render`。

## 另请参阅

- [自定义渲染](/docs/graphics-animation/custom-rendering)：高级渲染技术。
- [画刷](/docs/graphics-animation/brushes)：可用的画刷类型。
- [形状与几何图形](/docs/graphics-animation/shapes-and-geometries)：用于绘制的几何类型。
