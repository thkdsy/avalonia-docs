---
id: image
title: Image
---

import ImageUnscaledScreenshot from '/img/controls/image/image-unscaled.png';
import ImageUniformToFillScreenshot from '/img/controls/image/image-uniform-to-fill.png';
import BlendModeMultiply from '/img/reference/animations-and-graphics/bitmap-blend-modes/Multiply.png';

Image 控件可以显示来自指定图像源的光栅图像。图像源可以是：

* 一个指向应用资源的字符串常量，
* 通过资源名称绑定并使用绑定转换器加载得到的位图，
* 或者直接从内存流中加载的位图。  

图像也可以作为其他控件内容的一部分使用。例如，你可以用 Image 控件创建一个图形化按钮。

图像还可以使用多种不同的混合模式进行渲染，从而改变它与背景内容的叠加方式。有关所有受支持混合模式的列表和示例画廊，请参阅 [位图混合模式](/docs/graphics-animation/bitmap-blend-modes) 页面。

显示的图像可以被调整大小和缩放。默认的缩放设置为双方向等比拉伸，因此图像会适配你指定的尺寸（宽度和/或高度）。

:::info
图像的缩放设置与 [Viewbox](/controls/layout/containers/viewbox) 相同。
:::

## 常用属性

| 属性 | 类型 | 说明 |
|---|---|---|
| `Source` | `IImage` | 要显示的图像。可通过资源 URI 字符串、`Bitmap` 或 `DrawingImage` 设置。 |
| `Stretch` | `Stretch` | 图像如何调整大小以填充其边界。见下表。 |
| `StretchDirection` | `StretchDirection` | 控制图像是只能放大、只能缩小，还是两者都允许。 |
| `BlendMode` | `BitmapBlendingMode` | 图像合成时使用的混合模式。 |

### Stretch 模式

| 值 | 行为 |
|---|---|
| `None` | 以原始尺寸显示图像。 |
| `Fill` | 调整图像大小以填满边界，不保留纵横比。 |
| `Uniform` | 在保留纵横比的前提下，将图像缩放到边界内（默认）。 |
| `UniformToFill` | 在保留纵横比的前提下，将图像缩放到填满边界；部分内容可能会被裁剪。 |

## 示例

### 基本示例

此示例展示了如何将一个位图资源加载到 Image 控件中，同时限制其宽度和高度，但保持默认缩放设置不变。图像本身不是正方形，但这里将图像的宽度和高度设置为相同值。背景矩形用于帮助你观察图像的缩放方式：

```xml
<Panel>
  <Rectangle Height="300" Width="300" Fill="LightGray"/>
  <Image Margin="20" Height="200" Width="200" 
         Source="avares://AvaloniaControls/Assets/pipes.jpg"/>
</Panel>
```

<Image light={ImageUnscaledScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

### Stretch

在下一个示例中，使用 `UniformToFill` 这一拉伸设置后，图像会完整适应高度，但由于宽度否则会超出指定范围，因此宽度会被裁剪。这种处理不会导致图像变形。

```xml
<Panel>
  <Rectangle Height="300" Width="300" Fill="LightGray"></Rectangle>
  <Image Margin="20" Height="200" Width="200" 
         Stretch="UniformToFill"
         Source="avares://AvaloniaControls/Assets/pipes.jpg"/>
</Panel>
```

<Image light={ImageUniformToFillScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

### BlendMode

此示例使用了两张图像，其中第二张图像应用了 `Multiply` 混合模式。更多信息请参阅 [位图混合模式](/docs/graphics-animation/bitmap-blend-modes) 页面。

```xml
<Panel>
    <Image Source="./Cat.jpg"/>
    <Image Source="./Overlay-Color.png" BlendMode="Multiply"/>
</Panel>
```

<Image light={BlendModeMultiply} alt="" position="center" maxWidth={400} cornerRadius="true"/>

## 另请参阅

- [Image API 参考](/api/avalonia/controls/image)
- [`Image.cs` GitHub 源代码](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/Image.cs)
