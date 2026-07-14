---
id: viewbox
title: Viewbox
---

import ViewboxScaleUniformBothScreenshot from '/img/controls/viewbox/viewbox-scale-uniform-both.gif';
import ViewboxScaleUniformFillBothScreenshot from '/img/controls/viewbox/viewbox-scale-uniformtofill-both.gif';
import ViewboxScaleFillBothScreenshot from '/img/controls/viewbox/viewbox-scale-fill-both.gif';
import ViewboxScaleNoneBothScreenshot from '/img/controls/viewbox/viewbox-scale-none-both.gif';
import ViewboxScaleUniformDownOnlyScreenshot from '/img/controls/viewbox/viewbox-uniform-downonly.gif';
import ViewboxScaleUniformUpOnlyScreenshot from '/img/controls/viewbox/viewbox-uniform-uponly.gif';

`Viewbox` 是一个可以缩放其内容的容器控件。你可以定义内容的拉伸方式，以及拉伸发生的时机（拉伸方向）。

## 常用属性

你最常使用的通常是这些属性：

| 属性 | 默认值 | 说明 |
| ------------------ | ------- |--------------------------------------------------------------|
| `Stretch` | Uniform | 决定内容如何适应可用空间。 |
| `StretchDirection` | Both | 决定何时发生缩放。 |

`Stretch` 属性的取值如下：

<table><thead><tr><th width="250">Stretch</th><th>说明</th></tr></thead><tbody><tr><td><code>Uniform</code></td><td>（默认）调整内容大小以适应容器尺寸，同时保持其原始宽高比。</td></tr><tr><td><code>Fill</code></td><td>调整内容大小以填满容器尺寸，不保持宽高比。</td></tr><tr><td><code>UniformToFill</code></td><td>在保持原始宽高比的同时，将内容缩放到完全填满容器。但如果内容的宽高比与分配空间的宽高比不一致，部分内容可能会被隐藏。</td></tr></tbody></table>

`StretchDirection` 属性的取值如下：

| 拉伸方向 | 说明 |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| `UpOnly` | 仅当内容小于可用空间时向上缩放；如果内容更大，则不会向下缩放。 |
| `DownOnly` | 仅当内容大于可用空间时向下缩放；如果内容更小，则不会向上缩放。 |
| `Both` | （默认）始终根据拉伸模式进行缩放以适应可用空间。 |

### 示例

这个简单示例展示了 `Viewbox` 如何以统一方式放大一个圆形（拉伸模式和拉伸方向都使用默认值）。

```xml
<Viewbox Stretch="Uniform" Width="150" Height="150">
   <Ellipse Width="50" Height="50" Fill="CornflowerBlue" />  
</Viewbox>
```

### 演示

下面的演示展示了 `Stretch` 和 `StretchDirection` 不同组合的效果。第一组展示的是 `Stretch` 属性的效果：

<table><thead><tr><th width="275">Stretch 值</th><th>演示</th></tr></thead><tbody><tr><td><code>Uniform</code></td><td><Image light={ViewboxScaleUniformBothScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/></td></tr><tr><td><code>UniformToFill</code></td><td><Image light={ViewboxScaleUniformFillBothScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/></td></tr><tr><td><code>Fill</code></td><td><Image light={ViewboxScaleFillBothScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/></td></tr><tr><td><code>None</code></td><td><Image light={ViewboxScaleNoneBothScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/></td></tr></tbody></table>

下面这组演示展示的是 `StretchDirection` 属性的效果：

<table><thead><tr><th width="276">StretchDirection</th><th>演示</th></tr></thead><tbody><tr><td><code>UpOnly</code></td><td><Image light={ViewboxScaleUniformUpOnlyScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/></td></tr><tr><td><code>DownOnly</code></td><td><Image light={ViewboxScaleUniformDownOnlyScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/></td></tr></tbody></table>

## 另请参阅

- [Viewbox API reference](/api/avalonia/controls/viewbox)
- [`Viewbox.cs` source code on GitHub](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/Viewbox.cs)
