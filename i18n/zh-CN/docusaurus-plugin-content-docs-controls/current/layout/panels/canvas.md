---
id: canvas
title: Canvas
description: 了解如何在 Avalonia 中使用 Canvas 面板通过绝对坐标定位子控件。
doc-type: reference
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import CanvasContentZoneScreenshot from '/img/controls/canvas/canvas-contentzone.png';

Canvas 控件会将其子控件显示在指定位置（通过坐标给出）。

每个子控件的位置由两段距离来定义：从画布内容区边缘到子控件外部 margin 区边缘的距离。例如，这可以表示子控件左上角到画布左上角的距离，如下图所示：

<Image light={CanvasContentZoneScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

如果多个元素占据相同坐标，则它们在标记中出现的顺序决定了它们的绘制顺序。

[`Canvas`](/api/avalonia/controls/canvas) 在所有 `Panel` 中提供了最灵活的布局支持。`Height` 和 `Width` 属性用于定义画布区域，而其中的元素则会相对于父级 `Canvas` 区域被赋予绝对坐标。四个附加属性 `Canvas.Left`、`Canvas.Top`、`Canvas.Right` 和 `Canvas.Bottom` 允许你精细控制对象在 `Canvas` 中的位置，从而实现屏幕上的精确排列。

:::info
若要回顾布局区域的概念，请参阅 [Layout](/docs/layout/)。
:::

## 常用属性

你最常使用的通常是这些属性：

<table><thead><tr><th width="205">属性</th><th>说明</th></tr></thead><tbody><tr><td><code>Canvas.Left</code></td><td>附加在子控件上——表示从画布内容区内部左边缘到子控件外部左边缘（margin 区）的距离。</td></tr><tr><td><code>Canvas.Top</code></td><td>附加在子控件上——表示从画布内容区内部上边缘到子控件外部上边缘（margin 区）的距离。</td></tr><tr><td><code>Canvas.Right</code></td><td>附加在子控件上——表示从画布内容区内部右边缘到子控件外部右边缘（margin 区）的距离。</td></tr><tr><td><code>Canvas.Bottom</code></td><td>附加在子控件上——表示从画布内容区内部下边缘到子控件外部下边缘（margin 区）的距离。</td></tr><tr><td><code>ZIndex</code></td><td>继承自 <code>Visual</code> 的属性，可用于覆盖默认绘制顺序（见下文）。</td></tr></tbody></table>

Canvas 中的子控件会按照它们定义的顺序进行绘制，这可能会导致它们发生重叠。

:::caution
Canvas 不会为其子控件设置尺寸。你必须为子控件设置 width 和 height 属性，否则它将不会显示！
:::

## Z-index

默认情况下，每个子元素的 z-index 都为零。不过，你可以在任意子控件上设置 `ZIndex` 属性。该属性继承自 `Visual`，会覆盖默认绘制顺序（值越大，越晚绘制），从而改变子控件之间的重叠方式。

## 透明度

无论你如何定义绘制顺序，子控件的透明度都会被正确应用。这意味着，当子控件相互重叠时，如果上层控件的透明度小于 1，那么重叠区域显示的内容可能会发生混合。

## ClipToBounds

`Canvas` 可以将子元素定位到屏幕上的任意位置，甚至是超出自身 `Height` 和 `Width` 定义范围之外的坐标。此外，`Canvas` 不受其子元素尺寸影响。因此，子元素有可能会绘制到父级 `Canvas` 边界矩形之外并覆盖其他元素。`Canvas` 的默认行为是允许子元素绘制到其边界之外。如果你不希望如此，可以将 `ClipToBounds` 属性设置为 `true`。这样 `Canvas` 就会裁剪到自身尺寸范围内。`Canvas` 是唯一允许子元素绘制到自身边界之外的布局元素。

## 示例

<XamlPreview>

```xml
<Canvas xmlns="https://github.com/avaloniaui"
        Background="AliceBlue" Margin="20">
  <Rectangle Fill="Red" Height="100" Width="100" Margin="10"/>
  <Rectangle Fill="Blue" Height="100" Width="100" Opacity="0.5"
             Canvas.Left="50" Canvas.Top="20"/>
  <Rectangle Fill="Green" Height="100" Width="100" 
             Canvas.Left="60" Margin="40" Canvas.Top="40"/>
  <Rectangle Fill="Orange" Height="100" Width="100" 
             Canvas.Right="70" Canvas.Bottom="60"/>
</Canvas>
```

</XamlPreview>

:::info
请谨慎使用 Canvas 面板。虽然用这种方式定位子控件很方便，但你的 UI 将不再能够自适应应用窗口尺寸的变化。
:::

## 在代码中定义 Canvas

<Tabs
  defaultValue="xaml"
  values={[
      { label: 'XAML', value: 'xaml', },
      { label: 'C#', value: 'cs', },
  ]}
>
<TabItem value="xaml">

```xml
<Canvas Height="400" Width="400">
  <Canvas Height="100" Width="100" Top="0" Left="0" Background="Red"/>
  <Canvas Height="100" Width="100" Top="100" Left="100" Background="Green"/>
  <Canvas Height="100" Width="100" Top="50" Left="50" Background="Blue"/>
</Canvas>
```

</TabItem>
<TabItem value="cs">

```cs
// 创建 Canvas
myParentCanvas = new Canvas();
myParentCanvas.Width = 400;
myParentCanvas.Height = 400;

// 定义子 Canvas 元素
myCanvas1 = new Canvas();
myCanvas1.Background = Brushes.Red;
myCanvas1.Height = 100;
myCanvas1.Width = 100;
Canvas.SetTop(myCanvas1, 0);
Canvas.SetLeft(myCanvas1, 0);

myCanvas2 = new Canvas();
myCanvas2.Background = Brushes.Green;
myCanvas2.Height = 100;
myCanvas2.Width = 100;
Canvas.SetTop(myCanvas2, 100);
Canvas.SetLeft(myCanvas2, 100);

myCanvas3 = new Canvas();
myCanvas3.Background = Brushes.Blue;
myCanvas3.Height = 100;
myCanvas3.Width = 100;
Canvas.SetTop(myCanvas3, 50);
Canvas.SetLeft(myCanvas3, 50);

// 将子元素添加到 Canvas 的 Children 集合中
myParentCanvas.Children.Add(myCanvas1);
myParentCanvas.Children.Add(myCanvas2);
myParentCanvas.Children.Add(myCanvas3);
```
</TabItem>  

</Tabs>

## 另请参阅

- [Canvas API reference](/api/avalonia/controls/canvas)
- [`Canvas.cs` source code on GitHub](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/Canvas.cs)
- [DockPanel](/controls/layout/panels/dockpanel)
- [Grid](/controls/layout/panels/grid)
- [Panel](/controls/layout/panels/panel)
- [RelativePanel](/controls/layout/panels/relativepanel)
- [StackPanel](/controls/layout/panels/stackpanel)
- [UniformGrid](/controls/layout/panels/uniformgrid)
- [WrapPanel](/controls/layout/panels/wrappanel)
